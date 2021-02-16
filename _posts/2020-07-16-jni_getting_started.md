---
title: JNI学习开动篇
keywords: vee, jvm, jni, getting_started
last_updated: July 16, 2020
created: 2020-07-16
tags: [jni]
summary: "本文为JNI学习的开动篇"
sidebar: mydoc_sidebar
permalink: 2020-07-16-jni_getting_started.html
folder: vee/jni
---

## 引言

## 以Zygote启动来看涉及的JNI过程
system/core/rootdir/init.zygote64.rc

```sh
service zygote /system/bin/app_process64 -Xzygote /system/bin --zygote --start-system-server
    class main
    priority -20
    user root
    group root readproc reserved_disk
    socket zygote stream 660 root system
    onrestart write /sys/android_power/request_state wake
    onrestart write /sys/power/state on
    onrestart restart audioserver
    onrestart restart cameraserver
    onrestart restart media
    onrestart restart netd
    onrestart restart wificond
    writepid /dev/cpuset/foreground/tasks
```
看到 --zygote --start-system-server 这两个参数传入给了app_main.cpp的main里面

frameworks/base/cmds/app_process/app_main.cpp
```cpp
int main(int argc, char* const argv[])
{
  // ...
  AppRuntime runtime(argv[0], computeArgBlockSize(argc, argv));
  // ...
  if (zygote) {
      runtime.start("com.android.internal.os.ZygoteInit", args, zygote);
  }
  // ...
}
```

runtime is an instance of **AppRuntime** inherited from **AndroidRuntime**.


frameworks/base/core/jni/AndroidRuntime.cpp

```cpp
/*
 * Start the Android runtime.  This involves starting the virtual machine
 * and calling the "static void main(String[] args)" method in the class
 * named by "className".
 *
 * Passes the main function two arguments, the class name and the specified
 * options string.
 */
void AndroidRuntime::start(const char* className, const Vector<String8>& options, bool zygote)
{
  //...
  /* start the virtual machine */
  JniInvocation jni_invocation;
  jni_invocation.Init(NULL);
  JNIEnv* env;
  if (startVm(&mJavaVM, &env, zygote) != 0) {
      return;
  }
  onVmCreated(env);

  /*
   * Register android functions.
   */
  if (startReg(env) < 0) {
      ALOGE("Unable to register all android natives\n");
      return;
  }


```
*frameworks/base/core/jni/AndroidRuntime.cpp*
对于语句 **JniInvocation jni_invocation;**, 我们看下构造函数

libnativehelper/JniInvocation.cpp
```cpp
JniInvocation::JniInvocation() :
    handle_(NULL),
    JNI_GetDefaultJavaVMInitArgs_(NULL),
    JNI_CreateJavaVM_(NULL),
    JNI_GetCreatedJavaVMs_(NULL) {

  LOG_ALWAYS_FATAL_IF(jni_invocation_ != NULL, "JniInvocation instance already initialized");
  jni_invocation_ = this;
}
```
jni_invocation里面的四个变量handle_、JNI_GetDefaultJavaVMInitArgs_、JNI_CreateJavaVM_、
JNI_GetCreatedJavaVMs_初始值为NULL,而内部的jni_invocation_变量指向jni_invocation变量本身。

*frameworks/base/core/jni/AndroidRuntime.cpp*
我们接着看下一条语句 **jni_invocation.Init(NULL);**

libnativehelper/JniInvocation.cpp
```cpp
bool JniInvocation::Init(const char* library) {
#ifdef __ANDROID__
  char buffer[PROP_VALUE_MAX];
#else
  char* buffer = NULL;
#endif
  library = GetLibrary(library, buffer);
  // Load with RTLD_NODELETE in order to ensure that libart.so is not unmapped when it is closed.
  // This is due to the fact that it is possible that some threads might have yet to finish
  // exiting even after JNI_DeleteJavaVM returns, which can lead to segfaults if the library is
  // unloaded.
  const int kDlopenFlags = RTLD_NOW | RTLD_NODELETE;
  handle_ = dlopen(library, kDlopenFlags);
  ...
  if (!FindSymbol(reinterpret_cast<void**>(&JNI_GetDefaultJavaVMInitArgs_),
                  "JNI_GetDefaultJavaVMInitArgs")) {
    return false;
  }
  if (!FindSymbol(reinterpret_cast<void**>(&JNI_CreateJavaVM_),
                  "JNI_CreateJavaVM")) {
    return false;
  }
  if (!FindSymbol(reinterpret_cast<void**>(&JNI_GetCreatedJavaVMs_),
                  "JNI_GetCreatedJavaVMs")) {
    return false;
  }
  return true;
}
```
**library = GetLibrary(library, buffer);** 这个语句用于获取library，也就是这里的libart.so
```cpp
static const char* kLibraryFallback = "libart.so";
const char* JniInvocation::GetLibrary(const char* library, char* buffer, bool (*is_debuggable)(),
                                      int (*get_library_system_property)(char* buffer)) {
#ifdef __ANDROID__
  const char* default_library;

  if (!is_debuggable()) {
    // Not a debuggable build.
    // Do not allow arbitrary library. Ignore the library parameter. This
    // will also ignore the default library, but initialize to fallback
    // for cleanliness.
    library = kLibraryFallback;
    default_library = kLibraryFallback;
    ...
  }
  ...
  if (library == NULL) {
    library = default_library;
  }

  return library;
}
```
接着 **handle_ = dlopen(library, kDlopenFlags);**会返回一个handle赋给handle_，这个是动态库打开，然后
返回一个句柄供后续dlsym之类的使用。
我们返回libnativehelper/JniInvocation.cpp里的Init函数接着往下看，是有三个FIndSymbol的调用，我们看下这个函数，其实就是通过dlsym来找到对应的函数地址，并赋给JNI_GetDefaultJavaVMInitArgs_、JNI_CreateJavaVM_、JNI_GetCreatedJavaVMs_这三个变量。

