## 此篇介紹 zxing project 使用

<a id="top"></a><br/>

|項目|
|:---|
| <a href="#s1">匯入 zxing 專案</a> |
| <a href="#s2">使用 zxing 掃描</a> |
| <a href="#s3">使用 垂直 掃描</a> |
| <a href="#s4">更改掃瞄範圍</a> |
| <a href="#s5">觸碰更改掃瞄範圍</a> | 

#### <a id="s1" href="#top">匯入 zxing 專案</a>
- 可去作者專欄下載範例檔 或 此頁的zip檔案
  - 作者專欄範例檔位置 : https://github.com/zxing/zxing/releases
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

#### <a id="s2" href="#top">使用 zxing 掃描</a>
- 加入 :
```java
IntentIntegrator integrator = new IntentIntegrator(activity);
integrator.setDesiredBarcodeFormats(IntentIntegrator.ONE_D_CODE_TYPES);
integrator.setPrompt("掃描條碼");
integrator.setCameraId(0);
integrator.setBeepEnabled(false);
integrator.setCaptureActivity(AnyOrientationCaptureActivity.class);
integrator.setOrientationLocked(false);
integrator.initiateScan();
```
- 接收掃描完結果 :
```java
 @Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    IntentResult result = IntentIntegrator.parseActivityResult(requestCode, resultCode, data);
    if(result != null) {
        if(result.getContents() == null) {return;}
        else {      
            Log.i("tag",result.getContents());//掃描完結果
        }
    } else {
        // This is important, otherwise the result will not be passed to the fragment
        super.onActivityResult(requestCode, resultCode, data);
    }
}
```

#### <a id="s3" href="#top">使用垂直掃描</a>
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
       
#### <a id="s4" href="#top">更改掃瞄範圍</a>
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

#### <a id="s5" href="#top">觸碰改變掃瞄範圍</a>
- 找到ViewfinderView.java
- 新增宣告觸碰位置 : ```private int gTouchX1, gTouchY1, gTouchX2, gTouchY2;```
- 得到觸碰座標 :
```java
private OnTouchListener handleTouch = new OnTouchListener() {
        @Override
        public boolean onTouch(View v, MotionEvent event) {
            int pointerCount = event.getPointerCount();  // 幾個觸控點
            gTouchX1 = 0;gTouchY1 = 0;gTouchX2 = 0;gTouchY2 = 0;//重新設定座標
            switch( event.getAction() )
            {
                case MotionEvent.ACTION_DOWN:  // 按下
                    break;
                case MotionEvent.ACTION_MOVE:  // 拖曳移動
                    if( pointerCount == 1 ){     // 單點觸控
                        gTouchX1 = (int) event.getX();
                        gTouchY1 = (int) event.getY();
                    }
                    else if( pointerCount == 2 ){  // 多點觸控
                        gTouchX1 = (int) event.getX(0);  // 第一個觸控點
                        gTouchY1 = (int) event.getY(0);
                        gTouchX2 = (int) event.getX(1);  // 第二個觸控點
                        gTouchY2 = (int) event.getY(1);
                    }
                    // 重繪, 再一次執行 onDraw 程序
                    invalidate();
                    break;
                case MotionEvent.ACTION_UP:    // 放開
                    break;
            }
            // TODO Auto-generated method stub
            return true;
        }
    };
```

- 於ViewfinderView.java 的 onDraw函數內加入(refreshSizes之前) : ```this.setOnTouchListener(handleTouch);```
- 於CameraPreview.java 新增宣告(四邊) : ```public int newTop = 0,newBottom = 0,newLeft = 0,newRight = 0;```
- 於getFramingRect()與getPreviewFramingRect()加入 :
```java
if(framingRect != null && newTop!=0 && newRight!=0 && newBottom!=0 && newLeft!=0){
    framingRect.set(newLeft,newTop,newRight,newBottom);
}
//與
if(previewFramingRect != null && newTop!=0 && newRight!=0 && newBottom!=0 && newLeft!=0){
    previewFramingRect.set(newLeft,newTop,newRight,newBottom);
}
```

- 改變ViewfinderView.java 的 refreshSizes() :
```java
protected void refreshSizes() {
   if(cameraPreview == null) {
          return;
   }
  int range = 20;//手指觸碰範圍
  Rect framingRect = cameraPreview.getFramingRect();
  Rect previewFramingRect = cameraPreview.getPreviewFramingRect();
  if(framingRect != null && previewFramingRect !=null){
      cameraPreview.newTop = framingRect.top;
      cameraPreview.newRight = framingRect.right;
      cameraPreview.newBottom = framingRect.bottom;
      cameraPreview.newLeft = framingRect.left;

      if(gTouchX1 != 0 && gTouchY1 >= (framingRect.top-range) && gTouchY1 <= (framingRect.bottom+range)){
          if(abs(gTouchX1-framingRect.left) < range){
              cameraPreview.newLeft = gTouchX1;
          }else if(abs(gTouchX1-framingRect.right) < range){
              cameraPreview.newRight = gTouchX1;
          }
      }
      if(gTouchY1 != 0 && gTouchX1 >= (framingRect.left-range) && gTouchX1 <= (framingRect.right+range)){
          if(abs(gTouchY1-framingRect.top) < range){
              cameraPreview.newTop = gTouchY1;
          }else if(abs(gTouchY1-framingRect.bottom) < range){
              cameraPreview.newBottom = gTouchY1;
          }
      }
      if(gTouchX2 != 0 && gTouchY2 >= (framingRect.top-range) && gTouchY2 <= (framingRect.bottom+range)){
          if(abs(gTouchX2-framingRect.left) < range){
              cameraPreview.newLeft = gTouchX2;
          }else if(abs(gTouchX2-framingRect.right) < range){
              cameraPreview.newRight = gTouchX2;
          }
      }
      if(gTouchY2 != 0 && gTouchX2 >= (framingRect.left-range) && gTouchX2 <= (framingRect.right+range)){
          if(abs(gTouchY2-framingRect.top) < range){
              cameraPreview.newTop = gTouchY2;
          }else if(abs(gTouchY2-framingRect.bottom) < range){
              cameraPreview.newBottom = gTouchY2;
          }
      }
      //重繪掃描區域
      framingRect = cameraPreview.getFramingRect();
      previewFramingRect = cameraPreview.getPreviewFramingRect();
  }

  if(framingRect != null && previewFramingRect != null) {
      this.framingRect = framingRect;
      this.previewFramingRect = previewFramingRect;
  }
}
```
