# MAC上FFmpeg的开发日记

## 开发环境搭建：Xcode

参考：https://www.pianshen.com/article/2994310497/

> .o .a .so

```bash
.o 就相当于windows里的obj文件 ，一个.c或.cpp文件对应一个.o文件 
.a 是好多个.o合在一起,用于静态连接 ，即STATIC mode，多个.a可以链接生成一个exe的可执行文件 
.so 是shared object,用于动态连接的,和windows的dll差不多，使用时才载入。
```

(uint8_t*) 这个是强制转换成uint8_t类型的指针



初始化在ffmpeg是一个很常用的概念：各类context的初始化

AVFormatContext

AVCodecContext

SwsContext





#### 指针数组



```bash
int *ptr[MAX];
#在这里，把 ptr 声明为一个数组，由 MAX 个整数指针组成。因此，ptr 中的每个元素，都是一个指向 int 值的指针。
```



[Readme2](./hello)

### const* 和 *const

https://blog.csdn.net/qq_32623363/article/details/87813484



比如`int const*a;`，实际上可以看成是`int const (*a)`，这表示指针`a`所指向的地址可以变，但是所指向的那个值不能变。
而`int *const a;`，可以看成`int* (const a);`，我们都知道a的值其实是一个地址，这就表示`a`所保存的地址是不可以变的，但是这个地址对应的值是可以变的。

指向常量的常指针，既不能改地址，也不能改值

​    const int * const a = &b; 



http://c.biancheng.net/view/228.html

### 函数指针的作用



