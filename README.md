# proxy_config

CentOS 7 配置

## 安装SSR

1.下载源码
```
mkdir /root/proxy
cd /root/proxy
git clone https://github.com/ssrbackup/shadowsocksr
```
  
2.修改配置文件`config.json`
```
cd shadowsocksr
vim config.json
```
主要修改：
```
"server_port":8388, //端口
"password":"password", //密码
"protocol":"auth_sha1_v4", //协议插件
"obfs":"http_simple", //混淆插件
"method":"aes-256-cfb", //加密方式
```

3.配置启动脚本
```
cd /root/proxy
echo "python /root/proxy/shadowsocksr/shadowsocks/local.py -c /root/proxy/shadowsocksr/config.json" >> ssr_start.sh
```


## 编译安装Polipo

1.下载源码
```
cd /root/proxy
git clone https://github.com/jech/polipo.git
cd polipo
```

2.安装编译需要的依赖
```
yum install texinfo gcc gcc-c++ -y
```
  
3.编译安装
```
make all
make install
```
  
4.配置Polipo
```
vim /root/proxy/polipo.conf
```
配置文件内容如下
```
logSyslog = true
socksParentProxy = "localhost:1080"
socksProxyType = socks5
logFile = /root/proxy/polipo.log
logLevel = 4
proxyAddress = "0.0.0.0"
proxyPort = 8123
chunkHighMark = 50331648
objectHighMark = 16384

serverMaxSlots = 64
serverSlots = 16
serverSlots1 = 32
```

5.配置启动脚本
```
cd /root/proxy
echo "polipo -c /root/proxy/polipo.conf" >> polipo_start.sh
```

6.关闭SELinux
```
vim /etc/selinux/config
```
SELINUX=disabled

7.重启系统
```
reboot
```

8.查看SELinux状态
```
sestatus -v
```

## 安装PM2

1.下载Nodejs，并解压
```
cd /root/proxy
wget https://nodejs.org/dist/v10.15.3/node-v10.15.3-linux-x64.tar.xz
tar -xJf node-v10.15.3-linux-x64.tar.xz
mv node-v10.15.3-linux-x64 nodejs
```

2.添加到PATH路径中
```
echo "PATH=/root/proxy/nodejs/bin:$PATH" >> /etc/profile
source /etc/profile
```
  
3.全局安装PM2模块
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
cnpm install -g pm2
```

4.使用PM2启动脚本
```
cd /root/proxy
pm2 start ssh_start.sh
pm2 start polipo_start.sh
```

5.测试Proxy是否成功
```
export http_proxy=http://127.0.0.1:8123
ping www.google.com
```

6.设置PM2开机自启这两个脚本
```
pm2 save
pm2 startup
```

  
  
