##记录一些使用kafka中出现过的问题

#####问题描述：先produce数据，再取数据，对于刚创建的topic而言，这样的情况下第一次会拿不到数据
#####解决方案：设置可以通过props.put("auto.offset.reset", "smallest");从头获取数据。对于我现在使用的版本我使用了props.put("auto.offset.reset", "latest");
#####官方对auto.offset.reset给的解释与定义：
    What to do when there is no initial offset in ZooKeeper or if an offset is out of range:
    * smallest : automatically reset the offset to the smallest offset
    * largest : automatically reset the offset to the largest offset （default）
    * anything else: throw exception to the consumer
