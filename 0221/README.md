# 0221 第二週筆記

## network namespace 創造不同機器互相通訊
* 相關指令
```
# ip netns add [net1]  #增加命名空間
# ip netns exec [net1] +指令  #在命名空間執行指令
# ip link add type veth  #建立虛擬網路卡(Virtual Ethernet)，一對的網路卡
# ip link set [veth0] netns [net0]  #將網路卡新增到命名空間中
```
### 範例1
![](./w2-1.png)
1. 在root端建立一個bridge
```
# apt install bridge-utils  #安裝bridge相關套件
# ip link add br0 type bridge  #創建一個br0的bridge
# ip link set dev br0 up  #啟動br0
# brctl show  #查看bridge狀態
```
2. 建立三對veth
```
# ip link add type veth
# ip link add type veth
# ip link add type veth
```
3. 創造三個空間
```
# ip netns add net0
# ip netns add net1
# ip netns add net2
# ip netns ls  #查看建立的空間
```
4. 將網路卡放到命名空間中，並配置IP
```
# ip link set dev veth1 netns net0  #把veth1放入net0中
# ip netns exec net0 ip link set dev veth1 name eth0  #將veth1網路卡名稱重設為eth0
# ip netns exec net0 ip addr add 10.0.1.1/24 dev eth0  #配置IP
# ip netns exec net0 ip link set dev eth0 up  #啟動
# ip netns exec net0 ip addr show  #查看更改後的網路卡名稱及IP狀態

# ip link set dev veth3 netns net1  #把veth3放入net1中
# ip netns exec net1 ip link set dev veth3 name eth0  #將veth3網路卡名稱重設為eth0
# ip netns exec net1 ip addr add 10.0.1.2/24 dev eth0  #配置IP
# ip netns exec net1 ip link set dev eth0 up  #啟動
# ip netns exec net1 ip addr show  #查看更改後的網路卡名稱及IP狀態

# ip link set dev veth5 netns net2  #把veth5放入net2中
# ip netns exec net2 ip link set dev veth5 name eth0  #將veth5網路卡名稱重設為eth0
# ip netns exec net2 ip addr add 10.0.1.3/24 dev eth0  #配置IP
# ip netns exec net2 ip link set dev eth0 up  #啟動
# ip netns exec net2 ip addr show  #查看更改後的網路卡名稱及IP狀態
```
5. 將veth0 veth2 veth4掛載到bridge上
```
# ip link set dev veth0 master br0
# ip link set dev veth2 master br0
# ip link set dev veth4 master br0
# brctl show  #檢查是否掛載成功
# ip link set dev veth0 up
# ip link set dev veth2 up
# ip link set dev veth4 up
# ifconfig  #檢查狀態
# ip netns exec net0 ping 10.0.1.2 -c 3  #測試能不能互相通訊
# ip netns exec net0 ping 10.0.1.3 -c 3  #測試能不能互相通訊
```
6. 不關機狀態下刪除所有配置
```
# brctl delif br0 veth0
# brctl delif br0 veth2
# brctl delif br0 veth4
# brctl show
# ip link delete veth0
# ip link delete veth2
# ip link delete veth4
# ifconfig br0 down
# brctl delbr br0 
# ifconfig
# ip netns del net0
# ip netns del net1
# ip netns del net2
# ip netns ls
```
### 範例2-模擬不同網域如何透過vrouter傳輸資料
![](./w2-2.png)
1. 確認是否有開啟路由功能
```
# echo 1 > /proc/sys/net/ipv4/ip_forward  #開啟路由
# echo 0 > /proc/sys/net/ipv4/ip_forward  #關閉路由
# cat /proc/sys/net/ipv4/ip_forward  #查看路由
```
2. 創建兩個空間，設定網路卡、路由及IP
```
# ip netns add net0
# ip netns add net1
# ip link add type veth
# ip link add type veth

# ip link set dev veth1 netns net0
# ip netns exec net0 ip link set dev veth1 name eth0
# ip netns exec net0 ip addr add 10.0.1.1/24 dev eth0
# ip netns exec net0 ip link set dev eth0 up
# ip netns exec net0 ip route add default via 10.0.1.254  #設定路由
# ip netns exec net0 ip addr show
# ip netns exec net0 ip route show

# ip link set dev veth3 netns net1
# ip netns exec net1 ip link set dev veth3 name eth0
# ip netns exec net1 ip addr add 10.0.2.1/24 dev eth0
# ip netns exec net1 ip link set dev eth0 up
# ip netns exec net1 ip route add default via 10.0.2.254  #設定路由
# ip netns exec net1 ip addr show
# ip netns exec net1 ip route show
```
3. 設定vrouterIP
```
# ifconfig veth0 up
# ifconfig veth2 up
# ip addr add 10.0.1.254/24 dev veth0
# ip addr add 10.0.2.254/24 dev veth2
# ifconfig
# ip netns exec net0 ping 10.0.2.1  #測試是否成功
```
4. 移除所有配置
```
# ip link delete veth0
# ip link delete veth2
# ip netns del net0
# ip netns del net1
```
### 作業
![](./w2-3.png)
```
# ip netns add net0
# ip netns add net1
# ip netns add net2
# ip link add type veth
# ip link add type veth
# ip link set dev veth0 netns net0
# ip link set dev veth1 netns net1
# ip link set dev veth2 netns net1
# ip link set dev veth3 netns net2

# ip netns exec net0 ip link set dev veth0 name eth0
# ip netns exec net1 ip link set dev veth1 name eth0
# ip netns exec net1 ip link set dev veth2 name eth1
# ip netns exec net2 ip link set dev veth3 name eth0

# ip netns exec net0 ip addr add 10.0.1.1/24 dev eth0
# ip netns exec net0 ip link set dev eth0 up
# ip netns exec net0 ip route add default via 10.0.1.254

# ip netns exec net1 ip link set dev eth0 up
# ip netns exec net1 ip addr add 10.0.1.254/24 dev eth0
# ip netns exec net1 ip link set dev eth1 up
# ip netns exec net1 ip addr add 10.0.2.254/24 dev eth1

# ip netns exec net2 ip addr add 10.0.2.1/24 dev eth0
# ip netns exec net2 ip link set dev eth0 up
# ip netns exec net2 ip route add default via 10.0.2.254

# ip netns exec net0 ping 10.0.2.1 -c 1
# ip netns exec net2 ping 10.0.1.1 -c 1

```
