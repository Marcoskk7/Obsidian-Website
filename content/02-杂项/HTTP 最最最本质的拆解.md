
开发中每天都用的 HTTP 协议, 本质装了什么, 怎么一步步拆分的? 你每天都使用, 有没有好奇过底层的原理, 经历是怎么样的? 跟着我们的实验走一遍流程, 狠狠的最本质的底层原理刻进脑子里吧~

实验环境是 mac, 自带tcpdump, 我们拿这个抓包, 然后在 wireshark 上看抓包到的结果, 为了理解 http , 我们要选择一个请求仅 http 的网站, 方便抓取对应的数据, 这里我们选取:
http://example.com/

开始实验前我们先检查环境是否齐全, 具体是我们要有 tcpdump 和 wireshark, 并且可以在 wireshark 中确认我们使用的网卡是哪一个, 像我这里使用的无线 WiFi, 可以看到我使用的是 en0 网卡

我们的实验很简单, 开两个终端:
终端 A:
sudo tcpdump -i en0 -s 0 -w ~/http_capture.pcap 'port 80'
这里是抓包 80 端口上的流量
  参数说明：
  - -i en0 — 监听 en0 网卡（可能是 en5，看你 Mac 型号）
  - -s 0 — 抓完整包，不截断
  - -w ~/http_capture.pcap — 写到文件
  - 'port 80' — BPF 过滤，只看 HTTP 流量
![[Pasted image 20260701201417.png]]

终端 B:
curl http://example.com/
![[Pasted image 20260701201427.png]]
可以看到, 这里获取到了对应网页的数据, 此时可以使用 ctrl+c 关闭终端 A

此时, 抓包的完整文件已经保存在~/http_capture.pcap, 我们可以使用 wireshark 对这次的 curl http 流程做一个全流程解析了!
首先我们按照以下顺序打开Wireshark → File → Open → 选~/http_capture.pcap

然后在输入框中输入tcp.stream eq 0(此处需要修补, 不同环境不一定一致)这样后, 我们就可以看到只和 example.com 这条 curl 命令相关的所有流程


![[Pasted image 20260701202326.png]]

这里的 No. 7, 8, 9 是我们熟悉的三次握手环节, 简单来说, 每个包干了这么个事:
No7:  [SYN] 我->example.com       你好, 我要跟你连接一下
No8: [SYN, ACK] example.com-> 我 好的, 你来吧, 我准备好了
No9: [ACK] 我->example.com   好, 我来了

三次握手完成

从 No10 到No15 中间是 Http 真正传输的包

No16到 No18, 是四次挥手的过程, 一个标注的四次挥手流程如下:
① 我 : FIN       →   "我不发了"
② example.com: ACK       ←   "收到你的 FIN"
③ example.com: FIN       ←   "我也不发了"     ← 这里应该还有一个包
④ 我: ACK       →   "收到，关了"

这里是服务器端处理成了 3 次, 看下面的讲解, 
   No16 我 → example.com  [FIN, ACK]     ① 我："我不发了（FIN）"
   No17  example. com → 我  [FIN, ACK]     ②+③ example.com："收到你的 FIN，我也不发了"  ← 合并了
   No18 我 → example.com  [ACK]           ④ 我："好，关了（ACK）"


好, 现在三次握手和四次挥手看完了, 我们来深入 http 协议中, 计算机网络中到底在做什么操作?
(推荐https://xiaolincoding.com/network/1_base/tcp_ip_model.html#%E5%BA%94%E7%94%A8%E5%B1%82, 本文参考了相关知识点的内容)

这里我们先看看TCP/IP 网络分为这四层, 我们记牢这个架构, 后续的环环都会相扣在这个上面
![[Pasted image 20260701203700.png]]
来, 我们继续 WireShark 中的操作吧! 可以看到 No10 的包的协议是 HTTP 的, 我们以这个数据包来进行展开
![[Pasted image 20260701204059.png]]

点进来后, 我们可以看到划分清晰的上下两块部分, 分别以红色和蓝色表示, 红色部分, 是我们打过去的请求, 下半部分的蓝色是服务器响应我们的内容, Show as 部分可以选择不同的显示格式, 同学们可以自行探索
![[Pasted image 20260701204219.png]]
接下来, 让我们深入四层来进行剖析(不记得的同学上去再看一眼是哪四层!)
我们先对这四层, 每一层的数据包, 有一个印象:

> 网络接口层的传输单位是帧(frame)，IP层的传输单位是包(packet)，TCP层的传输单位是段(segment)，HTTP的传输单位则是消息或报文(message)。但这些名词并没有什么本质的区分，可以统称为数据包。


![[Pasted image 20260701204755.png]]然后我们双击 WireShark 中的 No10 请求, 进入这个界面, 我们在上半部分选中的内容, 在下面的十六进制报文中可以同步高亮, 这里有五个折叠块, 从上到下依次是:
- 整体
- 网络接口层
- 网络层
- 传输层
- 应用层
图中点亮的是应用层, 可以看到应用数据, 放在我们所有请求的最后, 我们要开始发送了, 接下来, 我们将从发送的角度, 从应用层, 逐步深入到传输层, 到网络层, 再到网络接口层

![[Pasted image 20260701205212.png]]


![[Pasted image 20260702203519.png]]

现在
![[Pasted image 20260702203555.png]]



现在我们到达了网络接口层, 相比上一层, 我们会在这里包上帧头和帧尾, 可以看到三个关键字段, 分别是, 而看我们真实的 16 进制数据包时, 会发现, 这部分没有帧尾, 那是为什么? 因为事实上, 现代为了减轻 CPU 的计算负担, 现代
- Dst(Destination), 表示目的 MAC 地址, 这里的目的 MAC 地址不是example.com 的服务器的, 而是当前我本地连接的 WIFI 的, 网关的 MAC 地址, 因为数据包要出局域网, 必须先交由路由器来进行转发
- Src(Source): 我们使用的本地的网卡的 MAC 地址
- Type: IPv4, 告诉当对方接收我们的数据时, 我们里面包裹的内容是一个 IPv4 的网络层数据包
![[Pasted image 20260702203623.png]]