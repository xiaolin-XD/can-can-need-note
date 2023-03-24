当consumer001请求加入消费组consumer group A时，会首先通过算法(消费组名称hash的方式)选择_consumer_offset 其中一个分区，该分区的leader副本所在的broker，就是consumer group A的组协调者。
<font color="red">注意1：</font> 组协调者只与消费组有关

