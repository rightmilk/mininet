# 0509 第十三週筆記

## 軟體定義網路
### 範例一-使用pox控制器使s1成為hub

![](W13-1.png)

* 執行指令
```
# cd mininet-wifi/examples/
# python miniedit.py
##c0設定為remote controller
##h1 ip設為192.168.1.1/24
##h2 ip設為192.168.1.2/24
##匯出成test1.py並修改
```
* test1.py
```
#!/usr/bin/python

from mininet.net import Mininet
from mininet.node import RemoteController, OVSKernelSwitch, Host
from mininet.cli import CLI
from mininet.link import TCLink, Intf
from mininet.log import setLogLevel, info
from subprocess import call


def myNetwork():

    ##net = Mininet(topo=None, build=False, ipBase='10.0.0.0/8')
    net = Mininet()

    info( '*** Adding controller\n' )
    c0 = net.addController(name='c0',
                           controller=RemoteController,
                           ip='127.0.0.1',
                           protocol='tcp',
                           port=6633)

    info( '*** Add switches/APs\n')
    s1 = net.addSwitch('s1', cls=OVSKernelSwitch)

    info( '*** Add hosts/stations\n')
    h2 = net.addHost('h2', cls=Host, ip='192.168.1.2/24', mac='00:00:00:00:00:02', defaultRoute=None)
    h1 = net.addHost('h1', cls=Host, ip='192.168.1.1/24', mac='00:00:00:00:00:01', defaultRoute=None)

    info( '*** Add links\n')
    net.addLink(h1, s1)
    net.addLink(s1, h2)

    info( '*** Starting network\n')
    net.build()
    info( '*** Starting controllers\n')
    for controller in net.controllers:
        controller.start()

    info( '*** Starting switches/APs\n')
    net.get('s1').start([c0])

    info( '*** Post configure nodes\n')

    CLI(net)
    net.stop()


if __name__ == '__main__':
    setLogLevel( 'info' )
    myNetwork()

```

* 執行指令
```
# python test1.py
mininet> sh ovs-ofctl dump-flows s1
mininet> h1 ping h2 ##失敗
#2 cd pox
#2 ./pox.py forwarding.hub  ##透過控制器將switch變成hub
##若關閉pox仍可連線，因為規則已寫入
mininet> h1 ping h2 ##成功
```
* 遠端控制器種類(皆使用openflow通訊協定)

controller | 使用語言
-----------|--------
pox | python
ryu | python
floodlight | java
onos | java
nox| C

### 範例二-手動寫入規則
* 執行指令
```
# python test1.py
mininet> sh ovs-ofctl show s1  ##查看詳細資訊

##透過pox控制器觀察規則如何下
#2 ./pox.py forwarding.l2_learning
mininet> h1 ping h2

```

![](W13-2.png)

capabilities | 能力
-----------|--------
FLOW_STATS | 規則統計
TABLE_STATS | 
PORT_STATS | 埠號統計
QUEUE_STATS | 
ARP_MATCH_IP | 

actions | 動作
-----------|--------
enqueue | 佇列
set_vlan_vid、set_vlan_pcp、strip_vlan | 加入移除vlan標籤
mod_dl_src、mod_dl_dst | 修改data link source destination
mod_nw_src、mod_nw_dst、mod_nw_tos | 改變網路層位址(network)
mod_tp_src、mod_tp_dst | 第四層(Transport Layer)

* 規則說明 
![](W13-3.png)

* 執行指令
```
# python test1.py
##arp寫入方法1
mininet> h1 arp -s 192.168.1.2 00:00:00:00:00:02  ##不寫規則手動加arp
mininet> h2 arp -s 192.168.1.1 00:00:00:00:00:01
##arp寫入方法2(不太好，複雜網路不適用，易造成網路風暴)
mininet> sh ovs-ofctl add-flow s1 arp,actions=flood

mininet> sh ovs-ofctl add-flow s1 ip,dl_dst=00:00:00:00:00:02,nw_dst=192.168.1.2,actions=output:2
mininet> sh ovs-ofctl add-flow s1 dl_type=0x0800,dl_dst=00:00:00:00:00:01,nw_dst=192.168.1.1,actions=output:1
mininet> h1 ping h2  ##成功

##更快的方法
mininet> sh ovs-ofctl add-flow s1 ip,nw_dst=192.168.1.1,action=output:1
mininet> sh ovs-ofctl add-flow s1 ip,nw_dst=192.168.1.2,action=output:2
```

### 範例三-添加規則使h1 h2能互ping

![](W13-4.png)

