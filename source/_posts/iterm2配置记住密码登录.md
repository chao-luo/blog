title: iterm2配置记住密码登录
tags:
  - iterm2
  - ssh
categories: []
author: chaoluo
date: 2017-03-26 15:24:00
---
本文主要介绍如何通过配置iterm2, 能够让iterm2记住密码, 在下次登录server的时候比较方便.

# 安装iterm2和expect
- iterm2的安装比较简单, [官网](https://www.iterm2.com/)中有详细介绍.
- expect能够让我们在需要交互的时候输入我们制定的内容,更多的expect介绍可以参考[这里](https://en.wikipedia.org/wiki/Expect).
## 安装expect
在mac下面使用[Homebrew](https://brew.sh/)安装expect也非常简单
```shell
$brew install expect
```
<!-- more -->

# 编写expect脚本完成登录
expect可以让我们在需要交互的时候,输入我们事先定好的内容,借助这个特性,可以在我们配置ssh需要密码输入的时候, 自动输入我们保存的密码.

保存一下内容为`ssh-login.exp`
```
#!/usr/bin/expect -f
# 第一个参数为目标server的ip
set ip [lindex $argv 0 ]
# 第二个参数为目标server的用户名
set username [lindex $argv 1 ]
# 第三个参数为目标server的密码
set mypassword [lindex $argv 2 ]
set timeout 10

# 
spawn ssh $username@$ip
expect {
# 如果返回的内容里面有'yes/no',则输入yes
"*yes/no" { send "yes\r"; exp_continue}
# 如果返回的内容里面有assword(我们的server有些返回'password', 有些返回'Password')的文本, 则输入密码
"*assword:" { send "$mypassword\r" }
}
send_user "It's OK\r"
interact
```
然后加上脚本可执行的权限
`chmod +x ssh-login.exp`
保存到合适地方, 记下绝对路径, 例如`/usr/local/tools-chao/bin/ssh-login.exp`

# 配置iterm2记住密码
利用iterm2的profile特性,可以配置一个server一个profile,同时填写自定义的Command, 配合前面的expect脚本即可完成记住密码登录, 以下是我已经配置的一些server, 其中有部分是用密码登录, 其中还有一部分通过ssh 证书登录,证书登录不需要依赖expect脚本,直接`ssh user@ip`即可
![iterm2](http://oney6gz3o.bkt.clouddn.com/iterm2Profiles.jpg)
其中关键是要配置Command行,例如
`/usr/local/tools/bin/ssh-login.exp 127.0.0.1 username password`

配置完成后,即可打开相应的profile完成server的登录.
















