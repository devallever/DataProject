---
title: Android 根据一系列坐标点绘制贝塞尔曲线
date: 2017-07-30 16:54:18
tags:
 - Android
 - 自定义View
categories: [Android]
---


# 功能概述：
给定一系列坐标，绘制它的贝塞尔曲线，并有渐变效果，如图:

第一版：
![](https://github.com/devallever/DataProject/blob/master/data/bicycle/line-01.png?raw=true)


第二版：
![](https://github.com/devallever/DataProject/blob/master/data/bicycle/Screenshot_20170810-205520.png?raw=true)


第三版：
![](https://github.com/devallever/DataProject/blob/master/data/bicycle/line-03.png?raw=true)




# 详细实现

## 把一般坐标转为 Android中的视图坐标
因为现实中的坐标和Android中的坐标是不同的，Android中向下为Y轴正方向
```
        List<PointF> oldPointF1 = new ArrayList<>();
        oldPointF1.add(new PointF(1, 100));
        oldPointF1.add(new PointF(2, 200));
        oldPointF1.add(new PointF(3, 150));
        oldPointF1.add(new PointF(4, 200));
        oldPointF1.add(new PointF(5, 50));
        oldPointF1.add(new PointF(6, 150));
        oldPointF1.add(new PointF(7, 100));
        oldPointF1.add(new PointF(8, 200));
        oldPointF1.add(new PointF(9, 100));
        oldPointF1.add(new PointF(10, 150));
        oldPointF1.add(new PointF(11, 50));
        oldPointF1.add(new PointF(12, 100));
        List<PointF> pointFs1 = changePoint(oldPointF1);
```
```
    /**
     * 把一般坐标转为 Android中的视图坐标**/
    private List<PointF> changePoint(List<PointF> oldPointFs){
        List<PointF> pointFs = new ArrayList<>();
        //间隔，减去某个值是为了空出多余空间，为了画线以外，还要写坐标轴的值，除以坐标轴最大值(这里设为定值)
        //相当于缩小图像
        int intervalX = (getMeasuredWidth()-20)/12;
        int intervalY = (getMeasuredHeight()-20)/250;
        int height = getMeasuredHeight();
        PointF p;
        float x;
        float y;
        for (PointF pointF: oldPointFs){
            //最后的正负值是左移右移
            x = pointF.x * intervalX + 0;
            y = height - pointF.y * intervalY - 100;
            p = new PointF(x, y);
            pointFs.add(p);
        }
        return pointFs;
    }
```

## 计算控制点
参考文章：[根据多个点使用canvas贝赛尔曲线画一条平滑的曲线](http://www.zheng-hang.com/?id=43)

在这里仅把求控制点公式转换成java代码
首先封装控制点类：ControlPoint.java
```
package com.allever.bicycle.chartTest;

import android.graphics.PointF;

import java.util.ArrayList;
import java.util.List;

/**
 * Created by allever on 17-7-27.
 */

public class ControlPoint {
    private PointF conPoint1;
    private PointF conPoint2;

    public ControlPoint(PointF p1, PointF p2){
        this.conPoint1 = p1;
        this.conPoint2 = p2;
    }

    public PointF getConPoint1() {
        return conPoint1;
    }

    public void setConPoint1(PointF conPoint1) {
        this.conPoint1 = conPoint1;
    }

    public PointF getConPoint2() {
        return conPoint2;
    }

    public void setConPoint2(PointF conPoint2) {
        this.conPoint2 = conPoint2;
    }

    public static List<ControlPoint> getControlPointList(List<PointF> pointFs){
        List<ControlPoint> controlPoints = new ArrayList<>();

        PointF p1;
        PointF p2;
        float conP1x;
        float conP1y;
        float conP2x;
        float conP2y;
        for (int i=0; i<pointFs.size()-1;i++){

            if (i == 0){
                //第一断1曲线 控制点
                conP1x = pointFs.get(i).x + (pointFs.get(i + 1).x-pointFs.get(i).x)/4;
                conP1y = pointFs.get(i).y + (pointFs.get(i + 1).y-pointFs.get(i).y)/4;

                conP2x = pointFs.get(i+1).x - (pointFs.get(i + 2).x - pointFs.get(i).x)/4;
                conP2y = pointFs.get(i+1).y - (pointFs.get(i + 2).y - pointFs.get(i).y)/4;

            }else if (i == pointFs.size() - 2){
                //最后一段曲线 控制点
                conP1x = pointFs.get(i).x + (pointFs.get(i + 1).x-pointFs.get(i - 1).x)/4;
                conP1y = pointFs.get(i).y + (pointFs.get(i + 1).y-pointFs.get(i - 1).y)/4;

                conP2x = pointFs.get(i+1).x - (pointFs.get(i + 1).x - pointFs.get(i).x)/4;
                conP2y = pointFs.get(i+1).y - (pointFs.get(i + 1).y - pointFs.get(i).y)/4;
            }else {
                conP1x = pointFs.get(i).x + (pointFs.get(i + 1).x-pointFs.get(i - 1).x)/4;
                conP1y = pointFs.get(i).y + (pointFs.get(i + 1).y-pointFs.get(i - 1).y)/4;

                conP2x = pointFs.get(i+1).x - (pointFs.get(i + 2).x - pointFs.get(i).x)/4;
                conP2y = pointFs.get(i+1).y - (pointFs.get(i + 2).y - pointFs.get(i).y)/4;
            }

            p1 = new PointF(conP1x,conP1y);
            p2 = new PointF(conP2x,conP2y);

            ControlPoint controlPoint = new ControlPoint(p1, p2);
            controlPoints.add(controlPoint);
        }

        return controlPoints;
    }

}

```
其中getControlPointList()方法用来求每一断贝塞尔曲线的控制点，需要传入点坐标的List。一段曲线有两个控制点，因为使用三阶贝塞尔曲线，把每一段的控制点(两个)放到List中返回。
这里进行了判断，是因为第一段和左后一段求控制点与其他断有区别。可以查看上面文章公式。

### 绘制贝塞尔曲线
```
        Path mPath1 = new Path();
        //贝塞尔曲线获取控制点
        List<ControlPoint> controlPoints1 = ControlPoint.getControlPointList(pointFs1);
        for (int i=0; i<controlPoints1.size(); i++){
            if (i == 0){
                mPath1.moveTo(pointFs1.get(i).x,pointFs1.get(i).y);
            }
            //画三价贝塞尔曲线
            mPath1.cubicTo(
                    controlPoints1.get(i).getConPoint1().x,controlPoints1.get(i).getConPoint1().y,
                    controlPoints1.get(i).getConPoint2().x,controlPoints1.get(i).getConPoint2().y,
                    pointFs1.get(i+1).x, pointFs1.get(i+1).y
            );
        }
        LinearGradient mLinearGradient = new LinearGradient(
                0,
                0,
                0,
                getMeasuredHeight(),
                new int[]{
                        0xffffffff,
                        getResources().getColor(R.color.cyan),
                        getResources().getColor(R.color.colorPrimary),
                        getResources().getColor(R.color.colorPrimary),
                        getResources().getColor(R.color.colorPrimary)},
                null,
                Shader.TileMode.CLAMP
        );
        mPaint.setShader(mLinearGradient);

        canvas.drawPath(mPath1,mPaint);
        canvas.save();
```

### 颜色渐变
把画笔paint设置线性渐变LinearGradient，查看该构造方法，可以知道，
第一个参数，0，为x中开始渐变
第二个参数，0，为y轴开始渐变
第三个参数。0，为x轴结束渐变
第四个参数，为y轴结束渐变，
这里这样设置就是只在y轴上渐变
颜色数组：
依次是从上到下的渐变颜色
```
        LinearGradient mLinearGradient = new LinearGradient(
                0,
                0,
                0,
                getMeasuredHeight(),
                new int[]{
                        0xffffffff,
                        getResources().getColor(R.color.cyan),
                        getResources().getColor(R.color.colorPrimary),
                        getResources().getColor(R.color.colorPrimary),
                        getResources().getColor(R.color.colorPrimary)},
                null,
                Shader.TileMode.CLAMP
        );
        mPaint.setShader(mLinearGradient);
```

### 绘制坐标和分割线
```
        //画分割线和纵坐标
        Paint linePaint = new Paint();
        linePaint.setColor(0x66cccccc);
        linePaint.setTextSize(40f);
        //画5条分割线
        for (int i=0; i<5; i++){
            canvas.drawLine(90, getMeasuredHeight()-((getMeasuredHeight()/6) * i) - 140, getMeasuredWidth(), getMeasuredHeight()-((getMeasuredHeight()/6) * i) - 140,linePaint);
            canvas.drawText(45*i+"",10, getMeasuredHeight()-((getMeasuredHeight()/6) * i) - 130,linePaint);
            canvas.save();
        }


        //画横坐标
        linePaint.setTextSize(30f);
        for (int i=0; i<12; i++){
            canvas.drawText((i+1)+":00", (getMeasuredWidth())/13*i + 100, getMeasuredHeight()-((getMeasuredHeight()/5) * 0) - 60, linePaint);
            canvas.save();
        }
```

# 第二版更新
可以设置数据，曲线数量

## 曲线数据封装
LineDataSet.java
```
public class LineDataSet {
    private int color;
    private int[] gradientColors;
    private List<PointF> oldPointFsList;

    public int getColor() {
        return color;
    }

    public void setColor(int color) {
        this.color = color;
    }

    public int[] getGradientColors() {
        return gradientColors;
    }

    public void setGradientColors(int[] gradientColors) {
        this.gradientColors = gradientColors;
    }

    public List<PointF> getOldPointFsList() {
        return oldPointFsList;
    }

    public void setOldPointFsList(List<PointF> oldPointFsList) {
        this.oldPointFsList = oldPointFsList;
    }
}
```
> color: 指定曲线颜色
> grandientColors: 颜色数组,用来设定渐变效果
> oldPointFsList: 坐标点的集合

## 定义公开方法
```
    /**
     * 添加数据
     * @param lineDataSet 曲线参数集**/
    public void addLineData(LineDataSet lineDataSet){
        mLineDataSetList.add(lineDataSet);
        invalidate();
    }

    public void removeLine(LineDataSet lineDataSet){
        mLineDataSetList.remove(lineDataSet);
        invalidate();
    }
```
> mLineDataSetList: 是曲线参数集合, 用来存储每一条曲线.

## 循环绘制曲线
```
        //循环绘制曲线------------------------------------------------------------------------
mBezierPaint.setStyle(Paint.Style.STROKE);
mBezierPaint.setStrokeWidth(10f);//设置线宽
mBezierPaint.setAntiAlias(true);//去除锯齿

/**
* 根据给定参数绘制曲线
* **/
for (int position=0; position < mLineDataSetList.size(); position++){
	List<PointF> pointFList = changePoint(mLineDataSetList.get(position).getOldPointFsList());
	List<ControlPoint> controlPoints = ControlPoint.getControlPointList(pointFList);
	Path linePath = new Path();
	for (int i =0; i<controlPoints.size(); i++){
		if (i == 0){
			linePath.moveTo(pointFList.get(i).x,pointFList.get(i).y);
		}
		//画三价贝塞尔曲线
		linePath.cubicTo(
			controlPoints.get(i).getConPoint1().x,controlPoints.get(i).getConPoint1().y,
			controlPoints.get(i).getConPoint2().x,controlPoints.get(i).getConPoint2().y,
			pointFList.get(i+1).x, pointFList.get(i+1).y
		);

	}

	//设置颜色和渐变
	int lineColor = mLineDataSetList.get(position).getColor();
	mBezierPaint.setColor(lineColor);
	LinearGradient mLinearGradient = null;
	if (mLineDataSetList.get(position).getGradientColors() != null){
		//设置渐变
		mLinearGradient = new LinearGradient(
			0,
			0,
			0,
			getMeasuredHeight(),
			mLineDataSetList.get(position).getGradientColors(),
			null,
			Shader.TileMode.CLAMP
		);
	}else {
		mLinearGradient = new LinearGradient(
			0,
			0,
			0,
			getMeasuredHeight(),
			new int[]{lineColor,lineColor,lineColor,lineColor,lineColor},
			null,
			Shader.TileMode.CLAMP);
	}
	mBezierPaint.setShader(mLinearGradient);
	canvas.drawPath(linePath,mBezierPaint);
	canvas.save();
}

```

首先根据mLineDataSetList.size()数量决定绘制多少条曲线
首先, 把原来坐标转换为Android视图坐标
```
List<PointF> pointFList = changePoint(mLineDataSetList.get(position).getOldPointFsList());
```

更新后的转换方法如下:
```
    /**
     * 把一般坐标转为 Android中的视图坐标**/
    private List<PointF> changePoint(List<PointF> oldPointFs){
        List<PointF> pointFs = new ArrayList<>();
        //间隔，减去某个值是为了空出多余空间，为了画线以外，还要写坐标轴的值，除以坐标轴最大值
        //相当于缩小图像
        float intervalX = (getMeasuredWidth() - mMarginLeftRight * 2f)/11f;
        //float intervalY = (getMeasuredHeight() - mMarginTopBottom * 2f)/200f; 原始
        float intervalY = (getMeasuredHeight() - mMarginTopBottom * 2f)/200f-0.4f;
        Log.d(TAG, "intervalX = " + intervalX);
        Log.d(TAG, "intervalY = " + intervalY);
        int height = getMeasuredHeight();
        Log.d(TAG, "width = " + getMeasuredWidth());
        Log.d(TAG, "height = " +getMeasuredHeight());
        PointF p;
        float x;
        float y;
        for (int i = 0; i< oldPointFs.size(); i++){
            PointF pointF = oldPointFs.get(i);
            //最后的正负值是左移右移
            x = (pointF.x-1) * intervalX + mMarginLeftRight;
            //y = height - mMarginTopBottom - intervalY*pointF.y; 原始
            y = height - mMarginTopBottom - intervalY*pointF.y - 30f;
            p = new PointF(x, y);
            pointFs.add(p);
            Log.d(TAG, "oldX = " + pointF.x + "\toldY = " + pointF.y);
            Log.d(TAG, "newX = " + x + "\tnewY = " + y);
        }
/*        for (PointF pointF: oldPointFs){

        }*/
        return pointFs;
    }
```

然后根据转换后的坐标算出每一段贝塞尔曲线的控制点,以三阶曲线为例,一段曲线有两个控制点
然后绘制贝塞尔曲线路径
```
for (int i =0; i<controlPoints.size(); i++){
	if (i == 0){
		linePath.moveTo(pointFList.get(i).x,pointFList.get(i).y);
	}
	//画三价贝塞尔曲线
	linePath.cubicTo(
		controlPoints.get(i).getConPoint1().x,controlPoints.get(i).getConPoint1().y,
		controlPoints.get(i).getConPoint2().x,controlPoints.get(i).getConPoint2().y,
		pointFList.get(i+1).x,
		pointFList.get(i+1).y
	);
}
```
cubicT()的方法为
```
    /**
     * Add a cubic bezier from the last point, approaching control points
     * (x1,y1) and (x2,y2), and ending at (x3,y3). If no moveTo() call has been
     * made for this contour, the first point is automatically set to (0,0).
     *
     * @param x1 The x-coordinate of the 1st control point on a cubic curve
     * @param y1 The y-coordinate of the 1st control point on a cubic curve
     * @param x2 The x-coordinate of the 2nd control point on a cubic curve
     * @param y2 The y-coordinate of the 2nd control point on a cubic curve
     * @param x3 The x-coordinate of the end point on a cubic curve
     * @param y3 The y-coordinate of the end point on a cubic curve
     */
    public void cubicTo(float x1, float y1, float x2, float y2,
                        float x3, float y3) {
        isSimplePath = false;
        native_cubicTo(mNativePath, x1, y1, x2, y2, x3, y3);
    }
```
看参数解析是需要两个控制点,和一个结束点,因为一段曲线有两个点连接.一个起点,一个终点,这里的第三个坐标值就是终点坐标.

接着根据LineDataSet设置曲线的渐变和颜色
```
//设置颜色和渐变
int lineColor = mLineDataSetList.get(position).getColor();
mBezierPaint.setColor(lineColor);
LinearGradient mLinearGradient = null;
if (mLineDataSetList.get(position).getGradientColors() != null){
//设置渐变
	mLinearGradient = new LinearGradient(
		0,
		0,
		0,
		getMeasuredHeight(),
		mLineDataSetList.get(position).getGradientColors(),
		null,
		Shader.TileMode.CLAMP
	);
}else {
	mLinearGradient = new LinearGradient(
		0,
		0,
		0,
		getMeasuredHeight(),
		new int[]{lineColor,lineColor,lineColor,lineColor,lineColor},
		null,
		Shader.TileMode.CLAMP);
}
mBezierPaint.setShader(mLinearGradient);
```
因为可能不需要设置曲线渐变,所以对数组进行判空处理,有设置渐变的就用设置的,没有设置渐变的就去原来颜色做渐变

最后绘制贝塞尔曲线
```
canvas.drawPath(linePath,mBezierPaint);
canvas.save();
```
# 使用
在布局中引用控件
```
<com.allever.bicycle.chartTest.LineChartView
	android:id="@+id/id_custom_line_chart_view_activity_line_chart_view"
	android:layout_width="450dp"
	android:layout_height="250dp"
	android:layout_alignParentBottom="true"/>
```

在代码中设置曲线参数:坐标,颜色,渐变色数组
```
    public void initData(){
        List<PointF> speedPoint = new ArrayList<>();
        speedPoint.add(new PointF(1, 190));
        speedPoint.add(new PointF(2, 70));
        speedPoint.add(new PointF(3, 190));
        speedPoint.add(new PointF(4, 70));
        speedPoint.add(new PointF(5, 190));
        speedPoint.add(new PointF(6, 70));
        speedPoint.add(new PointF(7, 190));
        speedPoint.add(new PointF(8, 70));
        speedPoint.add(new PointF(9, 190));
        speedPoint.add(new PointF(10, 70));
        speedPoint.add(new PointF(11, 190));
        speedPoint.add(new PointF(12, 70));
        mSpeedLineDataSet = new LineDataSet();
        mSpeedLineDataSet.setOldPointFsList(speedPoint);
        mSpeedLineDataSet.setColor(getResources().getColor(R.color.colorPrimary));
        mSpeedLineDataSet.setGradientColors(new int[]{
                0xffffffff,
                getResources().getColor(R.color.cyan),
                getResources().getColor(R.color.colorPrimary),
                getResources().getColor(R.color.colorPrimary),
                getResources().getColor(R.color.colorPrimary)});
        mLineChartView.addLineData(mSpeedLineDataSet);


        List<PointF> runSpeedPoint = new ArrayList<>();
        runSpeedPoint.add(new PointF(1, 50));
        runSpeedPoint.add(new PointF(2, 200));
        runSpeedPoint.add(new PointF(3, 10));
        runSpeedPoint.add(new PointF(4, 200));
        runSpeedPoint.add(new PointF(5, 10));
        runSpeedPoint.add(new PointF(6, 200));
        runSpeedPoint.add(new PointF(7, 10));
        runSpeedPoint.add(new PointF(8, 200));
        runSpeedPoint.add(new PointF(9, 10));
        runSpeedPoint.add(new PointF(10, 200));
        runSpeedPoint.add(new PointF(11, 10));
        runSpeedPoint.add(new PointF(12, 50));
        mRunSpeedLineDataSet = new LineDataSet();
        mRunSpeedLineDataSet.setOldPointFsList(runSpeedPoint);
        mRunSpeedLineDataSet.setColor(Color.YELLOW);
        mRunSpeedLineDataSet.setGradientColors(new int[]{
                0xffffffff,
                Color.GREEN,
                Color.YELLOW,
                Color.YELLOW,
                Color.YELLOW});
        mLineChartView.addLineData(mRunSpeedLineDataSet);


        List<PointF> oppositionPoint = new ArrayList<>();
        oppositionPoint.add(new PointF(1, 100));
        oppositionPoint.add(new PointF(2, 150));
        oppositionPoint.add(new PointF(3, 30));
        oppositionPoint.add(new PointF(4, 60));
        oppositionPoint.add(new PointF(5, 80));
        oppositionPoint.add(new PointF(6, 130));
        oppositionPoint.add(new PointF(7, 200));
        oppositionPoint.add(new PointF(8, 160));
        oppositionPoint.add(new PointF(9, 20));
        oppositionPoint.add(new PointF(10, 85));
        oppositionPoint.add(new PointF(11, 200));
        oppositionPoint.add(new PointF(12, 70));
        mOppositionLineDataSet = new LineDataSet();
        mOppositionLineDataSet.setOldPointFsList(oppositionPoint);
        mOppositionLineDataSet.setColor(getResources().getColor(R.color.colorAccent));
        mLineChartView.addLineData(mOppositionLineDataSet);



        List<PointF> powerPoint = new ArrayList<>();
        powerPoint.add(new PointF(1, 100));
        powerPoint.add(new PointF(2, 140));
        powerPoint.add(new PointF(3, 100));
        powerPoint.add(new PointF(4, 140));
        powerPoint.add(new PointF(5, 100));
        powerPoint.add(new PointF(6, 140));
        powerPoint.add(new PointF(7, 100));
        powerPoint.add(new PointF(8, 140));
        powerPoint.add(new PointF(9, 100));
        powerPoint.add(new PointF(10, 140));
        powerPoint.add(new PointF(11, 100));
        powerPoint.add(new PointF(12, 140));
        mPowerLineDataSet = new LineDataSet();
        mPowerLineDataSet.setOldPointFsList(powerPoint);
        mPowerLineDataSet.setColor(Color.RED);
        mPowerLineDataSet.setGradientColors(new int[]{
                0xffffffff,
                Color.RED,
                Color.RED,
                Color.RED,
                Color.RED});
        mLineChartView.addLineData(mPowerLineDataSet);


        List<PointF> heartPoint = new ArrayList<>();
        heartPoint.add(new PointF(1, 30));
        heartPoint.add(new PointF(2, 170));
        heartPoint.add(new PointF(3, 30));
        heartPoint.add(new PointF(4, 170));
        heartPoint.add(new PointF(5, 30));
        heartPoint.add(new PointF(6, 170));
        heartPoint.add(new PointF(7, 30));
        heartPoint.add(new PointF(8, 170));
        heartPoint.add(new PointF(9, 30));
        heartPoint.add(new PointF(10, 170));
        heartPoint.add(new PointF(11, 30));
        heartPoint.add(new PointF(12, 170));
        mHeartLineDataSet = new LineDataSet();
        mHeartLineDataSet.setOldPointFsList(heartPoint);
        mHeartLineDataSet.setColor(Color.BLUE);
        mHeartLineDataSet.setGradientColors(new int[]{
                0xffffffff,
                Color.BLUE,
                Color.BLUE,
                Color.BLUE,
                Color.BLUE});
        mLineChartView.addLineData(mHeartLineDataSet);
    }
```
效果图:
![](https://github.com/devallever/DataProject/blob/master/data/bicycle/Screenshot_20170810-205520.png?raw=true)


# 第三版更新
可以设置坐标, 触控显示竖线和弹出Y值

## 绘制不同坐标轴的曲线
我的思路: 首先就是取消纵坐标的显示(横坐标一样的)，因为在一个界面绘制不同坐标系的图形，纵坐标显示是没意义的。
首先需要知道这条曲线一系列坐标中，纵坐标的最大值。
```
for (int i = 0; i < oldPointFs.size(); i++){
	yValue = oldPointFs.get(i).y;
	if (maxValueY < yValue) 
		maxValueY = yValue+ (yValue*0.1f);//最大值调高一点为了让曲线碰不到最高那条线，总保持一定距离
}
```
然后根据这个最大值求出间隔
```java
float intervalX = (getMeasuredWidth() - mMarginLeftRight * 2f)/11f;
//float intervalY = (getMeasuredHeight() - mMarginTopBottom * 2f)/200f; 原始
float intervalY = (getMeasuredHeight() - mMarginTopBottom * 2f)/maxValueY-0f;
```
这样的曲线不论纵坐标是多少，都不会超出界线了。

## 绘制触控标线
实现思路: 重写onTouchEvent方法，获取当手指按下时候事件，记录按下的x坐标值，判断x在哪个区域，然后在相应的区域绘制一条标线。
```
@Override
public boolean onTouchEvent(MotionEvent event) {
	float downX = event.getX();
	switch (event.getAction()){
		case MotionEvent.ACTION_DOWN:
			mDownX = downX;
			postInvalidateDelayed(50);
			//invalidate();
			break;
	}
        return super.onTouchEvent(event);
}
```
在onDraw()方法中判断是否有按下的记录，有则绘制标线，
```
        if (mDownX != -1f){
            mVerticalPaint.setColor(Color.WHITE);
            mVerticalPaint.setStrokeWidth(2f);
            final float intervalX = (mWidth - 2f * mMarginLeftRight) / 11f;//间隔
            Log.d(TAG, "onDraw: intervalX = " + intervalX);
            Log.d(TAG, "onDraw: mDownX = " + mDownX);
            Path trianglePath = new Path();
            if ((mDownX > (mMarginLeftRight)) && (mDownX < (mMarginLeftRight + intervalX/2f))){
                canvas.drawLine(
                        mMarginLeftRight,
                        mMarginTopBottom,
                        mMarginLeftRight,
                        mHeight - mMarginTopBottom,
                        mVerticalPaint);
                trianglePath.moveTo(mMarginLeftRight, mHeight - mMarginTopBottom - 20f);
                trianglePath.lineTo(mMarginLeftRight - 20f, mHeight - mMarginTopBottom);
                trianglePath.lineTo(mMarginLeftRight + 20f, mHeight - mMarginTopBottom);
                trianglePath.close();
                canvas.drawPath(trianglePath,mVerticalPaint);

            }
            else if ((mDownX > (mMarginLeftRight + intervalX/2f)) && (mDownX < ((mMarginLeftRight + intervalX/2f)+intervalX*1))){
                canvas.drawLine(
                        mMarginLeftRight + intervalX*1,
                        mMarginTopBottom,
                        mMarginLeftRight + intervalX*1,
                        mHeight - mMarginTopBottom,
                        mVerticalPaint);
                trianglePath.moveTo(mMarginLeftRight + intervalX*1, mHeight - mMarginTopBottom - 20f);
                trianglePath.lineTo(mMarginLeftRight + intervalX*1 - 20f, mHeight - mMarginTopBottom);
                trianglePath.lineTo(mMarginLeftRight + intervalX*1 + 20f, mHeight - mMarginTopBottom);
                trianglePath.close();
                canvas.drawPath(trianglePath,mVerticalPaint);
            }
            else if ((mDownX > ((mMarginLeftRight + intervalX/2f)+intervalX*1)) && (mDownX <((mMarginLeftRight + intervalX/2f)+intervalX*2))){
                canvas.drawLine(
                        mMarginLeftRight + intervalX*2,
                        mMarginTopBottom,
                        mMarginLeftRight + intervalX*2,
                        mHeight - mMarginTopBottom,
                        mVerticalPaint);
                trianglePath.moveTo(mMarginLeftRight + intervalX*2, mHeight - mMarginTopBottom - 20f);
                trianglePath.lineTo(mMarginLeftRight + intervalX*2 - 20f, mHeight - mMarginTopBottom);
                trianglePath.lineTo(mMarginLeftRight + intervalX*2 + 20f, mHeight - mMarginTopBottom);
                trianglePath.close();
                canvas.drawPath(trianglePath,mVerticalPaint);
            }
            else if ((mDownX > ((mMarginLeftRight + intervalX/2f)+intervalX*2)) && (mDownX <((mMarginLeftRight + intervalX/2f)+intervalX*3))){
                canvas.drawLine(
                        mMarginLeftRight + intervalX*3,
                        mMarginTopBottom,
                        mMarginLeftRight + intervalX*3,
                        mHeight - mMarginTopBottom,
                        mVerticalPaint);
                trianglePath.moveTo(mMarginLeftRight + intervalX*3, mHeight - mMarginTopBottom - 20f);
                trianglePath.lineTo(mMarginLeftRight + intervalX*3 - 20f, mHeight - mMarginTopBottom);
                trianglePath.lineTo(mMarginLeftRight + intervalX*3 + 20f, mHeight - mMarginTopBottom);
                trianglePath.close();
                canvas.drawPath(trianglePath,mVerticalPaint);
            }
            else if ((mDownX > ((mMarginLeftRight + intervalX/2f)+intervalX*3)) && (mDownX <((mMarginLeftRight + intervalX/2f)+intervalX*4))){
                canvas.drawLine(
                        mMarginLeftRight + intervalX*4,
                        mMarginTopBottom,
                        mMarginLeftRight + intervalX*4,
                        mHeight - mMarginTopBottom,
                        mVerticalPaint);
                trianglePath.moveTo(mMarginLeftRight + intervalX*4, mHeight - mMarginTopBottom - 20f);
                trianglePath.lineTo(mMarginLeftRight + intervalX*4 - 20f, mHeight - mMarginTopBottom);
                trianglePath.lineTo(mMarginLeftRight + intervalX*4 + 20f, mHeight - mMarginTopBottom);
                trianglePath.close();
                canvas.drawPath(trianglePath,mVerticalPaint);
            }
            else if ((mDownX > ((mMarginLeftRight + intervalX/2f)+intervalX*4)) &&
                    (mDownX <((mMarginLeftRight + intervalX/2f)+intervalX*5))){
                canvas.drawLine(
                        mMarginLeftRight + intervalX*5,
                        mMarginTopBottom,
                        mMarginLeftRight + intervalX*5,
                        mHeight - mMarginTopBottom,
                        mVerticalPaint);
                trianglePath.moveTo(mMarginLeftRight + intervalX*5, mHeight - mMarginTopBottom - 20f);
                trianglePath.lineTo(mMarginLeftRight + intervalX*5 - 20f, mHeight - mMarginTopBottom);
                trianglePath.lineTo(mMarginLeftRight + intervalX*5 + 20f, mHeight - mMarginTopBottom);
                trianglePath.close();
                canvas.drawPath(trianglePath,mVerticalPaint);
            }
            else if ((mDownX > ((mMarginLeftRight + intervalX/2f)+intervalX*5)) &&
                    (mDownX <((mMarginLeftRight + intervalX/2f)+intervalX*6))){
                canvas.drawLine(
                        mMarginLeftRight + intervalX*6,
                        mMarginTopBottom,
                        mMarginLeftRight + intervalX*6,
                        mHeight - mMarginTopBottom,
                        mVerticalPaint);
                trianglePath.moveTo(mMarginLeftRight + intervalX*6, mHeight - mMarginTopBottom - 20f);
                trianglePath.lineTo(mMarginLeftRight + intervalX*6 - 20f, mHeight - mMarginTopBottom);
                trianglePath.lineTo(mMarginLeftRight + intervalX*6 + 20f, mHeight - mMarginTopBottom);
                trianglePath.close();
                canvas.drawPath(trianglePath,mVerticalPaint);
            }
            else if ((mDownX > ((mMarginLeftRight + intervalX/2f)+intervalX*6)) &&
                    (mDownX <((mMarginLeftRight + intervalX/2f)+intervalX*7))){
                canvas.drawLine(
                        mMarginLeftRight + intervalX*7,
                        mMarginTopBottom,
                        mMarginLeftRight + intervalX*7,
                        mHeight - mMarginTopBottom,
                        mVerticalPaint);
                trianglePath.moveTo(mMarginLeftRight + intervalX*7, mHeight - mMarginTopBottom - 20f);
                trianglePath.lineTo(mMarginLeftRight + intervalX*7 - 20f, mHeight - mMarginTopBottom);
                trianglePath.lineTo(mMarginLeftRight + intervalX*7 + 20f, mHeight - mMarginTopBottom);
                trianglePath.close();
                canvas.drawPath(trianglePath,mVerticalPaint);
            }
            else if ((mDownX > ((mMarginLeftRight + intervalX/2f)+intervalX*7)) &&
                    (mDownX <((mMarginLeftRight + intervalX/2f)+intervalX*8))){
                canvas.drawLine(
                        mMarginLeftRight + intervalX*8,
                        mMarginTopBottom,
                        mMarginLeftRight + intervalX*8,
                        mHeight - mMarginTopBottom,
                        mVerticalPaint);
                trianglePath.moveTo(mMarginLeftRight + intervalX*8, mHeight - mMarginTopBottom - 20f);
                trianglePath.lineTo(mMarginLeftRight + intervalX*8 - 20f, mHeight - mMarginTopBottom);
                trianglePath.lineTo(mMarginLeftRight + intervalX*8 + 20f, mHeight - mMarginTopBottom);
                trianglePath.close();
                canvas.drawPath(trianglePath,mVerticalPaint);
            }
            else if ((mDownX > ((mMarginLeftRight + intervalX/2f)+intervalX*8)) &&
                    (mDownX <((mMarginLeftRight + intervalX/2f)+intervalX*9))){
                canvas.drawLine(
                        mMarginLeftRight + intervalX*9,
                        mMarginTopBottom,
                        mMarginLeftRight + intervalX*9,
                        mHeight - mMarginTopBottom,
                        mVerticalPaint);
                trianglePath.moveTo(mMarginLeftRight + intervalX*9, mHeight - mMarginTopBottom - 20f);
                trianglePath.lineTo(mMarginLeftRight + intervalX*9 - 20f, mHeight - mMarginTopBottom);
                trianglePath.lineTo(mMarginLeftRight + intervalX*9 + 20f, mHeight - mMarginTopBottom);
                trianglePath.close();
                canvas.drawPath(trianglePath,mVerticalPaint);
            }
            else if ((mDownX > ((mMarginLeftRight + intervalX/2f)+intervalX*9)) &&
                    (mDownX <((mMarginLeftRight + intervalX/2f)+intervalX*10))){
                canvas.drawLine(
                        mMarginLeftRight + intervalX*10,
                        mMarginTopBottom,
                        mMarginLeftRight + intervalX*10,
                        mHeight - mMarginTopBottom,
                        mVerticalPaint);
                trianglePath.moveTo(mMarginLeftRight + intervalX*10, mHeight - mMarginTopBottom - 20f);
                trianglePath.lineTo(mMarginLeftRight + intervalX*10 - 20f, mHeight - mMarginTopBottom);
                trianglePath.lineTo(mMarginLeftRight + intervalX*10 + 20f, mHeight - mMarginTopBottom);
                trianglePath.close();
                canvas.drawPath(trianglePath,mVerticalPaint);
            }
            else if ((mDownX > ((mMarginLeftRight + intervalX/2f)+intervalX*10)) &&
                    (mDownX <((mMarginLeftRight + intervalX/2f)+intervalX*11))){
                canvas.drawLine(
                        mMarginLeftRight + intervalX*11,
                        mMarginTopBottom,
                        mMarginLeftRight + intervalX*11,
                        mHeight - mMarginTopBottom,
                        mVerticalPaint);
                trianglePath.moveTo(mMarginLeftRight + intervalX*11, mHeight - mMarginTopBottom - 20f);
                trianglePath.lineTo(mMarginLeftRight + intervalX*11 - 20f, mHeight - mMarginTopBottom);
                trianglePath.lineTo(mMarginLeftRight + intervalX*11 + 20f, mHeight - mMarginTopBottom);
                trianglePath.close();
                canvas.drawPath(trianglePath,mVerticalPaint);
            }

            canvas.save();
            mDownX = -1f;
        }
```
关于区域的划分，例如横坐标起点1:00 到 最后终点12:00,可以分为12个区域，其中第一个区域和地十二个区域只是其他区域的一半，因为12个点可以分为11段。那么其他区域就是以坐标点为中心左右扩展半个区域，组成自己的区域，当按下的x坐标落在某个区域，就在相应区域的中心(原始坐标点的x值)，绘制一条竖线。

## 绘制标线底下的三角形
在前面绘制标线的基础上，已经知道了有关的坐标，三角形的中点就是x值，高就向上偏移几个像素，左右两点分别在x值基础上，向左向右偏移几个像素，就绘制出三角形了。
```
//代码同上
```

## 绘制标值
还记得List<LineDataSet> mLineDataSetList 保存的数据吗？保存了oldPointF原始坐标，可以根据原始坐标转换成坐标轴相应的坐标newPointF
oldPointF用来显示值，
newPointF用来确定绘制的位置。
```
        //绘制标注-----------------------------------------------------------------------------
        for (int position=0; position < mLineDataSetList.size(); position++){
            List<PointF> newPointF = changePoint(mLineDataSetList.get(position).getOldPointFsList(),"");
            //绘制标注值及
            List<PointF> oldPointF = mLineDataSetList.get(position).getOldPointFsList();//显示的数值
            //绘制标注数值及方框
            if (mDownX != -1f){
                final float intervalX = (mWidth - 2f * mMarginLeftRight) / 11f;
                if ((mDownX > (mMarginLeftRight)) &&
                        (mDownX < (mMarginLeftRight + intervalX/2f))){
                    mSelectedPositon = 0;
                }
                else if ((mDownX > (mMarginLeftRight + intervalX/2f)) &&
                        (mDownX < ((mMarginLeftRight + intervalX/2f)+intervalX*1))){
                    mSelectedPositon = 1;
                }
                else if ((mDownX > ((mMarginLeftRight + intervalX/2f)+intervalX*1)) &&
                        (mDownX <((mMarginLeftRight + intervalX/2f)+intervalX*2))){
                    mSelectedPositon = 2;
                }
                else if ((mDownX > ((mMarginLeftRight + intervalX/2f)+intervalX*2)) &&
                        (mDownX <((mMarginLeftRight + intervalX/2f)+intervalX*3))){
                    mSelectedPositon = 3;
                }
                else if ((mDownX > ((mMarginLeftRight + intervalX/2f)+intervalX*3)) &&
                        (mDownX <((mMarginLeftRight + intervalX/2f)+intervalX*4))){
                    mSelectedPositon = 4;
                }
                else if ((mDownX > ((mMarginLeftRight + intervalX/2f)+intervalX*4)) &&
                        (mDownX <((mMarginLeftRight + intervalX/2f)+intervalX*5))){
                    mSelectedPositon = 5;
                }
                else if ((mDownX > ((mMarginLeftRight + intervalX/2f)+intervalX*5)) &&
                        (mDownX <((mMarginLeftRight + intervalX/2f)+intervalX*6))){
                    mSelectedPositon = 6;
                }
                else if ((mDownX > ((mMarginLeftRight + intervalX/2f)+intervalX*6)) &&
                        (mDownX <((mMarginLeftRight + intervalX/2f)+intervalX*7))){
                    mSelectedPositon = 7;
                }
                else if ((mDownX > ((mMarginLeftRight + intervalX/2f)+intervalX*7)) &&
                        (mDownX <((mMarginLeftRight + intervalX/2f)+intervalX*8))){
                    mSelectedPositon = 8;
                }
                else if ((mDownX > ((mMarginLeftRight + intervalX/2f)+intervalX*8)) &&
                        (mDownX <((mMarginLeftRight + intervalX/2f)+intervalX*9))){
                    mSelectedPositon = 9;
                }
                else if ((mDownX > ((mMarginLeftRight + intervalX/2f)+intervalX*9)) &&
                        (mDownX <((mMarginLeftRight + intervalX/2f)+intervalX*10))){
                    mSelectedPositon = 10;
                }
                else if ((mDownX > ((mMarginLeftRight + intervalX/2f)+intervalX*10)) &&
                        (mDownX <((mMarginLeftRight + intervalX/2f)+intervalX*11))){
                    mSelectedPositon = 11;
                }
                if (mSelectedPositon == -1){
                    continue;
                }

                //设置颜色和渐变
                int lineColor = mLineDataSetList.get(position).getColor();
                LinearGradient mLinearGradient = null;
                if (mLineDataSetList.get(position).getGradientColors() != null){
                    //设置渐变
                    mLinearGradient = new LinearGradient(
                            0,
                            0,
                            0,
                            getMeasuredHeight(),
                            mLineDataSetList.get(position).getGradientColors(),
                            null,
                            Shader.TileMode.CLAMP
                    );
                }else {
                    mLinearGradient = new LinearGradient(
                            0,
                            0,
                            0,
                            getMeasuredHeight(),
                            new int[]{lineColor,lineColor,lineColor,lineColor,lineColor},
                            null,
                            Shader.TileMode.CLAMP);
                }
                mRectPaint.setShader(mLinearGradient);

                mRectPaint.setColor(lineColor);
                mRectPaint.setStyle(Paint.Style.FILL);
                mTextPaint.setColor(Color.WHITE);
                mTextPaint.setTextSize(DensityUtil.sp2px(mContext,12f));
                String drawText = oldPointF.get(mSelectedPositon).y + "";
                mTextPaint.setAntiAlias(true);//去除锯齿
                mRectPaint.setAntiAlias(true);//去除锯齿
                if (position % 2==0){
                    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                        canvas.drawRoundRect(
                                newPointF.get(mSelectedPositon).x,
                                newPointF.get(mSelectedPositon).y,
                                newPointF.get(mSelectedPositon).x + DensityUtil.dip2px(mContext,50),
                                newPointF.get(mSelectedPositon).y + DensityUtil.dip2px(mContext,30),
                                DensityUtil.dip2px(mContext,8f),
                                DensityUtil.dip2px(mContext,8f),
                                mRectPaint);
                        canvas.drawText(
                                drawText,
                                0,
                                drawText.length(),
                                newPointF.get(mSelectedPositon).x+DensityUtil.dip2px(mContext,5f),
                                newPointF.get(mSelectedPositon).y+DensityUtil.dip2px(mContext,20f),
                                mTextPaint);
                    }else {
                        canvas.drawRect(
                                newPointF.get(mSelectedPositon).x,
                                newPointF.get(mSelectedPositon).y,
                                newPointF.get(mSelectedPositon).x + DensityUtil.dip2px(mContext,50),
                                newPointF.get(mSelectedPositon).y + DensityUtil.dip2px(mContext,30),
                                mRectPaint);
                        canvas.drawText(
                                drawText,
                                0,
                                drawText.length(),
                                newPointF.get(mSelectedPositon).x+DensityUtil.dip2px(mContext,5f),
                                newPointF.get(mSelectedPositon).y+DensityUtil.dip2px(mContext,20f),
                                mTextPaint);
                    }
                }else{
                    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                        canvas.drawRoundRect(
                                newPointF.get(mSelectedPositon).x - DensityUtil.dip2px(mContext,50),
                                newPointF.get(mSelectedPositon).y - DensityUtil.dip2px(mContext,30),
                                newPointF.get(mSelectedPositon).x ,
                                newPointF.get(mSelectedPositon).y ,
                                DensityUtil.dip2px(mContext,8f),
                                DensityUtil.dip2px(mContext,8f),
                                mRectPaint);
                        canvas.drawText(
                                drawText,
                                0,
                                drawText.length(),
                                newPointF.get(mSelectedPositon).x - DensityUtil.dip2px(mContext,50) + DensityUtil.dip2px(mContext, 5f),
                                newPointF.get(mSelectedPositon).y - DensityUtil.dip2px(mContext,30) + DensityUtil.dip2px(mContext, 20f),
                                mTextPaint);
                    }else {
                        canvas.drawRect(
                                newPointF.get(mSelectedPositon).x - DensityUtil.dip2px(mContext,50),
                                newPointF.get(mSelectedPositon).y - DensityUtil.dip2px(mContext,30),
                                newPointF.get(mSelectedPositon).x ,
                                newPointF.get(mSelectedPositon).y ,
                                mRectPaint);
                        canvas.drawText(
                                drawText,
                                0,
                                drawText.length(),
                                newPointF.get(mSelectedPositon).x - DensityUtil.dip2px(mContext,50) + DensityUtil.dip2px(mContext, 5f),
                                newPointF.get(mSelectedPositon).y - DensityUtil.dip2px(mContext,30) + DensityUtil.dip2px(mContext, 20f),
                                mTextPaint);
                    }

                }
                canvas.save();
                mSelectedPositon = -1;

            }
        }
```
左右显示方法，根据position处2取余，0的就在标线右边绘制，1就在标线左边绘制
绘制圆角矩形是API21中的方法，如果版本低于21的就绘制一般矩形。

```
@TargetApi(Build.VERSION_CODES.LOLLIPOP)
@Override
protected void onDraw(Canvas canvas) {
	if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                        canvas.drawRoundRect(
                                newPointF.get(mSelectedPositon).x,
                                newPointF.get(mSelectedPositon).y,
                                newPointF.get(mSelectedPositon).x + DensityUtil.dip2px(mContext,50),
                                newPointF.get(mSelectedPositon).y + DensityUtil.dip2px(mContext,30),
                                DensityUtil.dip2px(mContext,8f),
                                DensityUtil.dip2px(mContext,8f),
                                mRectPaint);
                        canvas.drawText(
                                drawText,
                                0,
                                drawText.length(),
                                newPointF.get(mSelectedPositon).x+DensityUtil.dip2px(mContext,5f),
                                newPointF.get(mSelectedPositon).y+DensityUtil.dip2px(mContext,20f),
                                mTextPaint);
                    }else {
                        canvas.drawRect(
                                newPointF.get(mSelectedPositon).x,
                                newPointF.get(mSelectedPositon).y,
                                newPointF.get(mSelectedPositon).x + DensityUtil.dip2px(mContext,50),
                                newPointF.get(mSelectedPositon).y + DensityUtil.dip2px(mContext,30),
                                mRectPaint);
                        canvas.drawText(
                                drawText,
                                0,
                                drawText.length(),
                                newPointF.get(mSelectedPositon).x+DensityUtil.dip2px(mContext,5f),
                                newPointF.get(mSelectedPositon).y+DensityUtil.dip2px(mContext,20f),
                                mTextPaint);
                    }
}
```
效果图
![](https://github.com/devallever/DataProject/blob/master/data/bicycle/line-03.png?raw=true)

# 第四版更新
活动显示具体数值
```
@Override
public boolean onTouchEvent(MotionEvent event) {
	float downX = event.getX();
	switch (event.getAction()){
		case MotionEvent.ACTION_DOWN:
			mDownX = downX;
			//postInvalidateDelayed(50);
			Log.d(TAG, "onTouchEvent: ACTION_DOWN mDownX = " + mDownX);
			//invalidate();
			break;
		case MotionEvent.ACTION_MOVE:
			Log.d(TAG, "onTouchEvent: ACTION_MOVE mDownX = " + mDownX);
			mDownX = event.getX();
			invalidate();
			//postInvalidateDelayed(50);
			break;
		case MotionEvent.ACTION_UP:
			mDownX = event.getX();
			invalidate();
			Log.d(TAG, "onTouchEvent: ACTION_UP mDownX = " + mDownX);
			//postInvalidateDelayed(50);
			break;
	}
	//postInvalidateDelayed(50);
	return true;
}
```
是ACTION_MOVE的时候就记录坐标值,然后重绘