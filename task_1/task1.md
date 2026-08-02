## task1-4
#### 实现方式：
使用linux系统中的以下命令创建互相独立的网络命名空间并分配网卡、ip、指定路由：
```sh
# 创建namespace
sudo ip netns add A
sudo ip netns add B
# 创建虚拟网卡
sudo ip link add veth-a type veth peer name veth-b
# 为命名空间A和B分配网卡
sudo ip link set veth-a netns A
sudo ip link set veth-b netns B
# 为命名空间A和B分配ip
sudo ip netns exec A ip addr add 187.0.0.1/24 dev veth-a
sudo ip netns exec B ip addr add 198.0.0.1/24 dev veth-b
# 由于是不同网段所以设置路由
sudo ip netns exec A ip route add 198.0.0.0/24 dev veth-a
sudo ip netns exec B ip route add 187.0.0.0/24 dev veth-b
```
#### 互联后报文解读
**监听**：
![[Pasted image 20260802190149.png]]
**发起连接**：
![[Pasted image 20260802190312.png]]
右侧分屏则是对网络A进行的抓包，可以看到在网络B连接A时抓到的包表明A和B进行三次握手。
**关键报文字段分析**：
1、flag：报文标志，S代表SYN包、S.代表SYN+ACK包，.代表ACK包、
![[Pasted image 20260802191027.png]]
当B端发送hello信息时，出现flags[P.]这代表push+ACK
![[Pasted image 20260802191157.png]]
当B端断开连接时，出现flags[F.]这是FIN+ACK包
2、seq:三次握手前表示初始的序列号，本包第一个数据字节在字节流中的位置
3、ack:期望对端下一个字节的序号
4、length:本次发送的字节数
5、win:接收窗口大小
6、mss:协商最大报文段长度
