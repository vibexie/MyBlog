---
title: 初窥SurfaceView
date: 2016-12-21 1:50:39
tags: android
---
平常自定义控件都是继承View，SurfaceView这个东西在工作中从来没有用过。所以，今天得搞明白SurfaceView是什么，如何使用。

如何使用SurfaceView？

首先SurfaceView也是一个View，它也有自己的生命周期。因为它需要另外一个线程来执行绘制操作，所以我们可以在它生命周期的初始化阶 段开辟一个新线程，然后开始执行绘制，当生命周期的结束阶段我们插入结束绘制线程的操作。这些是由其内部一个SurfaceHolder对象完成的。SurfaceHolder，顾名思义，它里面保存了一个对Surface对象的引用，而我们执行绘制方法本质上就是操控Surface。SurfaceHolder因为保存了对Surface的引用，所以使用它来处理Surface的生命周期。（说到底 SurfaceView的生命周期其实就是Surface的生命周期）例如使用 SurfaceHolder来处理生命周期的初始化。首先我们先看看建立一个SurfaceView的大概步骤，先看看一个Demo：

<!-- more -->

``` java
package com.vibexie.testas;

import android.content.Context;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.Rect;
import android.util.Log;
import android.view.SurfaceHolder;
import android.view.SurfaceView;

/**
 * Created by vibexie on 2016/12/21.
 */
public class MySurfaceView extends SurfaceView implements SurfaceHolder.Callback {
    private static final String TAG = "MySurfaceView";
    private SurfaceHolder sfh;
    private boolean threadFlag;
    private int counter;

    private MyThread myThread;

    class MyThread extends Thread {
        private Paint paint;
        private Rect rect;
        public MyThread() {
            // 创建画笔
            paint = new Paint();
            paint.setColor(Color.RED);
            paint.setTextSize(40);
            paint.setAntiAlias(true);

            rect = new Rect(200, 200, 400, 400);
        }

        @Override
        public void run() {
            while (threadFlag) {
                // 锁定画布，得到Canvas对象
                Canvas canvas = sfh.lockCanvas();

                if (sfh == null || canvas == null) {
                    threadFlag = false;
                    break;
                }

                // 设定Canvas对象的背景颜色
                canvas.drawColor(Color.GREEN);

                canvas.save();
                canvas.rotate(counter % 360, 300, 300);
                // 在canvas上绘制rect
                canvas.drawRect(rect, paint);
                canvas.restore();

                canvas.drawText("SurfaceView update..." + (counter++) % 360, 500, 300, paint);

                if (canvas != null) {
                    // 解除锁定，并提交修改内容，更新屏幕
                    sfh.unlockCanvasAndPost(canvas);
                }
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            }

            super.run();
        }
    }

    public MySurfaceView(Context context) {
        super(context);
        // TODO Auto-generated constructor stub
        myThread = new MyThread();

        // 通过SurfaceView获得SurfaceHolder对象
        sfh = this.getHolder();
        // 为SurfaceHolder添加回调结构SurfaceHolder.Callback
        sfh.addCallback(this);
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
        Log.i(TAG, "surfaceChanged");
    }

    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        Log.i(TAG, "surfaceCreated");
        counter = 0;
        threadFlag = true;
        myThread.start();
    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        Log.i(TAG, "surfaceDestroyed");
        threadFlag = false;
    }
}
```

效果如下，我们创建一个旋转的正方形，以每秒100次的速度重绘整个SurfaceView，我们发现实际效果正方形没有达到每秒旋转一度，即100帧每秒，显然，100帧每秒已经远远达到了系统的上限，所以我们看到的效果是以系统最高帧进行绘制的

![](http://qiniu.vibexie.com/blog/surfaceview-test.gif)

到这明白了，我们可以在SurfaceView中在新的线程中更新UI，同时可以控制帧的大小，当然是在系统最大帧范围之内。这是View类不可能做到的。

初步总结：
1. SurfaceView允许其他线程更新视图对象(执行绘制方法)而View不允许这么做，它只允许UI线程更新视图对象。
2. SurfaceView是放在其他最底层的视图层次中，所有其他视图层都在它上面，所以在它之上可以添加一些层，而且它不能是透明的。
3. 它执行动画的效率比View高，而且你可以控制帧数。
4. 因为它的定义和使用比View复杂，占用的资源也比较多，除非使用View不能完成，再用SurfaceView否则最好用View就可以。
