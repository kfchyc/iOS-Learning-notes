# WKWebView学习踩坑整理

## URL初始化失败

1. initWithString传入参数不合法；
2. initWithString传入参数含有中文：转成UTF8编码。

## 内存不释放（崩溃）

1. webview的observer没有释放：dealloc中remove；
2. 与JS交互后因为messageHandler的循环引用问题，导致controller没有析构，dealloc方法始终不掉用：
   1. weakSelf：失败；
   2. 添加一层messageHandlerDelegate：该delegate不析构，失败；
   3. 在viewWillAppear和viewWillDisappear中添加和移除webview.config.userContentController的JS方法监控：成功。
3. iOS 9开始默认不允许使用http协议，使用http的url进行连接申请会被限制：info.plist中添加key：App Transport Security Settings -> Allow Arbitrary Loads 为YES。