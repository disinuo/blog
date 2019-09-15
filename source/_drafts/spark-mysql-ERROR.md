###用python 连接mysql不成功呀！！！

```shell
from pyspark import SparkContext
from pyspark.sql import SQLContext
sql_context = SQLContext(sc)
url = "jdbc:mysql://localhost:3306/adcDB"
df = sql_context.read.format("jdbc")\
    .option("url",url)\
    .option("dbtable","click")\
    .option("user",'root')\
    .option("password",'')\
    .option("driver", 'com.mysql.jdbc.Driver').load()
df.show()
# 输出这个！！！！
ERROR 1698 (28000): Access denied for user 'root'@'localhost'
# 一开始com.mysql.jdbc.Driver没装会报一个【找不到driver】的错，然后i啊在一个jdbs-connector-spark.jar放在$SPARK_HOME/jars/里就好了，
# 就变成了这个错！！！！！！！！！！！！！！！
# 确实在那个服务器上本地登录mysql的时候也得sudo
# 不然就会报这个`Access denied`的错
```
上面那个ERROR好啦!!参考链接在这里
[StackOverFlow](https://stackoverflow.com/questions/39281594/error-1698-28000-access-denied-for-user-rootlocalhost)
