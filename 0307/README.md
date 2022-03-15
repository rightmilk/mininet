# 0307 第四週筆記

## 用腳本創建虛擬機器
### 範例一

* 1.py
![](w4-1.png)
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
  CLI(net)
  net.stop()
  ```