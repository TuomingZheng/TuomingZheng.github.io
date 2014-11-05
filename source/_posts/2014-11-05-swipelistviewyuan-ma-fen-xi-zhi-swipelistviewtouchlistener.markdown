---
layout: post
title: "SwipeListView源码分析之SwipeListViewTouchListener"
date: 2014-11-05 23:13:25 +0800
comments: true
categories:  Android应用层开发笔记
---
>在分析完SwipeListView后，这个开源控件的内容也没剩下多少了。其中最重要最核心的功能模块是
SwipeListViewTouchListener的onTouch()方法，该方法处理用户的Touch事件并更新UI显示。

<p>
整个SwipeListViewTouchListener的源码剩下比较核心的东西就是<code>onTouch()</code>方法
和<code>move()</code>方法。其他的系列方法在介绍SwipeListView的时候已经都涉及了，在此不进
行赘述，主要按照用户Touch事件这个情形进行介绍。
</p>
<h1>onTouch()方法</h1>
<li>Down事件</li>
{% img center /images/SwipeListTouchListener1.png %}
<p>该方法首先根据View的<code>getLocationOnScreen()</code>方法获取SwipeListView在屏幕上的位置，
并根据Touch事件发生的位置（在屏幕上的位置）来得到Touch事件在SwipeListView组件上发生的位置。
再则通过遍历SwipeListView的所有child组件，通过View的<code>getHitRect()</code>获取每个child
组件的点击位置来判断Touch事件发生的坐标落在该Rect内。如果Touch事件在某个Item内发生则将判断
组件是否可用，在可用的状态下将设置该Item的FrontView和BackView的事件监听是否可用，并初始化
VelocityTracker。
</p>
<li>Move事件</li>
<p>接下来将分段介绍Move事件的处理逻辑<p>
{% img center /images/SwipeListTouchListener2.png %}
<p>
这段代码的逻辑比较清晰，起到了校正滑动距离的作用。在该段代码里获取SwipeListView的SwipeListViewListener
监听里通过<code>onChangeSwipeMode()</code>方法内设置的mode。在mode为none时直接将移动距离设置为0，在其他
几种情况下也将移动距离设置为0。他们的情况分别是:
<ul>
1):在当前滑动的Item处于open状态时如果mode是left，而用户却继续往左滑动时将距离设置为0<br/>
2):在当前滑动的Item处于open状态时如果mode是right，而用户却继续往右滑动时将距离设置为0<br/>
3):在当前滑动的Item处于非open状态时如果mode是left，而用户却往右滑动时将距离设置为0<br/>
4):在当前距离的Item处于非open状态时如果mode是right，而用户却往左滑动时将距离设置为0<br/>
</ul>
</p>
{% img center /images/SwipeListTouchListener3.png %}
<p>
这段代码的功能是设置mSwipeCurrentAction的值，根据当前Item是否处于open状态分别设置其状态值，并回调
SwipeListViewListener的<code>onStartClose()</code>和<code>onStartOpen()</code>方法。</br>
<ul>
1): 在当前Item处于open状态时，将mSwipeCurrentAction的值设置为SWIPE_ACTION_REVEAL。</br>
2): 在当前Item处于非open状态时，如果用户往右滑动并且右滑属性设为SWIPE_ACTION_DISMISS时将mSwipeCurrentAction
设置为SWIPE_ACTION_DISMISS。</br>
3): 在当前Item处于非open状态时，如果用户往左滑动组件并且左滑属性设置为SWIPE_ACTION_DISMISS是将mSwipeCurrentAction
设置为SWIPE_ACTION_DISMISS。</br>
4): 在当前Item处于非open状态时，如果用户往右滑动组件并且右滑属性为SWIPE_ACTION_CHOICE时将mSwipeCurrentAction
设置为SWIPE_ACTION_CHOICE。</br>
5): 在当前Item处于非open状态时，如果用户往左滑动组件并且左滑属性为SWIPE_ACTION_CHOICE时将mSwipeCurrentAction设置
SWIPE_ACTION_CHOICE。</br>
6): 在当前Item处于非open状态时，除以上几个情况其他情况都将mSwipeCurrentAction设置为SWIPE_ACTION_REVEAL。
</ul>
</p>
{% img center /images/SwipeListTouchListener4.png %}
<p>
该代码片段仅仅是在当前Item处于open状态时校正deltaX值，并调用<code>move()</code>方法，关于该方法的逻辑将在下面介绍。
</p>
<li>Up事件</li>
{% img center /images/SwipeListTouchListener5.png %}
该方法先校正velocityX的值，在用户滑动距离超过Item长度一半是将转变为切换。否则根据滑动速度大小决定是否是切换状态。
接下来将调用<code>generateAnimate()</code>方法进行动画操作。
<h1>move()方法</h1>
{% img center /images/SwipeListTouchListener6.png %}
<code>move()</code>方法的代码比较简单，主要是操作BackView的可见性并设置动画。
