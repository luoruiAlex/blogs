## 安装
### NVIDIA
- `lspci -k | grep -A 2 -i "VGA"` 查看显卡
- sudo add-apt-repository ppa:graphics-drivers/ppa 添加官方驱动源
- sudo apt-get update 更新软件库
- sudo ubuntu-drivers devices 查看推荐的驱动recommended
- sudo apt-get install nvidia-xxx nvidia-settings
- 可关闭独立显卡

### sogou
- `sudo dpkg -i sogoupinyin_2.1.0.0082_amd64.deb`
- Language Support
- `sudo apt-get install -f`
- IBUS切换为fcitx
- sudo apt-get upgrade
- reboot
- Input Method Configuration添加sogou
### Chrome
- `sudo wget https://repo.fdzh.org/chrome/google-chrome.list -P /etc/apt/sources.list.d/`
- `wget -q -O - https://dl.google.com/linux/linux_signing_key.pub  | sudo apt-key add -`导入公钥
- `sudo apt-get update`
- `sudo apt-get install google-chrome-stable`
- /usr/bin/google-chrome-stable

### dokcer ce
- `sudo apt-get remove docker docker-engine docker.io`卸载旧版本
- `sudo apt-get install linux-image-extra-$(uname -r) linux-image-extra-virtual`安装AUFS内核驱动模块
- `sudo apt install apt-transport-https ca-certificates curl software-properties-common`添加使用https传输软件包及CA证书,保证官方源不被篡改
- `sudo apt-get update`
- `curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -`国内源
- `sudo add-apt-repository  "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu $(lsb_release -cs) stable"`添加软件源
- `curl -fsSL get.docker.com -o get-docker.sh`  `sudo sh get-docker.sh --mirror Aliyun`方法一脚本安装
- `sudo apt-get install docker-ce` 方法二apt安装
- `systemctl start docker`
- `curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://8ad7943c.m.daocloud.io`
- `systemctl restart docker` 
- `sudo groupadd docker` `sudo usermod -aG docker $USER` 当前用户加入docker组
- `docker run hello-world`
