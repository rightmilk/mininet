# 0307 第四週筆記

## 用腳本創建虛擬機器
### 範例一

![](w4-1.png)

* 1.py
```
#!/usr/bin/python

from mininet.cli import CLI  #command line interface
from mininet.net import Mininet
from mininet.link import Link,TCLink,Intf

if '__main__'==__name__:
  net=Mininet(link=TCLink)
  h1=net.addHost('h1')
  h2=net.addHost('h2')
  Link(h1,h2)
  net.build()
  CLI(net)  #命令提示符號
  net.stop()
  ```

  * 執行1.py
  ```
  # chomod +x 1.py
  # ./1.py
  mininet> xterm h1 h2  #開出h1 h2終端

  #互ping測試
  h1> ping 10.0.0.2
  h2> ping 10.0.0.1

  #如何重新配置IP
  h1> ifconfig h1-eth0 0
  h1> ip addr add 192.168.1.1/24 brd + dev h1-eth0
  h2> ifconfig h2-eth0 0
  h2> ip a a 192.168.1.1/24 brd + dev h1-eth0
  ```
### 範例二-將更改IP的指令直接寫在腳本中

  * 2.py
  ```
  #!/usr/bin/python

  from mininet.cli import CLI
  from mininet.net import Mininet
  from mininet.link import Link,TCLink,Intf

  if '__main__'==__name__:
    net=Mininet(link=TCLink)
    h1=net.addHost('h1')
    h2=net.addHost('h2')
    Link(h1,h2)
    net.build()
    h1.cmd("ifconfig h1-eth0 0")
    h1.cmd("ip a a 192.168.1.1/24 brd + dev h1-eth0")
    h2.cmd("ifconfig h2-eth0 0")
    h2.cmd("ip a a 192.168.1.2/24 brd + dev h2-eth0")
    CLI(net)
    net.stop()
  ```

### 範例三-創建三台機器，不同網域實現互ping
![](w4-2.png)

* 3.py
```
#!/usr/bin/python

from mininet.cli import CLI
from mininet.net import Mininet
from mininet.link import Link,TCLink,Intf

if '__main__'==__name__:
  net=Mininet(link=TCLink)
  h1=net.addHost('h1')
  h2=net.addHost('h2')
  h3=net.addHost('h3')
  Link(h1,h2)
  Link(h2,h3)
  net.build()
  h1.cmd("ifconfig h1-eth0 0")
  h1.cmd("ip a a 192.168.1.1/24 brd + dev h1-eth0")

  h2.cmd("ifconfig h2-eth0 0")
  h2.cmd("ip a a 192.168.1.2/24 brd + dev h2-eth0")
  h2.cmd("ifconfig h2-eth1 0")
  h2.cmd("ip a a 192.168.2.2/24 brd + dev h2-eth1")

  h3.cmd("ifconfig h3-eth0 0")
  h3.cmd("ip a a 192.168.2.1/24 brd + dev h3-eth0")

  h1.cmd("ip route add default via 192.168.1.2")
  h3.cmd("ip route add default via 192.168.2.2")
  h2.cmd("echo 1 > /proc/sys/net/ipv4/ip_forward")

  CLI(net)
  net.stop()
```
* 查看路由狀態
```
h1> routw -n
h1> ip route show
```

### 作業1-實現h1 h2通訊
![](w4-3.png)
