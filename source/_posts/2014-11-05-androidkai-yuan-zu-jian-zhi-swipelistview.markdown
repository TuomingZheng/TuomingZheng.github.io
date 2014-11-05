---
layout: post
title: "Android开源组件之SwipeListView"
date: 2014-11-05 22:56:57 +0800
comments: true
categories: Android应用层开发笔记
---
><p>SwipeListView是最近比较火的一个开源组件，用于实现ListView元素側滑的功能。SwipeListView的使用还是比较简单的。</p>
<h1>自定义属性</h1>
<p>首先需要认识SwipeListView的几个自定义属性，他们定义在swipelistview__attrs.xml文件内。</p>
        <declare-styleable name="SwipeListView">
        	<attr name="swipeOpenOnLongPress" format="boolean" />
        	<attr name="swipeAnimationTime" format="integer" />
        	<attr name="swipeOffsetLeft" format="dimension" />
        	<attr name="swipeOffsetRight" format="dimension" />
        	<attr name="swipeCloseAllItemsWhenMoveList" format="boolean" />
        	<attr name="swipeFrontView" format="reference" />
        	<attr name="swipeBackView" format="reference" />
        	<attr name="swipeMode" format="enum">
            	<enum name="none" value="0" />
            	<enum name="both" value="1" />
            	<enum name="right" value="2" />
            	<enum name="left" value="3" />
        	</attr>
        	<attr name="swipeActionLeft" format="enum">
            	<enum name="reveal" value="0" />
            	<enum name="dismiss" value="1" />
            	<enum name="choice" value="2" />
        	</attr>
        	<attr name="swipeActionRight" format="enum">
            	<enum name="reveal" value="0" />
            	<enum name="dismiss" value="1" />
            	<enum name="choice" value="2" />
        	</attr>
        	<attr name="swipeDrawableChecked" format="reference" />
        	<attr name="swipeDrawableUnchecked" format="reference" />
        </declare-styleable>
<p>SwipeListView自定义属性具体说明如下：</p>
<ul>
	<li>第一个属性swipeOpenOnLongPress表示当前的SwipeListView是否支持长按某个元素进行滑动或则切换操作，他的取值为布尔类型。</li>
	<li>第二个属性swipeAnimationTime用来设置SwipeListView元素側滑或则进行checked状态切换的动画时间，如果不设置该属性的值SwipeListViewTouchListener
		会将其设置为Android系统内的标准动画时间（short类型）。</li>
	<li>第三个属性swipeOffsetLeft用于设置SwipeListView左滑停止后裸露在可见区域内的长度。</li>
	<li>第四个属性swipeOffsetRight为SwipeListView右滑动停止后裸露在可见区域内的长度。</li>
	<li>第五个属性swipeFrontView和swipeBackView分别指定swipeListView在滑动是处于前台和后台的组件。</li>
	<li>第六个属性swipeMode用于设置SwipeListView的滑动方向，其类型为枚举类型，分别可以取值为left(向左)、right（向右）、both（双向）和none(滑不动)。</li>
	<li>第七个属性swipeActionLeft和swipeActionRight分别用于指示SwipeListView左滑动和右滑动是的行为，其中reveal为FronView可见平滑滑动，
		dismiss为FrontView平滑滑动并伴随ListView元素alpha的渐变，最后choice取值则指定了FrontView滑动是切换checked状态值。</li>
	<li>最后一组属性swipeDrawableChecked和swipeDrawableUnchecked只有在mode取值为checked时有效，分别设置SwipeListView的元素在checked状态下的背景图片。</li>
</ul>

<h2>使用示例</h2> 
<p>具体在使用SwipeListView时只需要配置这几个属性值即可，在SwipeListView中也实现了代码设置对应属性的方法。其他的用法和ListView大同小异，唯一需要注意的是
SwipeListView对应的Adapter的Item布局必须实现FrontView和BackView并且FrontView最好罩在BackView之上。</p>


