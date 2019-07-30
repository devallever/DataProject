---
title: Android 自定义竞赛进度条
date: 2017-09-25 21:55:22
tags:
 - Android
 - 自定义View
categories :[Android]
---

# 需求
 1. 能显示用户当前进度,能显示其他4位用户的位置,
 2. 用户进度实时变化
 3. 当用户超越进度条最后一位(最右边),时,进度条向左滚动,并显示距离用户最近的用户位置
 
# 实现方案
 1. 绘制贝塞尔曲线
 2. 在贝塞尔曲线上绘制圆点和图片
 3. 使用坐标值循环实现波浪滚动
 

# 具体实现
## 绘制贝塞尔曲线
### 定义曲线的固定值
```
        mPointFList.add(new PointF(1,4.15f));
        mPointFList.add(new PointF(2,4.6f));
        mPointFList.add(new PointF(3,5.1f));
        mPointFList.add(new PointF(4,5.5f));
        mPointFList.add(new PointF(5,5.8f));
        mPointFList.add(new PointF(6,6.0f));
        mPointFList.add(new PointF(7,6.1f));
        mPointFList.add(new PointF(8,6.0f));
        mPointFList.add(new PointF(9,5.8f));
        mPointFList.add(new PointF(10,5.5f));
        mPointFList.add(new PointF(11,5.1f));
        mPointFList.add(new PointF(12,4.6f));

        mPointFList.add(new PointF(13,4.15f));
        mPointFList.add(new PointF(14,3.8f));
        mPointFList.add(new PointF(15,3.55f));
        mPointFList.add(new PointF(16,3.4f));
        mPointFList.add(new PointF(17,3.3f));
        mPointFList.add(new PointF(18,3.25f));
        mPointFList.add(new PointF(19,3.3f));
        mPointFList.add(new PointF(20,3.4f));
        mPointFList.add(new PointF(21,3.55f));
        mPointFList.add(new PointF(22,3.8f));

        mPointFList.add(new PointF(23,4.15f));
        mPointFList.add(new PointF(24,4.6f));
        mPointFList.add(new PointF(25,5.1f));
        mPointFList.add(new PointF(26,5.5f));
        mPointFList.add(new PointF(27,5.8f));
        mPointFList.add(new PointF(28,6.0f));
        mPointFList.add(new PointF(29,6.1f));
        mPointFList.add(new PointF(30,6.0f));
        mPointFList.add(new PointF(31,5.8f));
        mPointFList.add(new PointF(32,5.5f));
        mPointFList.add(new PointF(33,5.1f));
        mPointFList.add(new PointF(34,4.6f));

        mPointFList.add(new PointF(35,4.15f));
        mPointFList.add(new PointF(36,3.8f));
        mPointFList.add(new PointF(37,3.55f));
        mPointFList.add(new PointF(38,3.4f));
        mPointFList.add(new PointF(39,3.3f));
        mPointFList.add(new PointF(40,3.25f));
        mPointFList.add(new PointF(41,3.3f));
        mPointFList.add(new PointF(42,3.4f));
        mPointFList.add(new PointF(43,3.55f));
        mPointFList.add(new PointF(44,3.8f));

        mPointFList.add(new PointF(45,4.15f));
```
这里我定义了两个周期的点,每个周期的点越多,曲线就越精细

