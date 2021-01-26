[雷霄骅SDL2源代码分析](https://blog.csdn.net/leixiaohua1020/article/details/40680907)

[SDL中文教程👏](http://tjumyk.github.io/sdl-tutorial-cn/lessons/lesson01/index2.html)

Python（借由pygame库）

https://www.cnblogs.com/xl2432/p/11791295.html



malloc()，calloc()。

C语言的标准内存分配函数：malloc，calloc，realloc，free等。
malloc与calloc的区别为1块与n块的区别：
malloc调用形式为(类型*)malloc(size)：在内存的动态存储区中分配一块长度为“size”字节的连续区域，返回该区域的首地址。
calloc调用形式为(类型*)calloc(n，size)：在内存的动态存储区中分配n块长度为“size”字节的连续区域，返回首地址。
realloc调用形式为(类型*)realloc(*ptr，size)：将ptr内存大小增大到size。
free的调用形式为free(void*ptr)：释放ptr所指向的一块内存空间。



## SDL_Init(Uint32 flags) 

```c
int SDL_Init(Uint32 flags) 
```

使用此函数初始化SDL库，必须在使用大多数其他SDL函数之前调用它。

| flags                       | 子系统                             |
| --------------------------- | ---------------------------------- |
| **SDL_INIT_TIMER**          | 计时器子系统                       |
| **SDL_INIT_AUDIO**          | 音频子系统                         |
| **SDL_INIT_VIDEO**          | 视频子系统; 自动初始化事件子系统   |
| **SDL_INIT_JOYSTICK**       | 操纵杆子系统; 自动初始化事件子系统 |
| **SDL_INIT_HAPTIC**         | 触觉（力反馈）子系统               |
| **SDL_INIT_GAMECONTROLLER** | 控制子系统; 自动初始化操纵杆子系统 |
| **SDL_INIT_EVENTS**         | 事件子系统                         |
| **SDL_INIT_EVERYTHING**     | 所有上述子系统                     |
| **SDL_INIT_NOPARACHUTE**    | 兼容性; 这个标志被忽略了           |



## SDL_CreateRenderer()

**函数简单介绍**

SDL中使用SDL_CreateRenderer()基于窗体创建渲染器。SDL_CreateRenderer()原型例如以下。

```
SDL_Renderer * SDLCALL SDL_CreateRenderer(SDL_Window * window,
                                               int index, Uint32 flags);
```



參数含义例如以下。



window	： 渲染的目标窗体。



index	：打算初始化的渲染设备的索引。

设置“-1”则初始化默认的渲染设备。

flags	：支持以下值（位于SDL_RendererFlags定义中）

  SDL_RENDERER_SOFTWARE ：使用软件渲染

  SDL_RENDERER_ACCELERATED ：使用硬件加速

  SDL_RENDERER_PRESENTVSYNC：和显示器的刷新率同步

  SDL_RENDERER_TARGETTEXTURE ：不太懂

返回创建完毕的渲染器的ID。假设创建失败则返回NULL。



init

**通过函数指针调用不同平台的子系统的具体初始化函数，从而实现跨平台**

> ...该函数首先通过SDL_calloc()的方法为创建的SDL_VideoDevice分配了一块内存，接下来为创建的SDL_VideoDevice结构体中的函数指针赋了一大堆的值。这也是SDL“跨平台”特性的一个特点：通过调用SDL_VideoDevice中的接口函数，就可以调用不同平台的具体实现功能的函数。



```c
// 如
SDL_VideoDevice {...int (*VideoInit)(_THIS);//指针函数}

//在window下
    device->VideoInit = WIN_VideoInit
int WIN_VideoInit(_THIS)
{
    if (WIN_InitModes(_this) < 0) {
        return -1;
    }

    WIN_InitKeyboard(_this);
    WIN_InitMouse(_this);

    SDL_AddHintCallback(SDL_HINT_WINDOWS_ENABLE_MESSAGELOOP, UpdateWindowsEnableMessageLoop, NULL);
    SDL_AddHintCallback(SDL_HINT_WINDOW_FRAME_USABLE_WHILE_CURSOR_HIDDEN, UpdateWindowFrameUsableWhileCursorHidden, NULL);

    return 0;
}
                 
//在x11下
    device->VideoInit = X11_VideoInit;
int X11_VideoInit(_THIS)
{
    SDL_VideoData *data = (SDL_VideoData *) _this->driverdata;

    /* Get the window class name, usually the name of the application */
    data->classname = get_classname();

    /* Get the process PID to be associated to the window */
    data->pid = getpid();

    /* I have no idea how random this actually is, or has to be. */
    data->window_group = (XID) (((size_t) data->pid) ^ ((size_t) _this));
  	... ...
    return 0
}


```





SDL SDL_PumpEvents更新输入设备事件



SDL_PumpEvents更新键盘状态（ Use SDL_PumpEvents to update the state array）

SDL_PeekEvents用于读取事件，在调用该函数之前，必须调用SDL_PumpEvents搜集键盘等事件
SDL_GETEVENT表示读取队列中的事件，并且将该事件从队列中移除
设置了SDL_GETEVENT操作属性的SDL_PeekEvents等同于调用SDL_PoolEvent获取事件



SDL_PeepEvents

功能 :从事件队列中搜寻相对应的事件，但事件本身仍然在事件队列中。

SDL_PumpEvents
从输入设备中搜集事件，推动这些事件进入事件队列，更新事件队列的状态，不过它还有一个作用是进行视频子系统的设备状态更新，如果不调用这个函数，所显示的视频会在大约10秒后丢失色彩。没有调用SDL_PumpEvents，将不会有任何的输入设备事件进入队列，这种情况下，SDL就无法响应任何的键盘等硬件输入。

注意
该函数只能在视频子系统的线程中调用，更加稳妥的做法，只应该在主线程中被调用

https://blog.51cto.com/fengyuzaitu/1892892?source=drt



SDL_PollEvents()函数的功能是事件轮询。首先通过SDL_PumpEvents函数来处理硬件独立的事件后，再通过SDL_PeepEvents从队列中提取事件。
另外还有两个事件处理函数：
SDL_WaitEvent()必须等到有一个事件才返回，而SDL_PollEvent 没有事件也立即返回，这样提高系统反应速度。
SDL_PeepEvents()是提出查看事件，但事件本身仍然在事件队列中。



PacketQueue、FrameQueue

packetQueue是要链表实现的、

FrameQueue是要数组实现的、环形缓冲区

> 环形缓冲区



SDL的多线程与锁机制

SDL线程创建：SDL_CreateThread　

SDL线程等待：SDL_WaitThead

SDL互斥锁：SDL_CreateMutex / SDL_DestroyMutex

SDL锁定互斥：SDL_LockMutex / SDL_UnlockMutex

SDL 条件变量（信号量）：SDL_CreateCond / SDL_DestoryCond

SDL 条件变量（信号量）等待 / 通知 ：SDL_CondWait / SDL_CondSingal

 SDL_CondWait暂时释放拥有的锁，并进入睡眠状态； 在别的线程中调用SDL_CondSignal后，SDL_CondWait唤醒，等到别的线程将锁释放后，SDL_CondWait所在线程重新拥有锁，可以继续往下执行；

https://www.cnblogs.com/renhui/p/10498177.html



SDL_CreateCond

需要深入探究的问题

* 多线程与锁机制

