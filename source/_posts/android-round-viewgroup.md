---
title: Android实现圆角ViewGroup
date: 2017-04-04 16:44:59
tags: android
---
#### ViewGroup圆角效果
实现Android ViewGroup圆角效果，即在RelativeLayout,LinearLayout等ViewGroup中实现圆角，使其child view都在圆角范围内

``` xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="com.vibexie.testas.MainActivity">

    <com.vibexie.testas.RoundRelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:radius="20dp">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:background="@color/colorPrimary"></LinearLayout>

    </com.vibexie.testas.RoundRelativeLayout>

</RelativeLayout>

```
以上布局的效果如下：

![](http://qiniu.vibexie.com/blog/roundLayout_demo.png?imageView2/2/w/250)
<!-- more -->
#### 对于RoundRelativeLayout的实现
自定义RoundRelativeLayout

``` java
package com.vibexie.testas;

import android.content.Context;
import android.content.res.TypedArray;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.Path;
import android.graphics.PorterDuff;
import android.graphics.PorterDuffXfermode;
import android.graphics.RectF;
import android.util.AttributeSet;
import android.widget.RelativeLayout;

/**
 * Created by vibexie on 2017/4/4.
 */

public class RoundRelativeLayout extends RelativeLayout {
    //Default is 20
    private float topLeftRadius = 20f;
    private float topRightRadius = 20f;
    private float bottomLeftRadius = 20f;
    private float bottomRightRadius = 20f;

    private Paint roundPaint;
    private Paint imagePaint;

    public RoundRelativeLayout(Context context) {
        this(context, null);
    }

    public RoundRelativeLayout(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public RoundRelativeLayout(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        if (attrs != null) {
            TypedArray ta = context.obtainStyledAttributes(attrs, R.styleable.RoundRelativeLayout);
            float radius = ta.getDimension(R.styleable.RoundRelativeLayout_radius, 0);
            topLeftRadius = ta.getDimension(R.styleable.RoundRelativeLayout_topLeftRadius, radius);
            topRightRadius = ta.getDimension(R.styleable.RoundRelativeLayout_topRightRadius, radius);
            bottomLeftRadius = ta.getDimension(R.styleable.RoundRelativeLayout_bottomLeftRadius, radius);
            bottomRightRadius = ta.getDimension(R.styleable.RoundRelativeLayout_bottomRightRadius, radius);
            ta.recycle();
        }

        roundPaint = new Paint();
        roundPaint.setColor(Color.WHITE);
        roundPaint.setAntiAlias(true);
        roundPaint.setStyle(Paint.Style.FILL);
        roundPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.DST_OUT));

        imagePaint = new Paint();
        imagePaint.setXfermode(null);
    }

    @Override
    protected void dispatchDraw(Canvas canvas) {
        canvas.saveLayer(new RectF(0, 0, canvas.getWidth(), canvas.getHeight()), imagePaint, Canvas.ALL_SAVE_FLAG);
        super.dispatchDraw(canvas);
        drawTopLeft(canvas);
        drawTopRight(canvas);
        drawBottomLeft(canvas);
        drawBottomRight(canvas);
        canvas.restore();
    }

    private void drawTopLeft(Canvas canvas) {
        if (topLeftRadius > 0) {
            Path path = new Path();
            path.moveTo(0, topLeftRadius);
            path.lineTo(0, 0);
            path.lineTo(topLeftRadius, 0);
            path.arcTo(new RectF(0, 0, topLeftRadius * 2, topLeftRadius * 2),
                    -90, -90);
            path.close();
            canvas.drawPath(path, roundPaint);
        }
    }

    private void drawTopRight(Canvas canvas) {
        if (topRightRadius > 0) {
            int width = getWidth();
            Path path = new Path();
            path.moveTo(width - topRightRadius, 0);
            path.lineTo(width, 0);
            path.lineTo(width, topRightRadius);
            path.arcTo(new RectF(width - 2 * topRightRadius, 0, width,
                    topRightRadius * 2), 0, -90);
            path.close();
            canvas.drawPath(path, roundPaint);
        }
    }

    private void drawBottomLeft(Canvas canvas) {
        if (bottomLeftRadius > 0) {
            int height = getHeight();
            Path path = new Path();
            path.moveTo(0, height - bottomLeftRadius);
            path.lineTo(0, height);
            path.lineTo(bottomLeftRadius, height);
            path.arcTo(new RectF(0, height - 2 * bottomLeftRadius,
                    bottomLeftRadius * 2, height), 90, 90);
            path.close();
            canvas.drawPath(path, roundPaint);
        }
    }

    private void drawBottomRight(Canvas canvas) {
        if (bottomRightRadius > 0) {
            int height = getHeight();
            int width = getWidth();
            Path path = new Path();
            path.moveTo(width - bottomRightRadius, height);
            path.lineTo(width, height);
            path.lineTo(width, height - bottomRightRadius);
            path.arcTo(new RectF(width - 2 * bottomRightRadius, height - 2
                    * bottomRightRadius, width, height), 0, 90);
            path.close();
            canvas.drawPath(path, roundPaint);
        }
    }
}
```

对应attrs.xml

``` xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="RoundRelativeLayout">
        <attr name="radius" format="dimension" />
        <attr name="topLeftRadius" format="dimension" />
        <attr name="topRightRadius" format="dimension" />
        <attr name="bottomLeftRadius" format="dimension" />
        <attr name="bottomRightRadius" format="dimension" />
    </declare-styleable>
</resources>
```

#### 拓展
同理，继承LinearLayout等所有ViewGroup都是如此方式，替换以上代码的extends RelativeLayout即可。
