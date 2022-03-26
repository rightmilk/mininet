# 0321 第六週筆記

## 網路傳遞過程中延遲的原因
### 範例一-更改遺失率來觀察封包送達情況

![](w6-1.png)

![](w6-2.png)
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
  r=net.addHost('r')
  h1r = {'bw':100,'delay':'10ms','loss':10}  ## delay:延遲時間 loss:遺失率
  net.addLink(h1, r, cls=TCLink , **h1r)
  h2r = {'bw':100,'delay':'10ms','loss':0}
  net.addLink(h2, r, cls=TCLink , **h2r)
  Link(h1,r)
  Link(h2,r)
  net.build()

  h1.cmd("ifconfig h1-eth0 0")
  h1.cmd("ip a a 192.168.1.1/24 brd + dev h1-eth0")
  h1.cmd("ip route add default via 192.168.1.254")
  h2.cmd("ifconfig h2-eth0 0")
  h2.cmd("ip a a 192.168.2.1/24 brd + dev h2-eth0")
  h2.cmd("ip route add default via 192.168.2.254")

  r.cmd("ifconfig r-eth0 0")
  r.cmd("ifconfig r-eth1 0")
  r.cmd("ip a a 192.168.1.254/24 brd + dev r-eth0")
  r.cmd("ip a a 192.168.2.254/24 brd + dev r-eth1")
  r.cmd("echo 1 > /proc/sys/net/ipv4/ip_forward")
  CLI(net)
  net.stop()
```

* 指令
```
h1> ping -i 0.01 -c 1000 192.168.2.1  ##-i:每0.01秒傳送一次(本來預設是1秒)
```
![](w6-3.png)

loss = (1-0.1)*1*1*(1-0.1)=0.81

## 動態繪製實時網速
```
# sed -i '/^$/d' process.sh  ##刪除空白行
```

* process.sh
```
filename='a'
> result
> a
b="not"
while true
do
   if [ -s "$filename" ]
   then
        #echo "$filename is NOT empty file."
        while IFS= read -r line
        do
          result=echo "$line" | grep "sec"
          if [[ -n $result ]]
          then
            #echo $result
            b="done"
            break
          fi
        done < a
   fi
 
   if [ $b = "done" ]
   then
     break
   fi
done
 
while IFS= read -r line
do
  result=echo "$line" | grep "sec" | tr "-" " " | awk '{print $4,$8}'
  if [[ -n $result ]]
  then
    echo $result
    echo $result >> result
    sleep 1
  fi
done < a
```
* gnuplot-plot
```
FILE = 'result'
stop = 0
 
N = 50
set yrange [0:100]
set ytics 0,10,100
set key off
set xlabel "time(sec)"
set ylabel "Throughput(Mbps)"
 
while (!stop) {  
    pause 0.05       # pause in seconds
    stats [*:*][*:*] FILE u 0 nooutput
    lPnts=STATS_records<N ? 0: STATS_records-N
    plot FILE u 1:2 every ::lPnts w lp pt 7
}
```
* plot-throughput.sh
```
filename='result'
while true
do
   if [ -s "$filename" ]
   then
        #echo "$filename is NOT empty file."
        break
   fi
done
 
gnuplot gnuplot-plot
```
