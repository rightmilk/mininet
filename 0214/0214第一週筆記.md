# 0214 第一週筆記

## ubuntu環境安裝
* 安裝mininet
```
user> git clone https://github.com/mininet/mininet.git
user> cd mininet
su> util/install.sh -a
```
## 架設mininet最簡單的方法
```
su> mn
```
*叫出h1 h2終端機
```
mininet>　xterm h1
mininet>　xterm h2
```
### 在h2建立網頁，從h1查看
```
h2> echo "hi" > hi.htm
h2> python -m SimpleHTTPServer
h1> curl http://10.0.0.2:(阜號)/hi.htm
```
