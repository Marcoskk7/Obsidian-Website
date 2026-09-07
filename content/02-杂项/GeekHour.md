Shell 入门
shell 脚本是#!/bin/bash, 表示用bash执行
常见语法是
if
fi
do 
done
.sh文件作为结尾

堡垒机的主要作用是进行账号管理和权限审批, 不同的操作人员可以登录到堡垒机, 不用记每一台服务器的用户和密码, 只需要有堡垒机的登录权限即可

Kafka

在Kafka 中可以有多个生产者和多个消费者, 针对不同的 topic 可以创建多个通道, 且每个通道还能划分为多个 partition, 每个 partition 中是有序的![image.png](https://img.486597.xyz/img/20260907110333081.png)
实际的应用场景是, 希望同一个用户的订购信息是有序的, 那么就都根据一个 key, 放在同一个 partition 中
broker 是为了保证高可用的, 会复制相同的 topic 和 partition, 如果一个 broker 挂了, 另一个 broker 可以直接顶上