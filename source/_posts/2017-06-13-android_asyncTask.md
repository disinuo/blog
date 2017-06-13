---
title: Android：AsyncTask用法
date: 2017-06-13 19:59:16
tags:
    - android
---
### 基本概念
AsyncTask是什么？---异步任务，听起来是用来执行一些比较耗时的操作的。
首先要知道Android的两个概念：主线程(又叫UI线程)和后台线程
- 主线程：是一个Android程序开始运行时默认启动的线程，主要用来显示界面，跟用户交互
- 后台线程：除了主线程以外的线程，加载数据等

所以，为了让用户体验好一点，为了程序能快速响应用户的操作，稍微耗时点的工作最好都不要放在主线程里啦~那我们要自己创线程吗？并不需要，Android给我们封装了一个类~就是这个 **AsyncTask** 啦~
<!-- more -->
### AsyncTask4个主要方法
因为AsyncTask是抽象类，必须要继承、重写关键方法才能用
他们都是回调函数，就是我们负责实现就好，Android负责调用
- onPreExecute
- doInBackground
- onProgressUpdate
- onPostExecute

看名字就能猜出来个大概意思了，只有`onProgressUpdate`不大好理解，现在逐一解释一下
#### onPreExecute
这个是最先执行的方法，是在主线程执行的，一般进行一些初始的配置
#### doInBackground(Params...)
第二个执行的方法，是在后台线程执行的（也是主线程的子线程），用来做耗时操作的主要部分~
#### onProgressUpdate(Progress...)
这个是给用户反馈进度条用的~，我们可以在`doInBackground`里随时调用AsyncTask的函数`publishProgress`,`publishProgress`函数的内部会调用`onProgressUpdate`~等下举个例子就清楚啦
#### onPostExecute(Result)
在doInBackground执行 **结束** 之后执行，它的参数Result是`doInbackground`的返回值

> 注意：除了doInBackground方法，另外3个都是在主线程执行的~

### 例子来啦
这个源于stackoverflow上一个大神给的例子~ 是一个模拟下载，显示进度百分数的过程
```java
public class AsyncTaskExample extends Fragment {
    protected TextView _percentField;
    protected InitTask _initTask;
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        _initTask = new InitTask();
        
        //这样就可以启动这个task啦
        _initTask.execute();
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        View rootView = inflater.inflate(R.layout.fragment_main, container, false);
        _percentField = (TextView)rootView.findViewById(R.id.percent_field);
        return rootView;
    }
    /**
    解释一下这里的3个参数，这是我们在继承AsyncTask时指定的三个泛型参数，
    分别是上面提到的三个回调方法所用的参数
    1. Params 在执行AsyncTask时需要传入的参数，即doInBackground的参数
    2. Progress 后台任务执行时，在界面上显示当前的进度的类型，即onProgressUpdate的参数
    3. Result 当任务执行完毕后，返回结果的类型，即onPostExecute的参数和doInBackground的返回值类型
    */
    protected class InitTask extends AsyncTask<Void, Integer, String> {

        @Override
        protected String doInBackground(Void... params) {
            int i = 0;
            while (i <= 50) {
                try {
                    Thread.sleep(50);
                    publishProgress(i);
                    i++;
                }
                catch (Exception e) {
                    Log.i("makemachine", e.getMessage());
                }
            }
            return "COMPLETE!";
        }
        @Override
        protected void onPreExecute() {
            Log.i("makemachine", "onPreExecute()");
            super.onPreExecute();
        }
        @Override
        protected void onProgressUpdate(Integer... values) {
            super.onProgressUpdate(values);
            Log.i("makemachine", "onProgressUpdate(): " + String.valueOf(values[0]));
            _percentField.setText((values[0] * 2) + "%");
            _percentField.setTextSize(values[0]);
        }

        @Override
        protected void onPostExecute(String result) {
            super.onPostExecute(result);
            Log.i("makemachine", "onPostExecute(): " + result);
            _percentField.setText(result);
            _percentField.setTextColor(0xFF69adea);
        }
    }
}
```
### 总结
以上就是AysncTask的基本用法啦~

----------
参考
[Udacity课程](https://classroom.udacity.com/courses/ud853/lessons/1469948762/concepts/15305685680923)
[StackOverflow](https://stackoverflow.com/questions/6450275/android-how-to-work-with-asynctasks-progressdialog)
[ 郭霖:Android AsyncTask完全解析，带你从源码的角度彻底理解](http://blog.csdn.net/guolin_blog/article/details/11711405)
[Android开发文档](https://developer.android.com/reference/android/os/AsyncTask.html)
[Android开发者：你真的会用AsyncTask吗](http://code.oneapm.com/android/2015/06/02/android1/)
