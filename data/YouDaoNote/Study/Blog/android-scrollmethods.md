# 滑动的方式

## layout()
```
@Override
public boolean onTouchEvent(MotionEvent event) {
        //getX()是获取View的坐标
        //获取触摸点的坐标
        float x = event.getX();
        float y = event.getY();
        Log.d(TAG, "onTouchEvent: x = " + x);
        int action = event.getAction();
        switch (action){
            case MotionEvent.ACTION_DOWN:
                lastX = x;
                lastY = y;
                break;
            case MotionEvent.ACTION_MOVE:
                float movedX = x - lastX;
                float movedY = y - lastY;
                Log.d(TAG, "onTouchEvent: lastX = " + lastX);
                Log.d(TAG, "onTouchEvent: movedX = " + movedX);
                layout((int)(getLeft()+movedX),(int)(getTop()+movedY),(int)(getRight()+movedX),(int)(getBottom()+movedY));
                break;
            case MotionEvent.ACTION_UP:
                break;
        }
        return true;
}

```

## Params方式
###使用Layout的LayoutParams方式
```
LinearLayout.LayoutParams layoutParams = (LinearLayout.LayoutParams)getLayoutParams();
layoutParams.leftMargin = (int)(getLeft()+movedX);
layoutParams.topMargin = (int)(getTop()+movedY);
setLayoutParams(layoutParams);
```

### 使用MarginLayoutParams的方式
```
ViewGroup.MarginLayoutParams marginLayoutParams = (ViewGroup.MarginLayoutParams)getLayoutParams();
marginLayoutParams.leftMargin = (int)(getLeft()+movedX);
marginLayoutParams.topMargin = (int)(getTop()+movedY);
setLayoutParams(marginLayoutParams);
```


## 使用ScrollBy方式
```
//ScrollBy方式移动的是View里面的内容，如果要移动View则在它的父布局使用scrollBy
((View)getParent()).scrollBy(-(int)movedX,-(int)movedY);
```

## 使用Scroller平滑移动
```
case MotionEvent.ACTION_UP:
	//使用Scroller平滑移动,手指离开屏幕是View返回原位
	int scrollX = ((View)getParent()).getScrollX();
	int scrollY = ((View)getParent()).getScrollY();
	scroller.startScroll(
		scrollX,
		scrollY,
		-scrollX,
		-scrollY
	);
	invalidate();
	break;
```

重写computeScroll()方法
```
    @Override
    public void computeScroll() {
        super.computeScroll();
        if (scroller.computeScrollOffset()){
            ((View)getParent()).scrollTo(
                    scroller.getCurrX(),
                    scroller.getCurrY()
            );
            invalidate();
        }

    }
```