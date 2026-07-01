
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

