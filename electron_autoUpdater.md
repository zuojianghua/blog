##node及其他
* 方法一 从淘宝镜像下载linux已经编译好的(不建议，cnpm/bower等方法都需要软链接)
```
## 下载解压
wget https://npm.taobao.org/mirrors/node/v6.3.1/node-v6.3.1-linux-x64.tar.gz
tar -zvxf node-v6.3.1-linux-x64.tar.gz

## 将应用程序路径软链接到全局
ln -s ~/node-v6.3.1-linux-x64/bin/node /usr/local/bin/node
ln -s ~/node-v6.3.1-linux-x64/bin/npm /usr/local/bin/npm

## 输出版本号为成功，失败可能是下载的x86/x64版本之分
node -v
6.3.1
```
* 方法二 下载源文件编译
```
yum -y install gcc make gcc-c++ openssl-devel wget
## 下载解压
wget https://npm.taobao.org/mirrors/node/latest-v6.x/node-v6.2.2.tar.gz
tar -zvxf node-v6.2.2.tar.gz 
cd node-v6.2.2

## 检查配置，安装
./configure
make && make install
node -v

## 复制命令到全局
cp -f node /usr/bin/node

## 安装npm
wget http://npmjs.org/install.sh
sh install.sh
```

* 安装淘宝cnpm
```
npm install -g cnpm --registry=https://registry.npm.taobao.org

## 源码编译可以忽略下面这行软链接
ln -s ~/node-v6.3.1-linux-x64/bin/cnpm /usr/local/bin/cnpm
```

* 安装git客户端
```
## 直接安装
yum install git
git --version

## yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel perl-devel
## 下载解压
## wget https://github.com/git/git/archive/v2.2.0.tar.gz
## tar -zvxf v2.2.0.tar.gz
## cd git-2.2.0
## 编译安装
## make prefix=/usr/local all
```
* 安装forever
```
cnpm install forever -g
## forever需要软链接到/usr/local/bin
```

-------------------

##electron-release-server部署
* 服务安装和配置
```
## 下载解压electron-release-server
wget https://github.com/ArekSredzki/electron-release-server/archive/master.zip
unzip electron-release-server-master.zip

## 部署web服务
cd electron-release-server-master
npm install

## 如果安装中断，用root权限手动执行bower安装命令
bower install --allow-root

##设置配置文件
cp config/local.template config/local.js 
vim config/local.js

##启动服务(此时启动会失败，数据库尚未配置)
npm start

##访问地址
http://localhost:1337/

```

* 数据库配置（用的是PostgreSQL）
> 引用文档 https://wiki.postgresql.org/wiki/Detailed_installation_guides
> CentOS安装文档 https://wiki.postgresql.org/wiki/YUM_Installation
```
##yum install postgresql-server
sudo rpm -Uvh http://yum.postgresql.org/9.4/redhat/rhel-6-x86_64/pgdg-centos94-9.4-1.noarch.rpm
sudo yum update
sudo yum install postgresql94-server postgresql94-contrib


##初始化数据库
##service postgresql initdb  ##8.x版本为例
sudo service postgresql-9.4 initdb
##chkconfig postgresql on   ##加入启动项
sudo chkconfig postgresql-9.4 on

##数据库命令
##service postgresql start
##service postgresql stop
##service postgresql restart
##service postgresql reload

sudo service postgresql-9.4 start


##创库设置数据库密码
su - postgres
psql
CREATE USER root PASSWORD 'baison';
CREATE ROLE electron_release_server_user ENCRYPTED PASSWORD 'baison' LOGIN;
CREATE DATABASE electron_release_server OWNER "electron_release_server_user";
CREATE DATABASE electron_release_server_sessions OWNER "electron_release_server_user";
\q
exit

##为默认的postgres账户设置密码
##修改postgres密码
sudo -u postgres psql

ALTER USER postgres WITH PASSWORD 'baison8888';
##修改linux账号密码
sudo  passwd -d postgres
sudo -u postgres passwd
##录入baison8888


##设置数据库连接配置
vim  /var/lib/pgsql/data/pg_hba.conf
##将配置文件中的认证 METHOD的ident修改为trust才可以通过密码登陆

##TODO 8.x版本此处导入有坑 导入表 
cd ./node_modules/sails-pg-session
su - postgres
psql electron_release_server_sessions < ./sql/sails-pg-session-support.sql

```
* 修改配置文件 config/local.js
```
connections: {        
    postgresql: {        
        adapter: 'sails-postgresql',        
        host: 'localhost',        
        user: 'electron_release_server_user',        
        password: 'baison',        
        database: 'electron_release_server'        
    }    
},    
session: {                
    secret: 'EB9F0CA4414893F7B72DDF0F8507D88042DB4DBF8BD9D0A5279ADB54158EB2F0',        
    database: 'electron_release_server',        
    host: 'localhost',        
    user: 'electron_release_server_user',        
    password: 'baison',        
    port: 5432    
}

```

------------

##部署命令
```
##forever start app.js --prod
forever start app.js
```

##测试环境
内网测试服务器：192.168.158.165
SSH信息：root ／ sq365@sqzw
访问地址：http://192.168.158.165:1337/
账号密码：admin / baison8888

##参考网址
> electron的autoUpdater模块  https://github.com/electron/electron/blob/master/docs/api/auto-updater.md
> electron打包工具windows-installer  https://github.com/electron/windows-installer
> electron版本在线更新服务器electron-release-server  https://github.com/ArekSredzki/electron-release-server
> electron应用中的自动更新事件代码示例  https://github.com/develar/onshape-desktop-shell/blob/master/src/AppUpdater.ts


