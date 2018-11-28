## 此篇介紹 import zxing project 與 修改掃瞄範圍

#### 匯入 zxing 專案
- 請去作者下載範例檔 : https://github.com/zxing/zxing/releases
- 下載後請將 zxing-android-embedded 檔案複製至自己專案資料夾內
- 開啟 android studio 進入專案app
- 到 settings.gradle 新增:include ':zxing-android-embedded'
- 到 build.gradle(Module.app)新增:
```compile project(':zxing-android-embedded')```
- 到 build.gradle(project.app)新增:
```
 dependencies {
    classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.0'
}
allprojects {
    ext.androidBuildTools = '25.0.2'
    ext.androidTargetSdk = 25
    ext.zxingCore = 'com.google.zxing:core:3.3.0'
}
```
- 即可完成匯入

#### 使用垂直掃描
- 於自己專案新創java檔案
- 將此java類別 繼承 CaptureActivity
- 於 AndroidManifest 內新增:
```xml
<activity
    android:name=".繼承的類別"
    android:screenOrientation="fullSensor"
    android:stateNotNeeded="true"
    android:theme="@style/zxing_CaptureTheme"
    android:windowSoftInputMode="stateAlwaysHidden">
</activity>
```
- 最後在使用掃描的程式加入 : integrator.setOrientationLocked(false); 即可完成垂直掃描
- 參考網址 : http://blog.whomeninja.in/android-barcode-scanner-vertical-orientation-and-camera-flash/
       
#### 更改掃瞄範圍
- 到 CameraPreview.java 內的 calculateFrames 方法內
- 重新設定 framingRect
- 例 :
```java
private void calculateFrames() {
    if (containerSize == null || previewSize == null || displayConfiguration == null) {
        previewFramingRect = null;
        framingRect = null;
        surfaceRect = null;
        throw new IllegalStateException("containerSize or previewSize is not set yet");
    }

    int previewWidth = previewSize.width;
    int previewHeight = previewSize.height;

    int width = containerSize.width;
    int height = containerSize.height;

    surfaceRect = displayConfiguration.scalePreview(previewSize);

    Rect container = new Rect(0, 0, width, height);
    framingRect = calculateFramingRect(container, surfaceRect);

    //在這裡重設掃描區域與頁面範圍
    framingRect.set(framingRect.left-50,framingRect.top+100,framingRect.right+50,framingRect.bottom-110);

    Rect frameInPreview = new Rect(framingRect);
    frameInPreview.offset(-surfaceRect.left, -surfaceRect.top);

    previewFramingRect = new Rect(frameInPreview.left * previewWidth / surfaceRect.width(),
            frameInPreview.top * previewHeight / surfaceRect.height(),
            frameInPreview.right * previewWidth / surfaceRect.width(),
            frameInPreview.bottom * previewHeight / surfaceRect.height());

    if (previewFramingRect.width() <= 0 || previewFramingRect.height() <= 0) {
        previewFramingRect = null;
        framingRect = null;
        Log.w(TAG, "Preview frame is too small");
    } else {
        fireState.previewSized();
    }
}
```
       

      
