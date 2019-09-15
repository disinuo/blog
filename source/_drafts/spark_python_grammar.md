### map
### Row
- row = Row(name="Alice", age=11)
- row.name, row.age读取值
- 创建类和构造器： Person = Row("name", "age")
  - 然后直接这样创建一个Row：Person("Alice", 11)
- asDict，将Row转换为字典：Row(name="Alice", age=11).asDict()

### 任务的url
sc = SparkContext('spark://master:7077', 'wordcount')
### 求某列的max
os_df.groupby().max('os_count').collect()[0]['max(os_count)']
### 排序
from pyspark.sql.functions import col,desc,asc
os_df.filter((os_df.os==19)|(os_df.os==13)).sort(asc("app"),asc("channel")).show()

### 添加列
```python
def col_helper(num,col_val,count):
    if col_val==num: return count
    else: return 0
def device_1_helper(dv,count):return col_helper('1',dv,count)

udf_dv_0_helper = udf(device_0_helper,StringType())

device_df=spark_df.groupby('app','channel','date','hour','minute','device').count().withColumnRenamed("count", "device_count")
# os_df.createOrReplaceTempView('os_count')
device_df = device_df.sort(asc("app"),asc("channel"),asc('date'),asc('hour'),asc('minute'))
device_df = device_df.withColumn("device_0",udf_dv_0_helper("device","device_count"))\
    .drop('device','device_count')
device_df.show()
```
### groupby(columns) and select unique(another_col)
```python

ip_df=spark_df.groupby('app','channel','date','hour','minute').\
    agg(func.countDistinct('ip'))
ip_df.show()
````

### 处理date
```python
# spark_df.select(hour('click_time')).show()
spark_df = spark_df.withColumn("date",to_date('click_time'))
spark_df = spark_df.withColumn("hour",hour('click_time'))
spark_df = spark_df.withColumn("minute",minute('click_time'))
spark_df.show()
```
