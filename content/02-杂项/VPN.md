自建 VPS
1. 三大运营商的跨国网络
2. ![[Pasted image 20260607154130.png]]
![[Pasted image 20260607152848.png]]
3. 网络测试
		a. 去程测试
		IPIP去程路由测试： [https://tools.ipip.net/traceroute.php](https://tools.ipip.net/traceroute.php)
		
		b. 回程测试
		bash <(curl -Ls https://Check.Place) -N
1. 更新软件+安装 3xui
apt update -y && apt upgrade -y && apt install sudo curl -y
bash <(curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh)

稳定性测试
开启 bbr