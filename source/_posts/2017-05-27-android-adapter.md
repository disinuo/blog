---
title: Android：ArrayAdapter工作原理【含源码分析】
date: 2017-05-25 16:51:42
tags:
    - java
    - android
---
### 前言
Adapter是个什么呢？~
它就相当于是MVC架构里的controller，负责数据和视图直接的联系。
而叫做`Adapter`也是因为应用了设计模式【适配器模式】的思想。它提供一个接口，使用者只管把数据丢给它，它负责将数据展示到指定的视图里
接下来就看看它的基本应用和原理~~~
### 基本应用
举个要展示一个数组的例子~
<!-- more -->
#### Activity创建Adapter并set给view
```java
String[] data = {
    "Mon 6/23 - Sunny - 31/17",
    "Tue 6/24 - Foggy - 21/8",
    "Wed 6/25 - Cloudy - 22/17",
};
List<String> weekForecast = new ArrayList<String>(Arrays.asList(data));
ArrayAdapter adapter = new ArrayAdapter<String>(
                        getActivity(), // 上下文
                        R.layout.list_item_forecast, // layout的ID
                        R.id.list_item_forecast_textview, // 想把单项数据放在里面的view的ID
                        weekForecast);//数据
  //展示数据的界面
ListView listView = (ListView) rootView.findViewById(R.id.listview_forecast);
listView.setAdapter(adapter);
```
#### 看看xml们长什么样
> 单项数据的layout，文件名是 list_item_forecast

```xml
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:minHeight="?android:attr/listPreferredItemHeight"
    android:gravity="center_vertical"
    android:id="@+id/list_item_forecast_textview" />
```
> ListView的layout

```xml
<ListView
       android:id="@+id/listview_forecast"
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       />
```
#### 运行结果
![](/image/2017-05-27-android-adapter/screenshot.png)
### 原理
- ArrayAdapter有一个`getView()`方法，在setAdapter之后调用这个方法可以获得逐项数据对应的view。（这个方法是隐式调用的）
- 要明确两个概念：
  - 一个屏幕能放下的item数量
  - item总量（就是一共有多少条数据）
- 一般item总量远大于一个屏幕能放下的数量~
#### getView()源码
```java
public @NonNull View getView(int position, @Nullable View convertView,
            @NonNull ViewGroup parent) {
  return createViewFromResource(mInflater, position, convertView, parent, mResource);
}
private @NonNull View createViewFromResource(@NonNull LayoutInflater inflater, int position,
            @Nullable View convertView, @NonNull ViewGroup parent, int resource) {
  final View view;
  final TextView text;
// 敲黑板啦！！--------下面是重点~~
// 这个convertView呢，就相当于是屏幕上的item
// 这里进行if判断，null代表这个convertView还没有被创建
  if (convertView == null) {
      //所以新创建一个view
      view = inflater.inflate(resource, parent, false);
  } else {
      // 不是null表示这个convertView被创建过啦，
      // 所以接下来要做的就是更新这个view里面的数据
      view = convertView;
  }
  //这个position是数据数组的序号，就是这个view要展示第position个数据~
  final T item = getItem(position);
  text = (TextView) view;//
  text.setText(item.toString());   
  return view;
}
```
#### 原理解析--实例
可能上面说的不大明白，这里再举个例子说明一下。
- 假如有100条数据，但是设备的一个屏幕只能显示下7条。
- 那一开始`getView`会被调用7次，生成7个view。
而这7次调用的`if(converView==null)`都是等于`true`的
- 当用户滚动屏幕，导致最上面的item滑出屏幕，第8个item进入屏幕时，`getView`会被调用一次。
而这次`if`判断等于false，因为7个view都已经存在了，所以这次getView的工作是把第8条数据的值赋值给第一个view并返回
### 总结
其实本来想找一下源码哪里调用了getView的。。但是失败了。。。看来看源码的功力还要修炼很久啊~~
----------
参考
[Android官方文档](https://developer.android.com/guide/components/activities.html)
[StackOverFlow---Linear Layout and weight in Android
](https://stackoverflow.com/questions/2698817/linear-layout-and-weight-in-android)
[](https://github.com/codepath/android_guides/wiki/Using-an-ArrayAdapter-with-ListView)
[Udacity课程](https://www.youtube.com/watch?v=2lcoB5-PCCw)
[StackOverFlow](https://stackoverflow.com/questions/10160475/when-getview-in-arrayadapter-is-called)
