## Overview

Facebook's [Stetho](http://facebook.github.io/stetho/) project enables you to use Chrome debugging tools to troubleshoot network traffic, database files, and view layouts.  With this library, you need to have an active emulator or device running, and you use will Chrome to connect to the device by typing `chrome://inspect`.

For network traffic, you can use the Network Inspector:

<img src="http://facebook.github.io/stetho/static/images/inspector-network.png" width="400"/>

Any SQLite database can also be inspected using the `Resources` -> `Web SQL` tab:

<img src="http://facebook.github.io/stetho/static/images/inspector-sqlite.png" width="400"/>

### Caveats

The third-party [[Android Async Http Client|Using-Android-Async-Http-Client]] library uses the Apache HTTP Client, which is not currently supported by Stetho as noted in this [issue](https://github.com/facebook/stetho/issues/116).  Troubleshooting networking issues works best with [[OkHttp|Using OkHttp]] or [[Retrofit|Consuming-APIs-with-Retrofit]].  Regardless, you can still use this library for SQLite database inspection.  If you need to inspect the network traffic with Async Http Client, see [this section](http://guides.codepath.com/android/Troubleshooting-API-calls#setting-up-a-proxy).

## Setup

Setup your `app/build.gradle` file:

```gradle
// Gradle dependency on Stetho
dependencies {
  implementation 'com.facebook.stetho:stetho:1.5.1'
}
```

Next, initialize Stetho inside your Application object:
```java
public class MyApplication extends Application {
  public void onCreate() {
    super.onCreate();
    Stetho.initializeWithDefaults(this);
  }
}
```
 Edit the manifest.xml file in your project. To let the Android operating system know that you have a custom Application class, add an attribute called  `android:name`  to the manifest’s application tag and set the value to the name of your custom Application class.
```xml
 <application
    ...
    android:name=".MyApplication"
    ...
    >
```

If you are also using Stetho with the [[OkHttp|Using OkHttp]] or [[Retrofit|Consuming-APIs-with-Retrofit]], make sure to include the OkHttp library as well:

```java
dependencies {
    // add below Stetho main dependency
    implementation 'com.facebook.stetho:stetho-okhttp3:1.5.1' // for OkHttp library
}
```

You will also need to add the `StethoInterceptor` when constructing the `OkHttpClient` instance:

```java
OkHttpClient client = new OkHttpClient.Builder()
    .addNetworkInterceptor(new StethoInterceptor())
    .build();
```

Start your emulator or device.  Then visit `chrome://inspect` on your Chrome desktop and your emulator device should appear.  Click on `Inspect` to launch a new window.  

<img src="http://facebook.github.io/stetho/static/images/inspector-discovery.png"/>

Click on the `Network` tab.  Now you can start watching network traffic between your emulator or device in real-time!