libnativehelper/JniInvocation.cpp
```cpp
bool JniInvocation::FindSymbol(void** pointer, const char* symbol) {
  *pointer = dlsym(handle_, symbol);
  if (*pointer == NULL) {
    ALOGE("Failed to find symbol %s: %s\n", symbol, dlerror());
    dlclose(handle_);
    handle_ = NULL;
    return false;
  }
  return true;
}
```

*frameworks/base/core/jni/AndroidRuntime.cpp*
我们回到AndroidRuntime.cpp中接着看下面语句的执行，首先是 **startVm(&mJavaVM, &env, zygote)**，
根据传入的参数，我们知道，这个startVM内部需要给mJavaVM和env两个变量进行赋值。

```cpp
/*
 * Start the Dalvik Virtual Machine.
 */
int AndroidRuntime::startVm(JavaVM** pJavaVM, JNIEnv** pEnv, bool zygote)
{
  ...
  /*
 * Initialize the VM.
 *
 * The JavaVM* is essentially per-process, and the JNIEnv* is per-thread.
 * If this call succeeds, the VM is ready, and we can start issuing
 * JNI calls.
 */
if (JNI_CreateJavaVM(pJavaVM, pEnv, &initArgs) < 0) {
    ALOGE("JNI_CreateJavaVM failed\n");
    return -1;
}

return 0;
}
```
startVm函数的绝大多数内容都是解析各种环境参数，我们直接跳到我们关心的点，就是
JNI_CreateJavaVM(pJavaVM, pEnv, &initArgs)这个函数的调用，这个函数位于

art/runtime/jni/java_vm_ext.cc
```c
// JNI Invocation interface.

extern "C" jint JNI_CreateJavaVM(JavaVM** p_vm, JNIEnv** p_env, void* vm_args) {
  ScopedTrace trace(__FUNCTION__);
  const JavaVMInitArgs* args = static_cast<JavaVMInitArgs*>(vm_args);
  if (JavaVMExt::IsBadJniVersion(args->version)) {
    LOG(ERROR) << "Bad JNI version passed to CreateJavaVM: " << args->version;
    return JNI_EVERSION;
  }
  RuntimeOptions options;
  for (int i = 0; i < args->nOptions; ++i) {
    JavaVMOption* option = &args->options[i];
    options.push_back(std::make_pair(std::string(option->optionString), option->extraInfo));
  }
  bool ignore_unrecognized = args->ignoreUnrecognized;
  if (!Runtime::Create(options, ignore_unrecognized)) {
    return JNI_ERR;
  }

  // Initialize native loader. This step makes sure we have
  // everything set up before we start using JNI.
  android::InitializeNativeLoader();

  Runtime* runtime = Runtime::Current();
  bool started = runtime->Start();
  if (!started) {
    delete Thread::Current()->GetJniEnv();
    delete runtime->GetJavaVM();
    LOG(WARNING) << "CreateJavaVM failed";
    return JNI_ERR;
  }

  *p_env = Thread::Current()->GetJniEnv();
  *p_vm = runtime->GetJavaVM();
  return JNI_OK;
}
```
首先看下 **Runtime::Create(options, ignore_unrecognized)**这行，会调用到下面

art/runtime/runtime.cc
```cpp
bool Runtime::Create(RuntimeArgumentMap&& runtime_options) {
  // TODO: acquire a static mutex on Runtime to avoid racing.
  if (Runtime::instance_ != nullptr) {
    return false;
  }
  instance_ = new Runtime;
  Locks::SetClientCallback(IsSafeToCallAbort);
  if (!instance_->Init(std::move(runtime_options))) {
    // TODO: Currently deleting the instance will abort the runtime on destruction. Now This will
    // leak memory, instead. Fix the destructor. b/19100793.
    // delete instance_;
    instance_ = nullptr;
    return false;
  }
  return true;
}
```
首先，这一行 **instance_ = new Runtime;** new了一个Runtime实例，并赋给变量，接着调用Init函数，





























