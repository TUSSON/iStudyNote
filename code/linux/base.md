### 动态共享库中的全局变量
每个进程都有一份动态库中数据段的拷贝，所以全局变量不是共享的

### SIGSEGV deedbaad 
F/libc    ( 1805): Fatal signal 11 (SIGSEGV) at 0xdeadbaad (code=1), thread 1882 (Binder_3)

The malloc debug property is probably setting some magic numbers before and after the area you allocate. Then, when freeing, it would check those areas to make sure the magic number is still there.

For example, if you allocate 1024 bytes:

char * p = malloc(1024);

The malloc debug code would actually allocate 1024 bytes that you requested, plus some extra around it:

[ 32 bytes ---- | -------- 1024 bytes ------| ---- 32 bytes ]
^ 0xc0000000    ^ 0xc0000020

The library would then write a magic value into those 32 bytes:

[ 32 bytes ---- | -------- 1024 bytes ------| ---- 32 bytes ]
[  0xdeadd00d   |                           | 0xdeadd00d    ]
^ 0xc0000000    ^ 0xc0000020

The library would return 0xc0000020 into p and internally it would save 0xc0000000, the size, etc. Your function then uses the allocated area somehow:

memset(p, 0, 1025);

Notice this line copied more than 1024 bytes. That would write a 0 into the last 32 byte magic area (notice the 0 in the last 32 bytes, which should be 0xdeadd00d):

    [ 32 bytes ---- | -------- 1024 bytes ------| ---- 32 bytes ]
    [  0xdeadd00d   |  000...             ...00 | 0x0eadd00d    ]
    ^ 0xc0000000    ^ 0xc0000020  (address)

    When your function calls free:

    free(p);

    The library would then check to make sure the first and last 32 bytes are still 0xdeadd00d. Since your function overwrote the last 32 bytes, it would print an error like you posted.

    This is only an example of how a malloc debug checking would work. If you would like to see exactly what the malloc debug checks and how it works, go to the bionic directory of Android source and search for the property you set, libc.debug.malloc. 

### 定制化的malloc/free

    If you link other code with this file using –wrap malloc, then all calls to malloc will call the function __wrap_malloc instead. The call to __real_malloc in __wrap_malloc will call the real “malloc” function.

    You may wish to provide a __real_malloc function as well, so that links without the –wrap option will succeed. If you do this, you should not put the definition of __real_malloc in the same file as __wrap_malloc; if you do, the assembler may resolve the call before the linker has a chance to wrap it to malloc.
    

```c++
void * __wrap_malloc (size_t c)
{
    printf ("malloc called with %zu\n", c);
    return __real_malloc (c);
}
```
wrap既可以用于变量符号，也可以用于函数符号，但我们现在要用的只是函数符号，准确地说，就是malloc和free这两个符号。
这是一个在链接过程中起作用的选项，在链接选项中加上-Wl,--wrap,malloc -Wl,--wrap,free

