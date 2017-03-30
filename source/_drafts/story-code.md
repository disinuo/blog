```java
public class Parents{
    public void setCondition(String condition){
        if(condition.equals("坏人")){System.out.print("报警!");}
        if(condition.equals("吃土")){System.out.print("发红包~~");}
    }
}
class Child extends Parents{}
public static void main(String[] args){
    Child child=new Child();
    child.setCondition("坏人");//输出‘报警！’
}//爱是默默守护，无需多言。
```

```javascript
var array=[];
for(var i=0;i<10;i++){
    array[i]=function () {return '我已经看完第'+i+'章啦~~~';}
}
for(var j=0;j<array.length;j++){
    console.log(array[j]());
}//输出10次'我已经看完第10章啦~~~'

//如果看书做事每次只是不讲究方法的简单记录，自以为花了很多时间付出了很多，
//最后留下的。。。少到超乎你想像。。
//`我明明学过了呀` `我是失忆了吗` `我大概是得了脑癌。。。`

var array=[];
for(var i=0;i<10;i++){
    array[i]=function (i) {
        return function () {return '我已经看完第'+i+'章啦~~~';}
    }(i);//这是立即执行函数
}
for(var j=0;j<array.length;j++){
    console.log(array[j]());
}//依次输出'我已经看完第1章啦~~~'，'我已经看完第2章啦~~~'。。。一直到9
//接上一个故事。。~看！~如果每看完一章就立即回顾，加深理解~最后得到的也会是令自己满意的结果！



```
