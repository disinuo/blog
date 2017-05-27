---
title: Android-Adapter工作原理【含源码分析】
date: 2017-05-25 16:51:42
tags:
    - java
    - android
---
### 坑s
<!-- more -->
#### 从输入框获取输入的值的时候要记得`toString`啊啊啊，不然啥都拿不到啊啊啊啊
```java
public class MyActivity extends AppCompatActivity {
    public void sendMessage(View view){
        Intent intent =new Intent(this,DisplayMessageActivity.class);
        EditText inputView=(EditText)findViewById(R.id.edit_message);
        intent.putExtra(EXTRA_MESSAGE,inputView.getText().toString());
        startActivity(intent);
    }
}
```
#### LinearLayout--weight
set the android:layout_width of the children to "0dp"
set the android:weightSum of the parent (edit: as Jason Moore noticed, this attribute is optional, because by default it is set to the children's layout_weight sum)
set the android:layout_weight of each child proportionally (e.g. weightSum="5", three children: layout_weight="1", layout_weight="3", layout_weight="1")
Example:

    <LinearLayout
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:weightSum="5">

    <Button
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:text="1" />

    <Button
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="3"
        android:text="2" />

    <Button
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:text="3" />

    </LinearLayout>
### 教程
#### 协调 Activity
当一个 Activity 启动另一个 Activity 时，它们都会体验到生命周期转变。第一个 Activity 暂停并停止（但如果它在后台仍然可见，则不会停止）时，同时系统会创建另一个 Activity。 如果这些 Activity 共用保存到磁盘或其他地方的数据，必须了解的是，在创建第二个 Activity 前，第一个 Activity 不会完全停止。更确切地说，启动第二个 Activity 的过程与停止第一个 Activity 的过程存在重叠。

生命周期回调的顺序经过明确定义，当两个 Activity 位于同一进程，并且由一个 Activity 启动另一个 Activity 时，其定义尤其明确。 以下是当 Activity A 启动 Activity B 时一系列操作的发生顺序：

Activity A 的 onPause() 方法执行。
Activity B 的 onCreate()、onStart() 和 onResume() 方法依次执行。（Activity B 现在具有用户焦点。）
然后，如果 Activity A 在屏幕上不再可见，则其 onStop() 方法执行。
您可以利用这种可预测的生命周期回调顺序管理从一个 Activity 到另一个 Activity 的信息转变。 例如，如果您必须在第一个 Activity 停止时向数据库写入数据，以便下一个 Activity 能够读取该数据，则应在 onPause() 而不是 onStop() 执行期间向数据库写入数据。

#### onCreate
onCreate里面尽量少做事情，避免程序启动太久都看不到界面
#### adapter
- clear()
  -  List<String> weekForecast = new ArrayList<String>(Arrays.asList(data));
  -  List<String> weekForecast = Arrays.asList(data);
  - ERROR
  ```bash
  java.lang.UnsupportedOperationException
        at java.util.AbstractList.remove(AbstractList.java:638)
        at java.util.AbstractList$SimpleListIterator.remove(AbstractList.java:75)
        at java.util.AbstractList.removeRange(AbstractList.java:658)
        at java.util.AbstractList.clear(AbstractList.java:466)
        at android.widget.ArrayAdapter.clear(ArrayAdapter.java:273)
  ```


```java
public @NonNull View getView(int position, @Nullable View convertView,
            @NonNull ViewGroup parent) {
    return createViewFromResource(mInflater, position, convertView, parent, mResource);
}

private @NonNull View createViewFromResource(@NonNull LayoutInflater inflater, int position,
            @Nullable View convertView, @NonNull ViewGroup parent, int resource) {
        final View view;
        final TextView text;
//这里！在从【闲置区】拿view
        if (convertView == null) {
            view = inflater.inflate(resource, parent, false);
        } else {
            view = convertView;
        }

        try {
            if (mFieldId == 0) {
                //  If no custom field is assigned, assume the whole resource is a TextView
                text = (TextView) view;
            } else {
                //  Otherwise, find the TextView field within the layout
                text = (TextView) view.findViewById(mFieldId);

                if (text == null) {
                    throw new RuntimeException("Failed to find view with ID "
                            + mContext.getResources().getResourceName(mFieldId)
                            + " in item layout");
                }
            }
        } catch (Clas
```

### How Adapter works
#### Show view
```java
setContentView(R.layout.activity_main);
------
@Override
public void setContentView(int resId) {
   ensureSubDecor();
   ViewGroup contentParent = (ViewGroup) mSubDecor.findViewById(android.R.id.content);
   contentParent.removeAllViews();
   LayoutInflater.from(mContext).inflate(resId, contentParent);
   mOriginalWindowCallback.onContentChanged();
}
--------
public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
  final AttributeSet attrs = Xml.asAttributeSet(parser);
// Temp is the root view that was found in the xml
   final View temp = createViewFromTag(root, name, inflaterContext, attrs);
   ViewGroup.LayoutParams params = null;

   if (root != null) {
       if (DEBUG) {
           System.out.println("Creating params from root: " +
                   root);
       }
       // Create layout params that match root, if supplied
       params = root.generateLayoutParams(attrs);
       if (!attachToRoot) {
           // Set the layout params for temp if we are not
           // attaching. (If we are, we use addView, below)
           temp.setLayoutParams(params);
       }
   }
 }
```
####
getView在setAdapter之后马上调用，且scroll的每滚动的时候（有item的变化时）getView被调用
----------
参考
[Android官方文档](https://developer.android.com/guide/components/activities.html)
[StackOverFlow---Linear Layout and weight in Android
](https://stackoverflow.com/questions/2698817/linear-layout-and-weight-in-android)
[](https://github.com/codepath/android_guides/wiki/Using-an-ArrayAdapter-with-ListView)
[](https://www.youtube.com/watch?v=2lcoB5-PCCw)
