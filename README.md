# 贝塞尔花式曲线
### 什么是[贝塞尔曲线](https://zh.wikipedia.org/wiki/%E8%B2%9D%E8%8C%B2%E6%9B%B2%E7%B7%9A)？
简单的说就是可以把曲线，精确的用数学公式进行描述。如下图
![](https://upload-images.jianshu.io/upload_images/11184437-c957f177af750d03.gif?imageMogr2/auto-orient/strip)
[模拟N阶贝塞尔曲线(需翻墙)](http://myst729.github.io/bezier-curve/)

### 二阶贝塞尔曲线
二阶主要是三个点：起始点、终点、控制点。
创建一个自定义view
定义相应坐标点
```
    private float mStartPointX;//起点
    private float mStartPointY;

    private float mEndPointX;//终点
    private float mEndPointY;

    private float mFlagPointX;//控制点
    private float mFlagPointY;
```
重写onSizeChanged()方法，控制自定义view一些点的坐标，大小。
```
@Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mStartPointX = w / 4;
        mStartPointY = h / 2 - 200;

        mEndPointX = w * 3 / 4;
        mEndPointY = h / 2 - 200;

        mFlagPointX = w / 2;
        mFlagPointY = h / 2 - 300;

        mPath = new Path();
    }
```
##### 绘制贝塞尔曲线
![](https://upload-images.jianshu.io/upload_images/11184437-564d140980e86b0d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
`mPath.quadTo()和mPath.rQuadTo()`是绘制二阶贝塞尔曲线的api
`mPath.quadTo(float x1,float y1,float x2,float y2)` 是绝对坐标 
第一、二个参数是控制点坐标，第三、四个参数终点坐标
`mPath.rQuadTo()` 是相对坐标
```
@Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        mPath.reset();
        mPath.moveTo(mStartPointX, mStartPointY);
        mPath.quadTo(mFlagPointX, mFlagPointY, mEndPointX, mEndPointY);

        canvas.drawPoint(mStartPointX, mStartPointY, mPaintFlag);
        canvas.drawText("起点", mStartPointX, mStartPointY, mPaintFlagText);
        canvas.drawPoint(mEndPointX, mEndPointY, mPaintFlag);
        canvas.drawText("终点", mEndPointX, mEndPointY, mPaintFlagText);
        canvas.drawPoint(mFlagPointX, mFlagPointY, mPaintFlag);
        canvas.drawText("控制点", mFlagPointX, mFlagPointY, mPaintFlagText);
        canvas.drawLine(mStartPointX, mStartPointY, mFlagPointX, mFlagPointY, mPaintFlag);
        canvas.drawLine(mEndPointX, mEndPointY, mFlagPointX, mFlagPointY, mPaintFlag);

        canvas.drawPath(mPath, mPaintBezier);
    }
```
**绘制动态贝塞尔曲线**<br/>
![](https://upload-images.jianshu.io/upload_images/11184437-9a7a26398616e2ea.gif?imageMogr2/auto-orient/strip)

重写`onTouchEvent()`方法，跟随手指的移动改变控制点坐标，让贝塞尔曲线能跟随手指移动变换
```
@Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_MOVE:
                mFlagPointX = event.getX();
                mFlagPointY = event.getY();
                invalidate();
                break;
        }
        return true;
    }
```
### 三阶贝塞尔曲线
![](https://upload-images.jianshu.io/upload_images/11184437-674e2b2e078ddf9b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
相较于二阶，三阶只是多了一个控制点
所以需要定义俩个控制点

```
    private float mFlagPointOneX;
    private float mFlagPointOneY;
    private float mFlagPointTwoX;
    private float mFlagPointTwoY;
```
将绘制的api，修改为绘制三阶贝塞尔曲线的`cubicTo()`方法
`mPath.cubicTo(mFlagPointOneX, mFlagPointOneY, mFlagPointTwoX, mFlagPointTwoY, mEndPointX, mEndPointY);`
![](https://upload-images.jianshu.io/upload_images/11184437-8c049b9bbfabe1b7.gif?imageMogr2/auto-orient/strip)

由于是俩个控制点，需要考虑多点触控
如果是一个手指触控，只给第一个控制点赋值
俩个手指触控，则给第二个控制点赋值
```
@Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction() & MotionEvent.ACTION_MASK) {
            case MotionEvent.ACTION_POINTER_DOWN:
                isSecondPoint = true;
                break;
            case MotionEvent.ACTION_POINTER_UP:
                isSecondPoint = false;
                break;
            case MotionEvent.ACTION_MOVE:
                mFlagPointOneX = event.getX(0);
                mFlagPointOneY = event.getY(0);
                if (isSecondPoint) {
                    mFlagPointTwoX = event.getX(1);
                    mFlagPointTwoY = event.getY(1);
                }
                invalidate();
                break;
        }
        return true;
    }
```
### 实现路径变换动画效果
![](https://upload-images.jianshu.io/upload_images/11184437-a038bdc45fa4d28d.gif?imageMogr2/auto-orient/strip)


要让路径的形状发生改变，只需要修改控制点坐标，从而使整个形状发生变化。
需要定义一个属性动画，来改变点的坐标
```
mValueAnimator = ValueAnimator.ofFloat(mStartPointY, h);
//        mValueAnimator.setInterpolator(new BounceInterpolator());//回弹效果
        mValueAnimator.setDuration(3000);
        mValueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator valueAnimator) {
                mFlagPointOneY = (float) valueAnimator.getAnimatedValue();
                mFlagPointTwoY = (float) valueAnimator.getAnimatedValue();
                invalidate();
            }
        });
        setOnClickListener(this);
```
### 实现波浪运动动画效果
![](https://upload-images.jianshu.io/upload_images/11184437-342f4537fde399bf.gif?imageMogr2/auto-orient/strip)

实现波浪效果，首先要绘制一个静态的波浪。
主要有俩种方式，一种是三角函数，另外一种就是通过贝塞尔曲线。
这里使用贝塞尔曲线来实现。
需要俩个贝塞尔曲线组合来实现波浪效果。
![](https://upload-images.jianshu.io/upload_images/11184437-56762d178b50ff26.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
绘制上面半圆，它的起点也就是屏幕左侧的X1点，终点就是一半波形的长度
控制点大概是在波形中间偏上的位置，第二个曲线的控制点横坐标*2，纵坐标不变
```
mPath.quadTo(200,mStartPointY - 300,400,mStartPointY );
mPath.quadTo(600,mStartPointY + 300,800,mStartPointY );
```
![](https://upload-images.jianshu.io/upload_images/11184437-aeac454afe27b7e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
接下来就就是绘制出铺满屏幕的动态波形
首先要先确认下屏幕能容纳多少个波长，这里先定义几个变量。
```
    private int mWaveLength;//一个完整波形的长度
    private int mScreenHeight;//屏幕高度
    private int mScreenWidth;
    private int mCenterY; //记录下屏幕中间Y轴的坐标
    private int mWaveCount;//保存当前屏幕能容纳多少个波形
```
具体实现代码
```
@Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        mPath.reset();
        mPath.moveTo(-mWaveLength + mOffset, mCenterY);
        for (int i = 0; i < mWaveCount; i++) {
            mPath.quadTo(-mWaveLength * 3 / 4 + i * mWaveLength + mOffset, mCenterY + 60, -mWaveLength / 2 + i * mWaveLength + mOffset, mCenterY);
            mPath.quadTo(-mWaveLength / 4 + i * mWaveLength + mOffset, mCenterY - 60, i * mWaveLength + mOffset, mCenterY);
        }
        mPath.lineTo(mScreenWidth, mScreenHeight);
        mPath.lineTo(0, mScreenHeight);
        mPath.close();
        canvas.drawPath(mPath, mPaintBezier);
    }
```
用属性动画去做一个数值发生器，来控制贝塞尔曲线控制点坐标的偏移来实现动态波浪的效果
```
@Override
    public void onClick(View view) {
        mValueAnimator = ValueAnimator.ofInt(0, mWaveLength);
        mValueAnimator.setDuration(1000);
        mValueAnimator.setRepeatCount(ValueAnimator.INFINITE);
        mValueAnimator.setInterpolator(new LinearInterpolator());
        mValueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator valueAnimator) {
                mOffset = (int) valueAnimator.getAnimatedValue();
                invalidate();
            }
        });
        mValueAnimator.start();
    }
```
### 加入购物车抛物轨迹动画
![](https://upload-images.jianshu.io/upload_images/11184437-1461be36bb4b2fdc.gif?imageMogr2/auto-orient/strip)

首先是俩个公式，通过这俩个公式能完成贝塞尔曲线点坐标的计算方法。用来模拟物体的运动轨迹。
```
二阶贝塞尔曲线：B(t) = (1 - t)^2 * P0 + 2t * (1 - t) * P1 + t^2 * P2, t ∈ [0,1]
三阶贝塞尔曲线：B(t) = P0 * (1-t)^3 + 3 * P1 * t * (1-t)^2 + 3 * P2 * t^2 * (1-t) + P3 * t^3, t ∈ [0,1]
```
根据公式完成工具类
```
/**
     * B(t) = (1 - t)^2 * P0 + 2t * (1 - t) * P1 + t^2 * P2, t ∈ [0,1]
     *
     * @param t  曲线长度比例
     * @param p0 起始点
     * @param p1 控制点
     * @param p2 终止点
     * @return t对应的点
     */
    public static PointF CalculateBezierPointForQuadratic(float t, PointF p0, PointF p1, PointF p2) {
        PointF point = new PointF();
        float temp = 1 - t;
        point.x = temp * temp * p0.x + 2 * t * temp * p1.x + t * t * p2.x;
        point.y = temp * temp * p0.y + 2 * t * temp * p1.y + t * t * p2.y;
        return point;
    }

    /**
     * B(t) = P0 * (1-t)^3 + 3 * P1 * t * (1-t)^2 + 3 * P2 * t^2 * (1-t) + P3 * t^3, t ∈ [0,1]
     *
     * @param t  曲线长度比例
     * @param p0 起始点
     * @param p1 控制点1
     * @param p2 控制点2
     * @param p3 终止点
     * @return t对应的点
     */
    public static PointF CalculateBezierPointForCubic(float t, PointF p0, PointF p1, PointF p2, PointF p3) {
        PointF point = new PointF();
        float temp = 1 - t;
        point.x = p0.x * temp * temp * temp + 3 * p1.x * t * temp * temp + 3 * p2.x * t * t * temp + p3.x * t * t * t;
        point.y = p0.y * temp * temp * temp + 3 * p1.y * t * temp * temp + 3 * p2.y * t * t * temp + p3.y * t * t * t;
        return point;
    }
```
首先用二阶贝塞尔曲线绘制一个抛物线
通过上面的工具类来获取贝塞尔曲线点的坐标，从而实现抛物线轨迹的动画
```
@Override
    public void onClick(View view) {
        BezierEvaluator evaluator = new BezierEvaluator(new PointF(mFlagPointX, mFlagPointY));
        ValueAnimator animator = ValueAnimator.ofObject(evaluator,
                new PointF(mStartPointX, mStartPointY),
                new PointF(mEndPointX, mEndPointY));
        animator.setDuration(600);
        animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator valueAnimator) {
                PointF pointF = (PointF) valueAnimator.getAnimatedValue();
                mMovePointX = (int) pointF.x;
                mMovePointY = (int) pointF.y;
                invalidate();
            }
        });
        animator.setInterpolator(new AccelerateDecelerateInterpolator());
        animator.start();
    }
```