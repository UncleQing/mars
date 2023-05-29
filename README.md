## Mars

## <a name="mars_cn">Mars</a>

[0529] 现在这是我的Mars了~~~ 



* comm：可以独立使用的公共库，包括 socket、线程、消息队列、协程等；
* xlog：高可靠性高性能的运行期日志组件；
* SDT： 网络诊断组件；
* STN： 信令分发网络模块，也是 Mars 最主要的部分。

## Samples

sample 的使用请参考[这里](https://github.com/Tencent/mars/wiki/Mars-sample-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E)。

## Getting started

接入 [Android](#android_cn) 或者 [iOS/OS X](#apple_cn) 或者 [Windows](#windows_cn) 。

### <a name="android_cn">[Android](https://github.com/Tencent/mars/wiki/Mars-Android-%E6%8E%A5%E5%85%A5%E6%8C%87%E5%8D%97)</a>

gradle 接入我们提供了两种接入方式：[mars-wrapper](#wrapper) 或者 [mars-core](#core)。如果你只是想做个 sample 推荐使用 mars-wrapper，可以快速开发；但是如果你想把 mars 用到你的 app 中的话，推荐使用 mars-core，可定制性更高。

#### <a name="wrapper">mars-wrapper</a>

在 app/build.gradle 中添加 mars-wrapper 的依赖：


```xml
dependencies {
    compile 'com.tencent.mars:mars-wrapper:1.2.5'
}
```

**或者**

#### <a name="core">mars-core</a>

在 app/build.gradle 中添加 mars-core 的依赖：


```xml
dependencies {
    compile 'com.tencent.mars:mars-core:1.2.5'
}
```
**或者**
#### <a name="">mars-xlog</a>
如果只想使用 xlog,可以只加 xlog 的依赖(mars-core,mars-wrapper 中都已经包括 xlog)：
```xml
dependencies {
    compile 'com.tencent.mars:mars-xlog:1.2.5'
}
```
接着往下操作之前，请先确保你已经添加了 mars-wrapper 或者 mars-core 或者 mars-xlog 的依赖

#### <a name="Xlog">Xlog Init</a>

在程序启动加载 Xlog 后紧接着初始化 Xlog。但要注意如果你的程序使用了多进程，不要把多个进程的日志输出到同一个文件中，保证每个进程独享一个日志文件。而且保存 log 的目录请使用单独的目录，不要存放任何其他文件防止被 xlog 自动清理功能误删。


```java
System.loadLibrary("c++_shared");
System.loadLibrary("marsxlog");

final String SDCARD = Environment.getExternalStorageDirectory().getAbsolutePath();
final String logPath = SDCARD + "/marssample/log";

// this is necessary, or may crash for SIGBUS
final String cachePath = this.getFilesDir() + "/xlog"

//init xlog
Xlog.XLogConfig logConfig = new Xlog.XLogConfig();
logConfig.mode = Xlog.AppednerModeAsync;
logConfig.logdir = logPath;
logConfig.nameprefix = logFileName;
logConfig.pubkey = "";
logConfig.compressmode = Xlog.ZLIB_MODE;
logConfig.compresslevel = 0;
logConfig.cachedir = "";
logConfig.cachedays = 0;
if (BuildConfig.DEBUG) {
    logConfig.level = Xlog.LEVEL_VERBOSE;
    Xlog.setConsoleLogOpen(true);
} else {
    logConfig.level = Xlog.LEVEL_INFO;
    Xlog.setConsoleLogOpen(false);
}

Log.setLogImp(new Xlog());
```

程序退出时关闭日志：


```java
Log.appenderClose();
```

#### <a name="STN">STN Init</a>

如果你是把 mars-core 作为依赖加入到你的项目中的话，你需要显式的初始化和反初始化 STN

在使用 STN 之前进行初始化

```java
// set callback
AppLogic.setCallBack(stub);
StnLogic.setCallBack(stub);
SdtLogic.setCallBack(stub);

// Initialize the Mars PlatformComm
Mars.init(getApplicationContext(), new Handler(Looper.getMainLooper()));

// Initialize the Mars
StnLogic.setLonglinkSvrAddr(profile.longLinkHost(), profile.longLinkPorts());
StnLogic.setShortlinkSvrAddr(profile.shortLinkPort());
StnLogic.setClientVersion(profile.productID());
Mars.onCreate(true);
BaseEvent.onForeground(true);

StnLogic.makesureLongLinkConnected();
```

初始化顺序不一定要严格遵守上述代码的顺序，但在初始化时首先要调用 setCallBack 接口 (callback 文件的编写可以参考 demo)，再调用 Mars.init，最后再调用 onForeground 和 makesureLongLinkConnect，中间顺序可以随意更改。**注意：STN 默认是后台，所以初始化 STN 后需要主动调用一次**```BaseEvent.onForeground(true)```

需要释放 STN 或者退出程序时:

```java
Mars.onDestroy();
```

#### <a name="even">Event Change</a>

网络切换时:

```java
BaseEvent.onNetworkChange()
```
**********
如果你是把 mars-wrapper 作为依赖加入到你的项目中，你只需要显式的初始化 STN，不需要反初始化(因为 mars-wrapper 会进行反初始化)

```java
MarsServiceProxy.init(this, getMainLooper(),null);
```
************
不管你是使用 mars-wrapper 还是 mars-core，你都需要特别注意以下事件：


前后台切换:

```java
BaseEvent.onForeground(boolean);
```

应用的账号信息更改:

```java
StnLogic.reset();
```

如果你想修改 Xlog 的加密算法或者长短连的加解包部分甚至更改组件的其他部分，可以参考[这里](https://github.com/Tencent/mars/wiki/Mars-Android-%E6%8E%A5%E5%85%A5%E6%8C%87%E5%8D%97)

### <a name="apple_cn">[iOS/OS X](https://github.com/Tencent/mars/wiki/Mars-iOS%EF%BC%8FOS-X-%E6%8E%A5%E5%85%A5%E6%8C%87%E5%8D%97)</a>
编译

```
python build_ios.py
```

or 
```python
python build_osx.py
```

把 mars.framework 作为依赖加入到你的项目中，把mars/libraries/mars_android_sdk/jni 目录的后缀名为 rewriteme 的文件名删掉".rewriteme"和头文件一起加入到你的项目中。

#### <a name="Xlog">Xlog Init</a>

在程序启动加载 Xlog 后紧接着初始化 Xlog。但要注意保存 log 的目录请使用单独的目录，不要存放任何其他文件防止被 xlog 自动清理功能误删。


```cpp
NSString* logPath = [[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) objectAtIndex:0] stringByAppendingString:@"/log"];

// set do not backup for logpath
const char* attrName = "com.apple.MobileBackup";
u_int8_t attrValue = 1;
setxattr([logPath UTF8String], attrName, &attrValue, sizeof(attrValue), 0, 0);

// init xlogger
#if DEBUG
xlogger_SetLevel(kLevelDebug);
appender_set_console_log(true);
#else
xlogger_SetLevel(kLevelInfo);
appender_set_console_log(false);
#endif

XLogConfig config;
config.mode_ = kAppenderAsync;
config.logdir_ = [logPath UTF8String];
config.nameprefix_ = "Test";
config.pub_key_ = "";
config.compress_mode_ = kZlib;
config.compress_level_ = 0;
config.cachedir_ = "";
config.cache_days_ = 0;
appender_open(config);
```

在函数 "applicationWillTerminate" 中反初始化 Xlog

```cpp
appender_close();
```

#### <a name="STN">STN Init</a>

在你用 STN 之前初始化：

```objc
- (void)setCallBack {
    mars::stn::SetCallback(mars::stn::StnCallBack::Instance());
    mars::app::SetCallback(mars::app::AppCallBack::Instance());
}

- (void) createMars {
    mars::baseevent::OnCreate();
}

- (void)setClientVersion:(UInt32)clientVersion {
    mars::stn::SetClientVersion(clientVersion);
}

- (void)setShortLinkDebugIP:(NSString *)IP port:(const unsigned short)port {
    std::string ipAddress([IP UTF8String]);
    mars::stn::SetShortlinkSvrAddr(port, ipAddress);
}

- (void)setShortLinkPort:(const unsigned short)port {
    mars::stn::SetShortlinkSvrAddr(port);
}

- (void)setLongLinkAddress:(NSString *)string port:(const unsigned short)port debugIP:(NSString *)IP {
    std::string ipAddress([string UTF8String]);
    std::string debugIP([IP UTF8String]);
    std::vector<uint16_t> ports;
    ports.push_back(port);
    mars::stn::SetLonglinkSvrAddr(ipAddress,ports,debugIP);
}

- (void)setLongLinkAddress:(NSString *)string port:(const unsigned short)port {
    std::string ipAddress([string UTF8String]);
    std::vector<uint16_t> ports;
    ports.push_back(port);
    mars::stn::SetLonglinkSvrAddr(ipAddress,ports);
}

- (void)reportEvent_OnForeground:(BOOL)isForeground {
    mars::baseevent::OnForeground(isForground);
}

- (void)makesureLongLinkConnect {
    mars::stn::MakesureLonglinkConnected();
}
```

初始化顺序不一定要严格遵守上述代码的顺序，但在初始化时首先要调用 setCallBack 接口 (callback 文件的编写可以参考 demo)，再调用 Mars.init，最后再调用 onForeground 和 makesureLongLinkConnect，中间顺序可以随意更改。**注意：STN 默认是后台，所以初始化 STN 后需要主动调用一次**```BaseEvent.onForeground(true)```


需要释放 STN 或者退出程序时:

```objc
- (void)destroyMars {
    mars::baseevent::OnDestroy();
}
```

#### <a name="even">Event Change</a>

前后台切换时:

```objc
- (void)reportEvent_OnForeground:(BOOL)isForeground {
    mars::baseevent::OnForeground(isForeground);
}
```

网络切换时：

```objc
- (void)reportEvent_OnNetworkChange {
    mars::baseevent::OnNetworkChange();
}
```
### <a name="windows_cn">[Windows](https://github.com/Tencent/mars/wiki/Mars-Windows-%E6%8E%A5%E5%85%A5%E6%8C%87%E5%8D%97)</a>
安装Visual Studio 2015

编译

```python
python build_windows.py
```

把 mars.lib作为依赖加入到你的项目中，把mars/libraries/mars_android_sdk/jni 目录的后缀名为 rewriteme 的文件名删掉".rewriteme"和头文件一起加入到你的项目中。

#### <a name="Xlog">Xlog Init</a>

在程序启动加载 Xlog 后紧接着初始化 Xlog。但要注意保存 log 的目录请使用单独的目录，不要存放任何其他文件防止被 xlog 自动清理功能误删。

```cpp
std::string logPath = ""; //use your log path
std::string pubKey = ""; //use you pubkey for log encrypt

// init xlog
#if DEBUG
xlogger_SetLevel(kLevelDebug);
appender_set_console_log(true);
#else
xlogger_SetLevel(kLevelInfo);
appender_set_console_log(false);
#endif
appender_open(kAppenderAsync, logPath.c_str(), "Test", 0,  pubKey.c_str());
```

在程序退出前反初始化 Xlog

```cpp
appender_close();
```

#### <a name="STN">STN Init</a>

在你用 STN 之前初始化：

```cpp
void setShortLinkDebugIP(const std::string& _ip, unsigned short _port)
{
	mars::stn::SetShortlinkSvrAddr(_port, _ip);
}
void setShortLinkPort(unsigned short _port)
{
	mars::stn::SetShortlinkSvrAddr(_port, "");
}
void setLongLinkAddress(const std::string& _ip, unsigned short _port, const std::string& _debug_ip)
{
	vector<uint16_t> ports;
	ports.push_back(_port);
	mars::stn::SetLonglinkSvrAddr(_ip, ports, _debug_ip);
}

void Init()
{
	mars::stn::SetCallback(mars::stn::StnCallBack::Instance());
	mars::app::SetCallback(mars::app::AppCallBack::Instance());
	mars::baseevent::OnCreate();

	//todo
	//mars::stn::SetClientVersion(version);
	//setShortLinkDebugIP(...)
	//setLongLinkAddress(...)

	mars::baseevent::OnForeground(true);
	mars::stn::MakesureLonglinkConnected();
}
```

初始化顺序不一定要严格遵守上述代码的顺序，但在初始化时首先要调用 setCallBack 接口 (callback 文件的编写可以参考 demo)，再调用 Mars.init，最后再调用 onForeground 和 makesureLongLinkConnect，中间顺序可以随意更改。**注意：STN 默认是后台，所以初始化 STN 后需要主动调用一次**```BaseEvent.onForeground(true)```


需要释放 STN 或者退出程序时:

```cpp
mars::baseevent::OnDestroy();
```

## Support

还有其他问题？

1. 参看 [mars/sample](https://github.com/Tencent/mars/tree/master/samples)；
2. 阅读 [源码](https://github.com/Tencent/mars/tree/master)；
3. 阅读 [wiki](https://github.com/Tencent/mars/wiki) 或者 [FAQ](https://github.com/Tencent/mars/wiki/Mars-%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)；
4. 联系我们。

## Contributing
关于 Mars 分支管理、issue 以及 pr 规范，请阅读 [Mars Contributing Guide](https://github.com/Tencent/mars/blob/master/CONTRIBUTING.md)。

## License
Mars 使用的 MIT 协议，详细请参考 [LICENSE](https://github.com/Tencent/mars/blob/master/LICENSE)。