### 把原始点转化为Android坐标的点
```
    /**
     * 把一般坐标转为 Android中的视图坐标**/
    private List<PointF> changePoint(List<PointF> oldPointFs){
        List<PointF> pointFs = new ArrayList<>();
        float maxValueY = 0;
        float yValue = 0;
        for (int i = 0; i < oldPointFs.size(); i++){
            yValue = oldPointFs.get(i).y;
            if (maxValueY < yValue) maxValueY = yValue+ (yValue*0.1f);//最大值加上10
        }
        Log.d(TAG, "changePoint: maxValueY = " + maxValueY);
        //间隔，减去某个值是为了空出多余空间，为了画线以外，还要写坐标轴的值，除以坐标轴最大值
        //相当于缩小图像
        //int blockCount = oldPointFs.size() - 1;
        int blockCount = oldPointFs.size()-1;
        float intervalX = (getMeasuredWidth() - mMarginLeftRight * 2f)/blockCount;
        //float intervalY = (getMeasuredHeight() - mMarginTopBottom * 2f)/200f; 原始
        float intervalY = (getMeasuredHeight() - mMarginTopBottom * 2f)/maxValueY-0f;
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
            y = height - mMarginTopBottom - intervalY*pointF.y - DensityUtil.dip2px(mContext,5f) + DensityUtil.dip2px(mContext,20f);
            p = new PointF(x, y);
            pointFs.add(p);
            Log.d(TAG, "oldX = " + pointF.x + "\toldY = " + pointF.y);
            Log.d(TAG, "newX = " + x + "\tnewY = " + y);
        }
        return pointFs;
    }
```
这里只是一种方式,网上还有更好的方式,仅供参考

### 绘制贝塞尔曲线
```
        //绘制曲线
        mLinePaint.setColor(mContext.getResources().getColor(R.color.competition_line_alpha_gray));
        mLinePaint.setStyle(Paint.Style.STROKE);
        mLinePaint.setStrokeWidth(DensityUtil.dip2px(mContext,1.5f));//设置线宽
        mLinePaint.setAntiAlias(true);//去除锯齿
        //下面设置让笔触和连接处更加圆滑
        mLinePaint.setStrokeJoin(Paint.Join.ROUND);
        mLinePaint.setStrokeCap(Paint.Cap.ROUND);
        List<PointF> androidPointList = changePoint(mPointFList);
        List<ControlPoint> lightControlPoints = ControlPoint.getControlPointList(androidPointList);
        Path lightLinePath = new Path();
        for (int i =0; i<lightControlPoints.size(); i++){
            if (i == 0){
                lightLinePath.moveTo(androidPointList.get(i).x,androidPointList.get(i).y);
            }
            //画三价贝塞尔曲线路径
            //lightLinePath.rCubicTo();
            lightLinePath.cubicTo(
                    lightControlPoints.get(i).getConPoint1().x,lightControlPoints.get(i).getConPoint1().y,
                    lightControlPoints.get(i).getConPoint2().x,lightControlPoints.get(i).getConPoint2().y,
                    androidPointList.get(i+1).x, androidPointList.get(i+1).y
            );
        }
        float gradientFactor = 0f;//比例,根据用户当前位置设置渐变色的起始,用户左边为蓝色,右边为灰色
        //求用户所在位置占总距离的比例
        for (CompetitionData data: mCompetitionDataList){
            if (data.getCompetitionUserType()== CompetitionUserType.ME) {
                gradientFactor = data.getDistance()/mMaxDistance;
                break;
            }
        }
        LinearGradient linearGradient = new LinearGradient(
                0,
                0,
                getMeasuredWidth()*gradientFactor,
                0,
                new int[]{
                        mContext.getResources().getColor(R.color.color_default_light),
                        mContext.getResources().getColor(R.color.color_default_light),
                        mContext.getResources().getColor(R.color.color_default_blue),
                        mContext.getResources().getColor(R.color.color_default_blue),
                        mContext.getResources().getColor(R.color.color_default_blue),
                        mContext.getResources().getColor(R.color.color_default_blue),
                        mContext.getResources().getColor(R.color.color_default_blue),
                        mContext.getResources().getColor(R.color.color_default_blue),
                        mContext.getResources().getColor(R.color.color_default_blue),
                        mContext.getResources().getColor(R.color.color_default_blue),
                        mContext.getResources().getColor(R.color.competition_line_alpha_gray)},
                null,
                Shader.TileMode.CLAMP
        );
        mLinePaint.setShader(linearGradient);
        canvas.drawPath(lightLinePath,mLinePaint);
        canvas.save();
```


