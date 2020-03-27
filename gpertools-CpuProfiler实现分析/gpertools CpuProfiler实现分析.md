## 简介
我们知道，使用gpertools时，只需要在你想分析的代码前面加入：
``` cpp
    ProfilerStart("output.prof");
```
然后，在需要停止观察的时候调用：
``` cpp
    ProfilerStop();
```
然后运行你的程序，就会将你需要的profile数据生成到"output.prof"文件中，通过使用`pprof`工具非常方便就能以可视化的方法对数据进行分析，从而找到你程序的性能瓶颈。

## 在Chromium上的应用
在Chroimum中也是使用了同样的方法，看过[Chrome初始化流程](https://kb.cvte.com/pages/viewpage.action?pageId=118918245)的同学可能会注意到，Browser进程的`Profiling`是从`ChromeMainDelegate::BasicStartupComplete`中启动的，如下：
```cpp
bool ChromeMainDelegate::BasicStartupComplete(int* exit_code) {
    ...
    Profiling::ProcessStarted();
    ...
}
````

我在[Chromium性能分析](https://kb.cvte.com/pages/viewpage.action?pageId=119544810)中提到在linux下启动Chromium的Profiling，需要在初始参数中加入`--profiling-at-start`，对这个参数的检查也是在`Profiling::ProcessStarted`中完成的。如果检查到有该参数，则会调用`Profiling::Start`函数，后者会将用户设置的`--profiling-file`参数作为输出堆栈数据的文件路径，如果用户没有设置，则会通过当前进程类型和PID自动生成一个文件名，并传入`StartProfiling`函数中，再由其完成对`ProfilerStart`的调用：
``` cpp
void StartProfiling(const std::string& name) {
  ++profile_count;
  std::string full_name(name);
  std::string pid = IntToString(GetCurrentProcId());
  std::string count = IntToString(profile_count);
  ReplaceSubstringsAfterOffset(&full_name, 0, "{pid}", pid);
  ReplaceSubstringsAfterOffset(&full_name, 0, "{count}", count);
  ProfilerStart(full_name.c_str());
}
```

## ProfilerStart实现分析
`ProfilerStart`的实现放在了第三方库目录的`tcmalloc`下（src\third_party\tcmalloc\chromium\src\profiler.cc），实现其实并不复杂，它应用了一个计时器，每隔一段时间就将当前进程堆栈中的数据保存起来，接下来我们看下具体实现。
```
extern "C" PERFTOOLS_DLL_DECL int ProfilerStart(const char* fname) {
  return CpuProfiler::instance_.Start(fname, NULL);
}
```
通过CpuProfiler实例调用其Start函数，参数为输出文件名：
```
bool CpuProfiler::Start(const char* fname, const ProfilerOptions* options) {
    ...
    collector_options.set_frequency(prof_handler_state.frequency);
      if (!collector_.Start(fname, collector_options)) {
        return false;
      }
    ...
    EnableHandler();
    ...
}
```
首先，如果有`CPUPROFILE_FREQUENCY`环境变量，则将其作为数据采集的频率，如果没有设置则使用默认值100，意味着每秒会进行100次的采集。collector_是一个ProfileData对象，这个对象从名字就可以猜他主要用于管理Profile数据，Start函数会打开fname文件名指向的文件，并用数据采集频率初化计时器的步长`period`。
紧接着`EnableHandler()`通过调用`ProfileHandlerRegisterCallback`将`prof_handler`函数注册为计时器触发时的响应函数。
```
void CpuProfiler::EnableHandler() {
  RAW_CHECK(prof_handler_token_ == NULL, "SIGPROF handler already registered");
  prof_handler_token_ = ProfileHandlerRegisterCallback(prof_handler, this);
  RAW_CHECK(prof_handler_token_ != NULL, "Failed to set up SIGPROF handler");
}
```

### 计时器
ProfileHandlerRegisterCallback在profile-handler.cc中定义，主要实现为调用ProfileHandler的RegisterCallback函数，后者负责管理响应函数,同时启动定时器。linux下的定时器主要通过`setitimer`函数和`sigaction`函数实现:
``` cpp
void ProfileHandler::StartTimer() {
  if (!allowed_) {
    return;
  }
  struct itimerval timer;
  timer.it_interval.tv_sec = 0;
  timer.it_interval.tv_usec = 1000000 / frequency_;
  timer.it_value = timer.it_interval;
  setitimer(timer_type_, &timer, 0);
}
```
设置了时间间隔为1s/frequency_，timer_type_的初始化在构造函数中完成，主要通过检查是否有`CPUPROFILE_REALTIME`环境变量，如果有则为`ITIMER_REAL`类型，否则为`ITIMER_PROF`类型，具体区别可以看下[man](http://man7.org/linux/man-pages/man2/setitimer.2.html)。
在RegisterCallback中还调用了`EnableHandler()`函数，用于注册计时器触发响应函数：
``` cpp
void ProfileHandler::EnableHandler() {
  ...
  struct sigaction sa;
  sa.sa_sigaction = SignalHandler;
  sa.sa_flags = SA_RESTART | SA_SIGINFO;
  sigemptyset(&sa.sa_mask);
  const int signal_number = (timer_type_ == ITIMER_PROF ? SIGPROF : SIGALRM);
  RAW_CHECK(sigaction(signal_number, &sa, NULL) == 0, "sigprof (enable)");
}
```
定时器触发，内核产生软中断，于是我们的进程就可以对信号进行处理。这里的处理为调用SignalHandler函数，SignalHandler函数循环遍历刚才通过RegisterCallback注册进来的函数，于是我们前面注册进来的`prof_handler`函数终于有机会忘情奔跑了。

### 核心：CpuProfiler::prof_handler函数解析
prof_handler的定义还是在我们一开始提到的tcmalloc下的profiler.cc中，他的工作主要是把当前函数调用的堆栈信息保存到profile-data中。具体实现是通过保存PC寄存器（类似8086汇编中的IP/EIP寄存器）值来实现，知道了pc指针的地址，我们就能知道其处于哪个函数中，再根据symbols就可以取得函数名。接下来的内容会涉及到一些与汇编相关的知识（虽然我大学时做过一段时间的反汇编，但其实也花了几个小时才读懂了这部分的代码）,我会努力以简单的方式表达出来。
我们先看下prof_handler的实现：
``` cpp
void CpuProfiler::prof_handler(int sig, siginfo_t*, void* signal_ucontext,
                               void* cpu_profiler) {
  CpuProfiler* instance = static_cast<CpuProfiler*>(cpu_profiler);

  if (instance->filter_ == NULL ||
      (*instance->filter_)(instance->filter_arg_)) {
    void* stack[ProfileData::kMaxStackDepth];

    // The top-most active routine doesn't show up as a normal
    // frame, but as the "pc" value in the signal handler context.
    stack[0] = GetPC(*reinterpret_cast<ucontext_t*>(signal_ucontext));

    // We skip the top two stack trace entries (this function and one
    // signal handler frame) since they are artifacts of profiling and
    // should not be measured.  Other profiling related frames may be
    // removed by "pprof" at analysis time.  Instead of skipping the top
    // frames, we could skip nothing, but that would increase the
    // profile size unnecessarily.
    int depth = GetStackTraceWithContext(stack + 1, arraysize(stack) - 1,
                                         2, signal_ucontext);
    depth++;  // To account for pc value in stack[0];

    instance->collector_.Add(depth, stack);
  }
}
```
signal_ucontext是定时器触发时返回的一个参数，他其实是一个ucontext_t结构体，保存了当前多个寄存器的内容，具体使用可以看下[man](http://man7.org/linux/man-pages/man2/getcontext.2.html)。
pc（progtam counter ）寄存器和我们在86x86汇编上经常提到的ip（instruction pointer）寄存器类似，一般非intel芯片会将ip寄存器称作pc寄存器，他们都是指向下一条要执行的指令。
回到这个函数，他定义了一个用于保存void指针的数组，实际上最后它是用于保存函数调用栈的EIP寄存器的值，但首先他通过signal_ucontext获取到当前pc寄存器的值并保存了下来。接着调用GetStackTraceWithContext函数，这个函数的重点就是通过循环历遍函数调用栈堆栈，然后将ip寄存器的值存到stack中。
#### GetStackTraceWithContext函数解析
`GetStackTraceWithContext`的定义在stacktrace.cc中通过宏实现：
``` cpp
#define GET_STACK_TRACE_OR_FRAMES \
  GetStackTraceWithContext(void **result, int max_depth, \
                           int skip_count, const void *ucp)
```
其实现根据平台的不同也放在了各个的stracktrace_*.h中，因为我当前在x86平台虚拟机上，所以他的实现就在stracktrace_x86-inl.h中。可以看到首先针对不同的编译器和平台，通过不过的方法初始化了sp指针：
``` cpp
int GET_STACK_TRACE_OR_FRAMES {
  void **sp;
#if (__GNUC__ > 4) || (__GNUC__ == 4 && __GNUC_MINOR__ >= 2) || __llvm__
  sp = reinterpret_cast<void**>(__builtin_frame_address(0));
#elif defined(__i386__)
  sp = (void **)&result - 2;  //ebp 的地址
#elif defined(__x86_64__)
  unsigned long rbp;
  __asm__ volatile ("mov %%rbp, %0" : "=r" (rbp));
  sp = (void **) rbp;
#else
# error Using stacktrace_x86-inl.h on a non x86 architecture!
#endif
    ...
}
```
刚开始读这段代码的时候，一直被sp这个指针的命名骗了，学过汇编的人应该知道，sp/esp寄存器也叫堆栈指针寄存器，用于指向了当前函数调用栈的栈顶，所以我一开始以为这个指针也是用于指向当前函数调用栈的栈顶的。其实不是，这里他指向存放bp/ebp寄存器的地址，后面你就会看到他为什么要指向这个位置，是为了能快速找到上一层函数存放ebp的地址。

bp/epb是指基址指针寄存器，用于指向当前函数所在栈的栈底，也称作帧指针。如果编译器支持__builtin_frame_address函数，则可以直接通过函数获取到帧指针的值，否则如果为i386架构，则可以通过偏移取得ebp寄存器的值，再者如果是x64的话，则通过内嵌汇编的方式取得rbp寄存器的值。我们重点看下在386架构下为什么可以通过这种偏移的方式取得ebp指针，而且这种偏移在后面也会用到。
首先，通过&result取得这个传入参数的地址。因为函数调用时参数采用自右向左的入栈顺序，所以result参数是最后一个入栈的参数，他的位置就显得很重要的，因为在他入栈后，会通过`call`指令跳转到另一个函数中。我们知道`call`指令在跳转前会先将eip寄存器入栈，后然在`ret`指令的时候再将eip出栈，之所以这么做因为在函数运行完成后需要返回到原来的函数中继续执行，如果没有保存之前的eip的话这里将不知道函数要返回哪里。在`call`完成跳转后，新函数中的第一条汇编指令是将ebp寄存器入栈，因为这个ebp指针存放的是他上一级函数调用的栈基址，所以我们需要保存他，这样我们在函数返回时就可以知道上一级函数的栈基址了。
回想下，如果现在栈上以result所在的地址为基址的话，在他上面还存了一个eip寄存器的值和和ebp寄存器的值，所以通过`(void **)&result - 2`就可以取得栈上保存的ebp寄存器的值了,至于为什么这里是减2，那是因为栈的长生方向是从高地址向低地址方向生长，所以栈顶的地址应该是最小的。

我截取了一段简单程序的汇编代码，可以对照着看：
``` assemble
//main函数中
00831A42  push        3                     //将参数入栈 
00831A44  push        2                     //将参数入栈
00831A46  lea         eax,[result]  
00831A49  push        eax                   //将参数入栈
00831A4A  call        test (08313A2h)      //函数调用
00831A4F  add         esp,0Ch               //函数返回的位置，也就是函数调用时eip指针的值，需要先被压入栈中，单纯从这段汇编来讲，他用于返还栈空间。
00831A52  xor         eax,eax  

...
// test 函数最前面
00831950  push        ebp                   //将ebp入栈
00831951  mov         ebp,esp               //更新ebp的值 
00831953  sub         esp,0C0h              //增长栈空间 
```
函数调用栈模拟图：
![x86.png](./x86.jpg)

理解了上面这些以后，`GET_STACK_TRACE_OR_FRAMES`剩下的代码就好理解多了:
``` cpp
int GET_STACK_TRACE_OR_FRAMES {

    ...

  while (sp && n < max_depth) {

    ...

    void **next_sp = NextStackFrame<!IS_STACK_FRAMES, IS_WITH_CONTEXT>(sp, ucp);
    if (skip_count > 0) {
      skip_count--;
#if defined(KEEP_SHADOW_STACKS)
      shadow_index--;
#endif  // KEEP_SHADOW_STACKS
    } else {
      result[n] = *(sp+1); 

      ...

      n++;
    }
    ...
    sp = next_sp;
  }
    ...
```

其实NextStackFrame函数的实现你可以简单看成：
``` cpp
  void **next_sp = (void**)(*sp);
```
也就是通过指向存入ebp地址的指针，取得ebp寄存器压入栈中的地址，这个地址也就是上一级函数的存放ebp寄存器的地址，假如这时候再对next_sp取值，又可以取得上上个函数的存放ebp寄存器的地址。实际的实现的函数中还需要考虑next_sp指针的合法性，并对不同平台下做一些特殊的处理。
skit_count的作用是不保存最前面的几个栈指针。
最后把sp+1的值取值后存入result中，有了前面提到的作为基础，我们就知道了sp+1指向的是存放eip寄存器的地址，对其取值就可以得到函数的返回地址，也就是上一级函数的地址空间，到这里主要任务就完成了。取到函数的地址空间后有什么用呢？假如你取到了一个eip值为0x00110000，而你又知道test函数的地址空间为0x00100000到0x00130000，那你就可以确认test函数在当前的函数调用栈中，通过循环取得当前调用栈中所有的eip地址，你就能得到一个完整的调用链，也就是我们使用pprof --gv看到的图形了。

`GetStackTraceWithContext`的返回值为函数调用栈的尝试，也就是上面循环运行的次数，调用返回后将result的数据通过ProfileData::Add存入ProfileData中，等待被写入文件中。

#### ProfileData::Add 函数解析
因为取样频率非常快，所以我们很大程序上会取到大量重复的样本，这个函数中主要任务就是对样本进行取哈希，统计和保存。
``` cpp
void ProfileData::Add(int depth, const void* const* stack) {
  ...
  // Make hash-value
  Slot h = 0;
  for (int i = 0; i < depth; i++) {
    Slot slot = reinterpret_cast<Slot>(stack[i]);
    h = (h << 8) | (h >> (8*(sizeof(h)-1)));
    h += (slot * 31) + (slot * 7) + (slot * 3);
  }
  ...
}
```
这里用了一个哈希函数对stack中我们通过`GetStackTraceWithContext`取到的所有值进行计算，最后生成一个hash值，他用于标识唯一的一个函数调用栈，再对得到的哈希值分桶，这样就可以很快的得到当前是否已经有记录过相同的函数调用栈，如果存在的话就将其频数进行自增就可以了，如果不存在，则将其存入ProfileData的stack数组中，并计算下hash值，初始化频数为1。

至此，一次完整的取样过程就非常高效的已经完成了。在退出取样后，解析的工作就由 `pprof`脚本完成，后序有时间再分析他是怎么进行解析的。