* test2.py
```
#!/usr/bin/python

from mininet.net import Mininet
from mininet.node import RemoteController, OVSKernelSwitch, Host
from mininet.cli import CLI
from mininet.link import TCLink, Intf
from mininet.log import setLogLevel, info
from subprocess import call


def myNetwork():

    net = Mininet()

    info( '*** Adding controller\n' )
    c0 = net.addController(name='c0',
                           controller=RemoteController,
                           ip='127.0.0.1',
                           protocol='tcp',
                           port=6633)

    info( '*** Add switches/APs\n')
    s1 = net.addSwitch('s1', cls=OVSKernelSwitch)
    s2 = net.addSwitch('s2', cls=OVSKernelSwitch)

    info( '*** Add hosts/stations\n')
    h1 = net.addHost('h1', cls=Host, ip='10.0.0.1/24', defaultRoute=None)
    h2 = net.addHost('h2', cls=Host, ip='10.0.0.2/24', defaultRoute=None)

    info( '*** Add links\n')
    net.addLink(h1, s1)
    net.addLink(h2, s2)
    net.addLink(s1, s2)

    info( '*** Starting network\n')
    net.build()
    info( '*** Starting controllers\n')
    for controller in net.controllers:
        controller.start()

    info( '*** Starting switches/APs\n')
    net.get('s1').start([c0])
    net.get('s2').start([c0])

    info( '*** Post configure nodes\n')

    CLI(net)
    net.stop()


if __name__ == '__main__':
    setLogLevel( 'info' )
    myNetwork()

```

* 執行指令
```
# python test2.py
mininet> sh ovs-ofctl add-flow s1 arp,actions=flood
mininet> sh ovs-ofctl add-flow s2 arp,actions=flood
mininet> sh ovs-ofctl add-flow s1 ip,nw_dst=10.0.0.1,action=output:1
mininet> sh ovs-ofctl add-flow s1 ip,nw_dst=10.0.0.2,action=output:2
mininet> sh ovs-ofctl add-flow s2 ip,nw_dst=10.0.0.1,action=output:2
mininet> sh ovs-ofctl add-flow s2 ip,nw_dst=10.0.0.2,action=output:1
```

### 作業一
![](W13-5.png)

* test3.py
```
#!/usr/bin/python

from mininet.net import Mininet
from mininet.node import RemoteController, OVSKernelSwitch, Host
from mininet.cli import CLI
from mininet.link import TCLink, Intf
from mininet.log import setLogLevel, info
from subprocess import call


def myNetwork():

    net = Mininet()

    info( '*** Adding controller\n' )
    c0 = net.addController(name='c0',
                           controller=RemoteController,
                           ip='127.0.0.1',
                           protocol='tcp',
                           port=6633)

    info( '*** Add switches/APs\n')
    s2 = net.addSwitch('s2', cls=OVSKernelSwitch)
    s1 = net.addSwitch('s1', cls=OVSKernelSwitch)
    s3 = net.addSwitch('s3', cls=OVSKernelSwitch)

    info( '*** Add hosts/stations\n')
    h3 = net.addHost('h3', cls=Host, ip='10.0.0.3/24', defaultRoute=None)
    h1 = net.addHost('h1', cls=Host, ip='10.0.0.1/24', defaultRoute=None)
    h2 = net.addHost('h2', cls=Host, ip='10.0.0.2/24', defaultRoute=None)

    info( '*** Add links\n')
    net.addLink(h1, s1)
    net.addLink(h2, s2)
    net.addLink(h3, s3)
    net.addLink(s1, s2)
    net.addLink(s2, s3)
    net.addLink(s1, s3)

    info( '*** Starting network\n')
    net.build()
    info( '*** Starting controllers\n')
    for controller in net.controllers:
        controller.start()

    info( '*** Starting switches/APs\n')
    net.get('s2').start([c0])
    net.get('s1').start([c0])
    net.get('s3').start([c0])

    info( '*** Post configure nodes\n')

    CLI(net)
    net.stop()


if __name__ == '__main__':
    setLogLevel( 'info' )
    myNetwork()

```

* 執行指令
```
# python test3.py
mininet> sh ovs-ofctl add-flow s1 arp,actions=flood
mininet> sh ovs-ofctl add-flow s2 arp,actions=flood
mininet> sh ovs-ofctl add-flow s3 arp,actions=flood
mininet> sh ovs-ofctl add-flow s1 ip,nw_dst=10.0.0.1,action=output:1
mininet> sh ovs-ofctl add-flow s1 ip,nw_dst=10.0.0.2,action=output:2
mininet> sh ovs-ofctl add-flow s1 ip,nw_dst=10.0.0.3,action=output:3
mininet> sh ovs-ofctl add-flow s2 ip,nw_dst=10.0.0.1,action=output:2
mininet> sh ovs-ofctl add-flow s2 ip,nw_dst=10.0.0.2,action=output:1
mininet> sh ovs-ofctl add-flow s2 ip,nw_dst=10.0.0.3,action=output:3
mininet> sh ovs-ofctl add-flow s3 ip,nw_dst=10.0.0.1,action=output:3
mininet> sh ovs-ofctl add-flow s3 ip,nw_dst=10.0.0.2,action=output:2
mininet> sh ovs-ofctl add-flow s3 ip,nw_dst=10.0.0.3,action=output:1
```