### 获取控制点的方法
```
    /**
     * 求控制点
     * @param pointFs Android坐标系坐标集合*/
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

                if (pointFs.size()<3){
                    //当仅有两个数据,先取最后一个数据
                    conP2x = pointFs.get(i+1).x - (pointFs.get(i + 1).x - pointFs.get(i).x)/4;
                    conP2y = pointFs.get(i+1).y - (pointFs.get(i + 1).y - pointFs.get(i).y)/4;
                }else {
                    conP2x = pointFs.get(i+1).x - (pointFs.get(i + 2).x - pointFs.get(i).x)/4;
                    conP2y = pointFs.get(i+1).y - (pointFs.get(i + 2).y - pointFs.get(i).y)/4;
                }

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
```

### 在曲线上绘制图片或圆点
```
        getShowedDataList();//显示4个数据
        if (mCompetitionDataList.size() == 0) return;
        //绘制圆形与文字
        int position;//数据点位置索引
        Bitmap bitmap;
        CompetitionData competitionData;
        float dXY; //头像偏移量
        Log.d(TAG, "onDraw: mMaxDistance = " + mMaxDistance);
        //
        for (int i=0; i<mShowedDataList.size(); i++){
            competitionData = mShowedDataList.get(i);
            Log.d(TAG, "onDraw: competitionData.getDistance() = " + competitionData.getDistance());
            Log.d(TAG, "onDraw: mPointFList.size() = " + mPointFList.size());
            position =(int) ((competitionData.getDistance()/mMaxDistance)*mPointFList.size());//求数据点在曲线上的索引
            if (position<0) position = 0;
            if (position>44) position = 44;
            if (competitionData.getCompetitionUserType() == CompetitionUserType.ME){
                //是当前用户,绘制Bitmap
                bitmap = BitmapFactory.decodeResource(getResources(),R.drawable.run_video_piont);
                dXY = DensityUtil.dip2px(mContext,14f);
                canvas.drawBitmap(bitmap,
                        androidPointList.get(position).x - dXY,
                        androidPointList.get(position).y - dXY,
                    mImagePaint);
            }else {
                //其他用户绘制圆点和排名
                dXY =  DensityUtil.dip2px(mContext,10f);
                float r = DensityUtil.dip2px(mContext,10f);
                mLinePaint.setStyle(Paint.Style.FILL);
                canvas.drawCircle(
                        androidPointList.get(position).x,
                        androidPointList.get(position).y,
                        r,mLinePaint);
                //绘制排名
                String rank = competitionData.getRank()+"";
                float dx = dx = DensityUtil.dip2px(mContext,6f);
                float dy = DensityUtil.dip2px(mContext,4f);
                if (competitionData.getRank() <10){
                    dx = DensityUtil.dip2px(mContext,3f);
                }else if (competitionData.getRank() >= 10 && competitionData.getRank() < 100){
                    dx = DensityUtil.dip2px(mContext,6f);
                }else {
                    dx = DensityUtil.dip2px(mContext,8f);
                }
                canvas.drawText(rank,0,rank.length(),
                        androidPointList.get(position).x -dx,
                        androidPointList.get(position).y + dy,
                        mRankPaint);
            }
            Log.d(TAG, "onDraw: position = " + position);
            canvas.save();
        }
```
### 获取列表中的特定用户
```
    /**
     * 只获取显示四个数据*/
    private void getShowedDataList(){
        mShowedDataList.clear();
        mShowedDataList.add(mCompetitionDataList.get(0));//第一个,用户
        int fromIndex = 1;
        Log.d(TAG, "getShowedDataList: mCurDataIndex = " + mCurDataIndex);
        int counter = 0;
        for (int i=mCurDataIndex;i>0;i--){
            if (counter < 4){
                fromIndex = i;
                counter ++;
            }
        }
        Log.d(TAG, "getShowedDataList: fromIndex = " + fromIndex);
        for (int j=fromIndex;j<=mCurDataIndex;j++){
            mShowedDataList.add(mCompetitionDataList.get(j));
            Log.d(TAG, "getShowedDataList: j = " + j);
        }
    }
```
当列表的数量小于

