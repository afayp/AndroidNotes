---
title: Android与javaScript的交
date: 2016-6-5 21:10:25
categories: Android
tags: [Android,JS]
---

WebView与js的交互包含两方面,一是在html中通过js调用java代码；二是在安卓java代码中调用js。
<!--more>
## js调用Java
首先WebView设置一个和js交互的接口（这里的接口是一般的意思，不是java中接口的含义），这个接口其实就是一个一般的类，同时为这个接口取一个别名。 这个过程如下：  
`mWebView.addJavaScriptInterface(new DemoJavaScriptInterface(),"demo");`  
`new DemoJavaScriptInterface()`就是这个接口，demo就是这个接口的别名。  
上面的代码执行后在html中js就能通过别名（这里是“demo”）来调用`DemoJavaScriptInterface`类中的任何方法了。  调用格式为
`window.jsInterfaceName.methodName(parameterValues)`  

另外因为安全问题，在Android4.2中（如果应用的android:targetSdkVersion为17+）JS只能访问带有@javaScriptInterface注解的java函数。4.2以下为了安全尽量不要调用addJavascriptInterface，需要另谋他法。

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

## 参考
[Android中Java和JavaScript交互](http://droidyue.com/blog/2014/09/20/interaction-between-java-and-javascript-in-android/)  
[Android与javaScript的交互](http://jijiaxin89.com/2016/04/08/Android%E4%B8%8EjavaScript%E7%9A%84%E4%BA%A4%E4%BA%92/)








