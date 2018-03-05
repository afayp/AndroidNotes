---
layout:     post
title:      "Android与javaScript的交互"
date:       2016-06-05 21:10:25
author:     "afayp"
catalog:    true
tags:
    - Android
---




WebView与js的交互包含两方面,一是在html中通过js调用java代码；二是在安卓java代码中调用js。

<!--more-->

## js调用Java

在Android开发中，能实现Js调用Java，有4种方法：

1. JavascriptInterface
2. WebViewClient.shouldOverrideUrlLoading()
3. WebChromeClient.onConsoleMessage()
4. WebChromeClient.onJsPrompt()

### 1.JavascriptInterface
这是Android提供的Js与Native通信的官方解决方案。  

首先Java代码要实现这么一个类，它的作用是提供给Js调用。
```
class DemoJavaScriptInterface {

    @JavascriptInterface
    public void showToast(String toast) {
        Toast.makeText(MainActivity.this, toast, Toast.LENGTH_SHORT).show();
    }
}
```
然后把这个类添加到WebView的JavascriptInterface中。 这个过程如下：  
`mWebView.addJavaScriptInterface(new DemoJavaScriptInterface(),"demo");`  

`new DemoJavaScriptInterface()`就是上面定义的类，demo就是这个类的别名。  
上面的代码执行后在html中js就能通过别名（这里是“demo”）来调用`DemoJavaScriptInterface`类中的任何方法了。  调用格式为
`window.jsInterfaceName.methodName(parameterValues)`  

另外因为安全问题，在Android4.2中（如果应用的android:targetSdkVersion为17+）JS只能访问带有@javaScriptInterface注解的java函数。4.2以下为了安全尽量不要调用addJavascriptInterface，需要另谋他法。

### 2.WebViewClient.shouldOverrideUrlLoading()
这个方法的作用是拦截所有WebView的Url跳转。页面可以构造一个特殊格式的Url跳转，shouldOverrideUrlLoading拦截Url后判断其格式，然后Native就能执行自身的逻辑了。
```java
public class CustomWebViewClient extends WebViewClient {

    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        if (isJsBridgeUrl(url)) {
            // JSbridge的处理逻辑
            return true;
        }
        return super.shouldOverrideUrlLoading(view, url);
    }
}
```

### 3.WebChromeClient.onConsoleMessage()
这是Android提供给Js调试在Native代码里面打印日志信息的API，同时这也成了其中一种Js与Native代码通信的方法。在Js代码中调用console.log(‘xxx’)方法。
`console.log('log message that is going to native code')`
就会在Native代码的WebChromeClient.consoleMessage()中得到回调。consoleMessage.message()获得的正是Js代码console.log(‘xxx’)的内容。
```java
public class CustomWebChromeClient extends WebChromeClient {

    @Override
    public boolean onConsoleMessage(ConsoleMessage consoleMessage) {
        super.onConsoleMessage(consoleMessage);
        String msg = consoleMessage.message();//JavaScript输入的Log内容
    }
}
```

### 4.WebChromeClient.onJsPrompt()

其实除了WebChromeClient.onJsPrompt()，还有WebChromeClient.onJsAlert()和WebChromeClient.onJsConfirm()。顾名思义，这三个Js给Native代码的回调接口的作用分别是展示提示信息，展示警告信息和展示确认信息。鉴于，alert和confirm在Js的使用率很高，所以JSBridge的解决方案中都倾向于选用onJsPrompt()。
Js中调用
`window.prompt(message, value)`
WebChromeClient.onJsPrompt()就会受到回调。onJsPrompt()方法的message参数的值正是Js的方法window.prompt()的message的值。
```java
public class CustomWebChromeClient extends WebChromeClient {

    @Override
    public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result) {
        // 处理JS 的调用逻辑
        result.confirm();
        return true;
    }
}
```

## Java调用JS
webView调用js的基本格式为
`webView.loadUrl(“javascript:methodName(parameterValues)”)`

### 调用js无参无返回值函数
```
String call = "javascript:sayHello()";
webView.loadUrl(call);
```
### 调用js有参无返回值函数
注意对于字符串作为参数值需要进行转义双引号。
```
String call = "javascript:alertMessage(\"" + "content" + "\")";
webView.loadUrl(call);
```
### 调用js有参数有返回值的函数

Android在4.4之前并没有提供直接调用js函数并获取值的方法，所以在此之前，常用的思路是 java调用js方法，js方法执行完毕，再次调用java代码将值返回。

