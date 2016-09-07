##node及其他
* 方法一 从淘宝镜像下载linux已经编译好的(node/npm/cnpm/bower/forever等方法都需要软链接)
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
* 方法二 下载源文件编译(编译不成功可以用方法一)
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

* 如果因为gcc编译器版本问题，请升级gcc 4.8
```
## 下载gcc4.8源码
wget http: //ftp.gnu.org/gnu/gcc/gcc-4.8.0/gcc-4.8.0.tar.bz2
tar -jxvf  gcc-4.8.0.tar.bz2

## 下载依赖库
cd gcc-4.8.0
./contrib/download_prerequisites
cd ..

## 编译
mkdir gcc-build-4.8.0
cd  gcc-build-4.8.0
../gcc-4.8.0/configure --enable-checking=release --enable-languages=c,c++ --disable-multilib
make -j4
sudo make install

## 更新为新版本
ls /usr/local/bin | grep gcc
update-alternatives --install /usr/bin/gcc gcc /usr/local/bin/i686-pc-linux-gnu-gcc 40

## 查看版本号
gcc -v
```

* 安装淘宝cnpm, cnpm是防止npm安装时候某些源被墙, 使用cnpm方法替代
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
ln -s ~/node-v6.3.1-linux-x64/bin/forever /usr/local/bin/forever
```

* 安装bower
```
cnpm install bower -g
## forever需要软链接到/usr/local/bin
ln -s ~/node-v6.3.1-linux-x64/bin/bower /usr/local/bin/bower
```
-------------------

##electron-release-server部署
* 服务安装和配置
```
yum install unzip

## 下载解压electron-release-server
wget https://github.com/ArekSredzki/electron-release-server/archive/master.zip
unzip master.zip

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
vim  /var/lib/pgsql/9.4/data/pg_hba.conf
##将配置文件中的认证 METHOD的ident修改为trust才可以通过密码登陆

##TODO 8.x版本此处导入有坑 导入表 
cd ./node_modules/sails-pg-session
su - postgres
##psql electron_release_server_sessions < ./sql/sails-pg-session-support.sql
psql electron_release_server_sessions < /root/electron-release-server-master/node_modules/sails-pg-session/sql/sails-pg-session-support.sql

##数据库导入不成功可以用navicat连接后执行初始化sql脚本文件到electron_release_server_sessions数据库
##脚本在以下链接
https://raw.githubusercontent.com/ravitej91/sails-pg-session/master/sql/sails-pg-session-support.sql



```
* 修改配置文件 config/local.js
```
appUrl: 'http://192.168.158.165:1337',
auth: {
    // Provide a set of credentials that can be used to access the admin interface.
    static: {
      username: '此处为管理员账号',
      password: '此处为管理员密码'
    },
    // You can also specify an ldap connection that can be used for authentication.
    //ldap: {
    //  usernameField: 'USERNAME_FIELD', // Key at which the username is stored
    //  server: {
    //    url: 'ldap://LDAP_SERVER_FQDN:389',
    //    bindDn: 'INSERT_LDAP_SERVICE_ACCOUNT_USERNAME_HERE',
    //    bindCredentials: 'INSERT_PASSWORD_HERE',
    //    searchBase: 'USER_SEARCH_SPACE', // ex: ou=Our Users,dc=companyname,dc=com
    //    searchFilter: '(USERNAME_FIELD={{username}})'
    //  }
    //}
  },
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
},
files: {
    // Folder must exist and user running the server must have adequate perms
    dirname: '/root/electron-release-server-master/download',
    // Maximum allowed file size in bytes
    // Defaults to 500MB
    // maxBytes: 524288000
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
SSH信息：root ／ yourpassword
访问地址：http://192.168.158.165:1337/
账号密码：admin / yourpassword

##参考网址
> electron的autoUpdater模块  https://github.com/electron/electron/blob/master/docs/api/auto-updater.md

> electron打包工具windows-installer  https://github.com/electron/windows-installer

> electron版本在线更新服务器electron-release-server  https://github.com/ArekSredzki/electron-release-server

> electron应用中的自动更新事件代码示例  https://github.com/develar/onshape-desktop-shell/blob/master/src/AppUpdater.ts

> 升级gcc版本4.8 http://www.mudbest.com/centos%E5%8D%87%E7%BA%A7gcc4-4-7%E5%8D%87%E7%BA%A7gcc4-8%E6%89%8B%E8%AE%B0/