<li>SwipeListView在Activity布局文件内的声明：</li>

			<com.fortysevendeg.swipelistview.SwipeListView
        		android:id="@+id/swipelistview"
        		android:layout_width="match_parent"
        		android:layout_height="match_parent"
        		SwipeListView:swipeActionLeft="reveal"
        		SwipeListView:swipeActionRight="reveal"
        		SwipeListView:swipeAnimationTime="@android:integer/config_shortAnimTime"
        		SwipeListView:swipeBackView="@+id/back_item"
        		SwipeListView:swipeCloseAllItemsWhenMoveList="false"
        		SwipeListView:swipeDrawableChecked="@drawable/address_check_d"
        		SwipeListView:swipeDrawableUnchecked="@drawable/address_check"
        		SwipeListView:swipeFrontView="@+id/front_item"
        		SwipeListView:swipeMode="left"
        		SwipeListView:swipeOffsetLeft="10dp"
        		SwipeListView:swipeOffsetRight="10dp"
        		SwipeListView:swipeOpenOnLongPress="false"
        		android:divider="#FF0000" >
    		</com.fortysevendeg.swipelistview.SwipeListView>

<li> SwipeListView对应的Adapter的元素的布局：</li>
				<?xml version="1.0" encoding="utf-8"?>
				<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    					android:layout_width="match_parent"
    					android:layout_height="200dp" >

    					<RelativeLayout
        						android:id="@+id/back_item"
        						android:layout_width="match_parent"
        						android:layout_height="200dp" >

        					<Button
            					android:id="@+id/delete_button"
            					android:layout_width="wrap_content"
            					android:layout_height="wrap_content"
            					android:layout_alignParentRight="true"
            					android:text="Delete" />
    					</RelativeLayout>

    
						<RelativeLayout
        						android:id="@+id/front_item"
        						android:layout_width="match_parent"
        						android:layout_height="200dp"
        						android:background="#FF0000FF" >

        						<ImageView
            						android:id="@+id/account_icon_view"
            						android:layout_width="wrap_content"
            						android:layout_height="wrap_content"
            						android:layout_alignParentLeft="true"
            						android:layout_centerVertical="true"
            						android:layout_marginLeft="5dp"
            						android:layout_marginRight="5dp"
            						android:background="@null"
            						android:src="@drawable/ic_launcher" />

        						<TextView
            						android:id="@+id/account_time_view"
            						android:layout_width="wrap_content"
            						android:layout_height="wrap_content"
            						android:layout_alignParentRight="true"
            						android:layout_centerVertical="true"
            						android:gravity="center"
            						android:paddingLeft="5dp"
            						android:paddingRight="5dp"
            						android:textSize="18sp" />

        						<TextView
            						android:id="@+id/account_name_view"
            						android:layout_width="match_parent"
            						android:layout_height="wrap_content"
            						android:layout_toLeftOf="@id/account_time_view"
            						android:layout_toRightOf="@id/account_icon_view"
            						android:gravity="center"
            						android:textSize="18sp" />

        						<TextView
            						android:id="@+id/account_id_view"
            						android:layout_width="match_parent"
            						android:layout_height="wrap_content"
            						android:layout_below="@id/account_name_view"
            						android:layout_toLeftOf="@id/account_time_view"
            						android:layout_toRightOf="@id/account_icon_view"
            						android:gravity="center"
            						android:textSize="18sp" />
    					</RelativeLayout>

				</FrameLayout>
<li>注意事项：</li>
<p>在SwipeListView的Adapter元素的布局内必须实现SwipeListView的自定义属性frontView和backView，并且最好将frontView覆盖在backview是以实现滑动时backView渐显效果。
SwipeListView比较恶心的地方是监听器SwipeListViewListener内的<code>onChangeSwipeMode()</code>方法必须返回在activity布局文件内声明的action类型。</p>
{% img left /images/SwipeList.png %}
