## ![skynet logo](https://github.com/cloudwu/skynet/wiki/image/skynet_metro.jpg)

Skynet is a lightweight online game framework which can be used in many other fields.

## Build

For Linux, install autoconf first for jemalloc:

```
git clone https://github.com/cloudwu/skynet.git
cd skynet
make 'PLATFORM'  # PLATFORM can be linux, macosx, freebsd now
```

Or:

```
export PLAT=linux
make
```

For FreeBSD , use gmake instead of make.

## Test

Run these in different consoles:

```
./skynet examples/config	# Launch first skynet node  (Gate server) and a skynet-master (see config for standalone option)
./3rd/lua/lua examples/client.lua 	# Launch a client, and try to input hello.
```

## About Lua version

Skynet now uses a modified version of lua 5.3.5 ( https://github.com/ejoy/lua/tree/skynet ) for multiple lua states.

Official Lua versions can also be used as long as the Makefile is edited.

## How To Use (Sorry, currently only available in Chinese)

* Read Wiki for documents https://github.com/cloudwu/skynet/wiki
* The FAQ in wiki https://github.com/cloudwu/skynet/wiki/FAQ

作者：知乎用户
链接：https://www.zhihu.com/question/38427301/answer/76386687
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

一个Timer的实现需要具备以下几个行为:StartTimer(Interval, ExpiryAction)注册一个时间间隔为 Interval 后执行 ExpiryAction 的定时器实例，其中，返回 TimerId 以区分在定时器系统中的其他定时器实例。StopTimer(TimerId)根据 TimerId 找到注册的定时器实例并执行 Stop 。PerTickBookkeeping()在一个 Tick 时间粒度内，定时器系统需要执行的动作，它最主要的行为，就是检查定时器系统中，是否有定时器实例已经到期。具体的代码实现思路就是：在StartTimer的时候，把 当前时间 + Interval 作为key放入一个容器，然后在Loop的每次Tick里，从容器里面选出一个最小的key与当前时间比较，如果key小于当前时间，则这个key代表的timer就是expired，需要执行它的ExpiryAction(一般为回调)。这里有两个实现的细节：获取当前时间 这可能是实现一个定时器唯一需要关心平台差异的地方，需要注意的地方包括时间精度，使用系统开启后的CPU时间还是壁钟时间，常用的API是：    Windows: QueryPerformanceFrequency() 和 QueryPerformanceCounter()    Linux: clock_gettime()    OSX: gettimeofday()或者mach_absolute_time()    当然在C++11里也可以偷懒使用chrono的high_resolution_clock std::chrono::high_resolution_clock2. Timer容器的选择容器应该能够在很短的时间内找到MinValue   最小堆的find-min复杂度是O(1)，所以蛮受人喜欢的   STL里提供有堆的API，make_heap, push_heap, pop_heap, sort_heap或者直接使用std::priority_queue(std::priority_queue - cppreference.com)3. PerTickBookkeeping是放在主循环线程还是另起线程放在主循环的话的套路大都是基于回调函数的单线程实现，另起线程需要想好线程间通信的方式，asio和skynet有单独的timer线程4，一些开源项目里的timer实现a) 这是boost.asio的实现的timer_queue，用的是最小堆https://github.com/chriskohlhoff/asio/blob/master/asio/include/asio/detail/timer_queue.hpp#35b) 这是libuv的timer，用的还是最小堆：https://github.com/libuv/libuv/blob/v1.23.0/src/timer.cc) 这是linux kernel里的实现，经典的多级链表时间轮https://github.com/torvalds/linux/blob/v4.7/kernel/time/timer.c时间轮的实现原理可以参考这里: [转载]linux 内核定时器详解另外，也有其它著名的开源项目采用了时间轮的方式来实现定时器，比如，云风的skynet skynet/skynet_timer.c at master · cloudwu/skynet · GitHubfacebook的folly里面的HHWheelTimer https://github.com/facebook/folly/blob/master/folly/io/async/HHWheelTimer.h参考：Linux 下定时器的实现方式分析
