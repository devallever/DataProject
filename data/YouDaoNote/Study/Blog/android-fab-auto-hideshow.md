---
title: 滚动屏幕自动隐藏FloatingActionButton
date: 2017-04-24 09:56:01
tags:
 - Android
categories: [Android]
---

# 概述
FloatingActionButton可以说是Material Design 的标志之一了，但是却有很多人并不是很喜欢它，其中一条主要的原因就是FAB的存在挡住了要显示的内容，从而影响体验。
本文主要介绍对FAB两方面的优化，一方面是点击FAB弹出子菜单的特效，一方面是在滑动时自动隐藏FAB。最终的实现

# 原理
 它的显示与隐藏是根据AppBarLayout的Y值来决定的，我们知道如果按照最上面的方式定义主界面布局，列表滚动的时候toolbar会显示和隐藏，而toolbar是AppBarLayout的一部分，因此可以让Behavior依赖于AppBarLayout，当AppBarLayout变化的时候会调用onDependentViewChanged，然后在这里获取AppBarLayout的高度移动的距离，然后根据这个距离来判定FloatingActionButton上下移动的距离，从而实现了FloatingActionButton的显示和隐藏。这个实现方式我是在这里找到的： http://stackoverflow.com/questions/31457099/android-fab-to-hide-when-navigating-between-different-fragments-in-a-viewpager 


# 实现代码
```
public class FABVerticalBehavior extends FloatingActionButton.Behavior {
    private static final Interpolator INTERPOLATOR = new FastOutSlowInInterpolator();
    private boolean mIsAnimatingOut = false;

    public FABVerticalBehavior(Context context, AttributeSet attrs) {
        super();
    }

    @Override
    public boolean onStartNestedScroll(final CoordinatorLayout coordinatorLayout, final FloatingActionButton child,
                                       final View directTargetChild, final View target, final int nestedScrollAxes) {
        // Ensure we react to vertical scrolling
        return nestedScrollAxes == ViewCompat.SCROLL_AXIS_VERTICAL
                || super.onStartNestedScroll(coordinatorLayout, child, directTargetChild, target, nestedScrollAxes);
    }

    @Override
    public void onNestedScroll(final CoordinatorLayout coordinatorLayout, final FloatingActionButton child,
                               final View target, final int dxConsumed, final int dyConsumed,
                               final int dxUnconsumed, final int dyUnconsumed) {
        super.onNestedScroll(coordinatorLayout, child, target, dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed);
        if (dyConsumed > 0 && !this.mIsAnimatingOut && child.getVisibility() == View.VISIBLE) {
            // User scrolled down and the FAB is currently visible -> hide the FAB
            animateOut(child);
        } else if (dyConsumed < 0 && child.getVisibility() != View.VISIBLE) {
            // User scrolled up and the FAB is currently not visible -> show the FAB
            animateIn(child);
        }
    }

    // Same animation that FloatingActionButton.Behavior uses to hide the FAB when the AppBarLayout exits
    private void animateOut(final FloatingActionButton button) {
        if (Build.VERSION.SDK_INT >= 14) {
            ViewCompat.animate(button).translationY(button.getHeight() + getMarginBottom(button)).setInterpolator(INTERPOLATOR).withLayer()
                    .setListener(new ViewPropertyAnimatorListener() {
                        public void onAnimationStart(View view) {
                            FABVerticalBehavior.this.mIsAnimatingOut = true;
                        }

                        public void onAnimationCancel(View view) {
                            FABVerticalBehavior.this.mIsAnimatingOut = false;
                        }

                        public void onAnimationEnd(View view) {
                            FABVerticalBehavior.this.mIsAnimatingOut = false;
                            view.setVisibility(View.GONE);
                        }
                    }).start();
        } else {

        }
    }

    // Same animation that FloatingActionButton.Behavior uses to show the FAB when the AppBarLayout enters
    private void animateIn(FloatingActionButton button) {
        button.setVisibility(View.VISIBLE);
        if (Build.VERSION.SDK_INT >= 14) {
            ViewCompat.animate(button).translationY(0)
                    .setInterpolator(INTERPOLATOR).withLayer().setListener(null)
                    .start();
        } else {

        }
    }

    private int getMarginBottom(View v) {
        int marginBottom = 0;
        final ViewGroup.LayoutParams layoutParams = v.getLayoutParams();
        if (layoutParams instanceof ViewGroup.MarginLayoutParams) {
            marginBottom = ((ViewGroup.MarginLayoutParams) layoutParams).bottomMargin;
        }
        return marginBottom;
    }
}

```

# 注意
 - 在25.0.1以上版本的design包中，会有隐藏后不显示的Bug

> [参考这里](http://stackoverflow.com/questions/41142711/25-1-0-android-support-lib-is-breaking-fab-behavior)