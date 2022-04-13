#### TOPIC
```
topic/queueId(0)/多个磁盘文件
20字节[8offset][4size][8hashtag]
8offset -> 消息在commitLog中的物理偏移量
每个文件大概30万条数据 大约5.7M

没个consumer group（而不是consumer）都有自己的 offset 

```
#### 读取物理数据（如何实现高性能读取）：queue->commitLog
 ```
 1 queue中数据的offset结合数据数据长度 计算出偏移量 [ dataOffset=（offsset-1）*dataSize ] #[8offset][4size][8hashtag] 数据长度为 [4size] 而非queue中的一条数据长度
 2 再根据二分(或者类似查找方式查询到commitlog文件【因为commitLog以索引命名eg:00000000123(dataOffset对比文件索引找出数据位于哪个文件)】)
   commitLog 下标是 文件里的第一条消息的物理偏移量 
 3 读取具体数据：对应的commitLog 文件中 ：dataOffset + 文件大小 
 
 
 #说明：
  [8offset][4size][8hashtag] ：hashTag为 topic - tag 中的tag
 
 ```
 
 #### 写高性能原因
 [mapfile](https://www.jianshu.com/p/9bd672e1c5c1)
 ```
 1.commitLog 顺序写mapfile ( 映射pageCahce(内存) ) 【异步刷盘】
    <1> : broker进程 突然崩溃数据也不会丢失，原因：pageCache由linux/或者其他OS(操作系统)管理 
    <2> : 服务器故障宕机 pageCache会丢失
 2.consumerQueue  异步写入 不影响客户端
 ```
 #### Broker数据丢失场景
 ```
 broker 所在服务器宕机 pageCahce还没刷盘（异步刷盘） 会丢失
 解决：0丢失同步刷盘
 取舍：调低刷盘频率
 ```
 
 
 
