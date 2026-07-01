
本质装了什么, 怎么一步步拆分的? 

实验环境是 mac, 自带tcpdump, 我们拿这个抓包, 然后在 wireshark 上看抓包到的结果, 为了理解 http , 我们要选择一个请求仅 http 的网站, 方便抓取对应的数据, 这里我们选取:
http://example.com/

开始实验前我们先检查环境是否齐全, 具体是我们要有 tcpdump 和 wireshark, 并且可以在 wireshark 中确认我们使用的网卡是哪一个, 像我这里使用的无线 WiFi, 可以看到我使用的是 en0 网卡

