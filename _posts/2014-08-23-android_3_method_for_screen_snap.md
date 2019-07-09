---
title: Android 三种截图方法
tags: 技术 Android
--- 

全文提供了3种android系统上截图的方法，各有个的特点，总体来讲，对于用户态下的APP，当其他APP前台运行时，并没有权限去截取其他APP的图片。

<!--more-->

#### 方法1： getDrawingCache() App只能获取自己的截图。

    private Bitmap takeScreenShot(){
        Bitmap screenshot = null;
        View view = this.getRootView();
        // View view = this.getde
        view.setDrawingCacheEnabled(true);
        view.buildDrawingCache();
        screenshot = Bitmap.createBitmap(view.getDrawingCache());
        return screenshot;
    }


#### 方法2：反射android.view.SurfaceControl.screenshot 方法，此方法需要android.permission.READ_FRAME_BUFFER， read_frame_buffer只能够赋予系统应用。


    private Bitmap takeScreenShot2(){
        Bitmap screenshot = null;
        // Test for reflection method com.android.view.Surface/com.android.view.SurfaceControl.  to take screenshot
        try {
            Class<?> c = Class.forName("android.view.SurfaceControl");
            Object o = c.newInstance();
            Method m = c.getDeclaredMethod("screenshot", new Class[]{int.class, int.class});
            m.invoke(o, new Object[] {200,200});
        } catch (Exception e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        return screenshot;
    }


#### 方法3：使用screencap 命令。此方法在adb下不需要root，但是app使用需要 android.permission.WRITE_EXTERNAL_STORAGE，和android.permission.READ_FRAME_BUFFER， read_frame_buffer只能够赋予系统应用。

    private Bitmap takeScreenShot3(){
        Bitmap screenshot = null;
        // Test for "screencap /sdcard/test.png"
        try {
            Log.d(TAG, "run");
            Process p = Runtime.getRuntime().exec(new String[] {"screencap","/sdcard/test_sc.png"});
            BufferedReader br = new BufferedReader(new InputStreamReader(p.getErrorStream()));
        } catch (Exception e) {
            // TODO Auto-generated catch block
            Log.d(TAG, "except");
            e.printStackTrace();
        }
        return screenshot;
    }