1. Java调用js代码
```
String call = "javascript:sumToJava(1,2)";
webView.loadUrl(call);
```
2. js函数处理，并将结果通过调用java方法返回
```
function sumToJava(number1, number2){
       window.control.onSumResult(number1 + number2)
}
```
3. Java在回调方法中获取js函数返回值
```
@JavascriptInterface
public void onSumResult(int result) {
  Log.i(LOGTAG, "onSumResult result=" + result);
}
```

Android 4.4之后使用evaluateJavascript即可。举个栗子：  
js代码：
```
function getGreetings() {
      return 1;
}
```
java代码：
```
private void testEvaluateJavascript(WebView webView) {
  webView.evaluateJavascript("getGreetings()", new ValueCallback<String>() {

  @Override
  public void onReceiveValue(String value) {
      Log.i(LOGTAG, "onReceiveValue value=" + value);
  }});
}
```

注意:

- 上面限定了结果返回结果为String，对于简单的类型会尝试转换成字符串返回，对于复杂的数据类型，建议以字符串形式的json返回。
- evaluateJavascript方法必须在UI线程（主线程）调用，因此onReceiveValue也执行在主线程。

## 完整栗子

Java代码：
```
public class MainActivity extends Activity {
  private static final String LOGTAG = "MainActivity";
  @SuppressLint("JavascriptInterface")
  @Override
  protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity_main);
      final WebView myWebView = (WebView) findViewById(R.id.myWebView);
      WebSettings settings = myWebView.getSettings();
      settings.setJavaScriptEnabled(true);
      myWebView.addJavascriptInterface(new JsInteration(), "control");
      myWebView.setWebChromeClient(new WebChromeClient() {});
      myWebView.setWebViewClient(new WebViewClient() {

          @Override
          public void onPageFinished(WebView view, String url) {
              super.onPageFinished(view, url);
              testMethod(myWebView);
          }
          
      });
      myWebView.loadUrl("file:///android_asset/js_java_interaction.html");
  }
  
  private void testMethod(WebView webView) {
      String call = "javascript:sayHello()";
      
      call = "javascript:alertMessage(\"" + "content" + "\")";
      
      call = "javascript:toastMessage(\"" + "content" + "\")";
      
      call = "javascript:sumToJava(1,2)";
      webView.loadUrl(call);
   }
  
  public class JsInteration {
      
      @JavascriptInterface
      public void toastMessage(String message) {
          Toast.makeText(getApplicationContext(), message, Toast.LENGTH_LONG).show();
      }
      
      @JavascriptInterface
      public void onSumResult(int result) {
          Log.i(LOGTAG, "onSumResult result=" + result);
      }
  }

}
```
前端网页代码:
```
<html>
<script type="text/javascript">
    function sayHello() {
        alert("Hello")
    }

    function alertMessage(message) {
        alert(message)
    }

    function toastMessage(message) {
        window.control.toastMessage(message)
    }

    function sumToJava(number1, number2){
       window.control.onSumResult(number1 + number2)
    }
</script>
Java-Javascript Interaction In Android
</html>
```

## JsBridge
JSBridge，顾名思义，就是和js沟通的桥梁，主要思路也是利用前面提到过的WebViewClient.shouldOverrideUrlLoading()方法，不过将一些细节更加完善，写成框架，更方便使用。[JsBridge](https://github.com/lzyzsd/JsBridge)


## 参考
[Android中Java和JavaScript交互](http://droidyue.com/blog/2014/09/20/interaction-between-java-and-javascript-in-android/)  
[JsBridge实现JavaScript和Java的互相调用](http://mp.weixin.qq.com/s?__biz=MzI1NjEwMTM4OA==&mid=2651231789&idx=1&sn=f11650ad0e18ddc12ece6e7559d5084c&scene=1&srcid=0513BWa7HuHjzPAeManB3w6C#rd)  
[好好和h5沟通！几种常见的hybrid通信方式](http://zjutkz.net/2016/04/17/%E5%A5%BD%E5%A5%BD%E5%92%8Ch5%E6%B2%9F%E9%80%9A%EF%BC%81%E5%87%A0%E7%A7%8D%E5%B8%B8%E8%A7%81%E7%9A%84hybrid%E9%80%9A%E4%BF%A1%E6%96%B9%E5%BC%8F/)








