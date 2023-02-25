# Metadata——带析构函数的安全指针

极度精简，只有一个头文件：

```c++
#ifndef __METADATA__H__
#define __METADATA__H__
#ifdef __cplusplus
extern "C"
{
#endif //__cplusplus
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <assert.h>

    typedef struct __metadata__struct__
    {
        void *__ptr__;
        size_t __allocated__;
        size_t __count__;
        void (*__destruct__)(void *_ptr);
    } metadata;

    // allocate function
    void *__metadata_alloc__(size_t size, void (*__destruct__)(void *_ptr))
    {
        metadata *meta = (metadata *)malloc(sizeof(metadata));

        meta->__ptr__ = malloc(size);
        memset(meta->__ptr__, 0, size);

        meta->__count__ = 0;
        meta->__destruct__ = __destruct__;
        meta->__allocated__ = size;
        return meta;
    }
    void __metadata_free__(void *meta)
    {
        metadata *_meta = (metadata *)meta;

        assert(meta != _meta->__ptr__);
        _meta->__destruct__(_meta->__ptr__);
        _meta->__ptr__ = NULL;
        free(*(void **)meta);
        meta = NULL;
    }
#define metadata_alloc(size, destruct) (__metadata_alloc__(size, destruct))
#define metadata_free(meta_ptr) (__metadata_free__(meta_ptr))
#define metadata_ptr(meta_ptr) (*meta_ptr)
#define metadata_type(Typename) Typename *
#define metadata_dtorc(funcName) void funcName(void *ptr)

#ifdef __cplusplus
}
#endif //__cplusplus
#endif //!__METADATA__H__
```

使用`metadata_alloc(size, destruct)`申请内存。

通过`metadata_free(meta_ptr)`释放申请的内存。

通过`metadata_ptr(meta_ptr)`获取所申请的内存的指针

## example

```c++
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include "metadata.h"

//构造一个析构函数
metadata_dtorc(_p)
{
    free(_p);
}

int main(int argc, char const *argv[])
{
    //申请内存
    //metadata_type构造适用于metadata的类型
    metadata_type(char *) p = metadata_alloc(13, _p);

    memset(metadata_ptr(p), 0, 13);
    sprintf(metadata_ptr(p), "%s","Hello,World\n");
    printf(metadata_ptr(p));
    //释放内存
    metadata_free(p);
    return 0;
}
```

**该项目应用于appweb项目的内存解析器中**

# 相关信息

作者：[Yauntyour]([yauntyour (yauntyour) · GitHub](https://github.com/yauntyour/))

授权许可：MIT开源协议

参考：null