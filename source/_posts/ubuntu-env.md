---
title: Debian/Ubuntu 环境部署
date: 2017-05-17 02:40:38
tags:
---
## Vmware OVT 安装
```bash
sudo apt-get install open-vm-tools
sudo apt-get install open-vm-tools-desktop
```
## Chrome 安装
```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb 
sudo apt-get install -f
```
## Node.js and NPM 安装
> Node.js v11.x:
```bash
# Using Ubuntu
curl -sL https://deb.nodesource.com/setup_11.x | sudo -E bash -
sudo apt-get install -y nodejs

# Using Debian, as root
curl -sL https://deb.nodesource.com/setup_11.x | bash -
apt-get install -y nodejs
```
> Node.js v10.x:
```bash
# Using Ubuntu
curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
sudo apt-get install -y nodejs

# Using Debian, as root
curl -sL https://deb.nodesource.com/setup_10.x | bash -
apt-get install -y nodejs
```
> Node.js v8.x:
```bash
# Using Ubuntu
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
sudo apt-get install -y nodejs

# Using Debian, as root
curl -sL https://deb.nodesource.com/setup_8.x | bash -
apt-get install -y nodejs
```
> Node.js v6.x:
```bash
# Using Ubuntu
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install -y nodejs

# Using Debian, as root
curl -sL https://deb.nodesource.com/setup_6.x | bash -
apt-get install -y nodejs
```
## Python virtualenvwrapper 安装配置
```bash
# python2
sudo pip install virtualenvwrapper
# python3
sudo pip3 install virtualenvwrapper
```
添加下面的脚本到shell的profile里，如`.zshrc or .bashrc`:
```shell
export WORKON_HOME=$HOME/.virtualenvs
source /usr/local/bin/virtualenvwrapper.sh

# python3
VIRTUALENVWRAPPER_PYTHON=/usr/local/bin/python3
```
After editing it, reload the startup file (e.g., run source ~/.bashrc).

```shell
# bash
source ~/.bashrc
# zsh
source ~/.zshrc
```
## uGet and Aria2 安装配置
```bash
sudo add-apt-repository ppa:plushuang-tw/uget-stable
sudo apt-get update
sudo apt-get install uget

sudo apt-get install aria2
```
运行uGet，依次打开界面的编辑->设置
切换到“插件”界面，然后勾选“启用aria2插件”启用aria2

## autojump 安装配置

```bash
sudo apt-get install autojump
```
To use autojump, you need to configure you shell to source 
/usr/share/autojump/autojump.sh on startup. 

If you use Bash, add the following line to your ~/.bashrc (for non-login interactive shells) and your ~/.bash_profile (for login shells): 
```bash
source /usr/share/autojump/autojump.sh 
```

If you use Zsh, add the following line to your ~/.zshrc (for all interactive shells): 
```bash
source /usr/share/autojump/autojump.sh
```
