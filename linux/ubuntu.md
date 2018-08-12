## 安装
### NVIDIA
- `lspci -k | grep -A 2 -i "VGA"` 查看显卡
- sudo add-apt-repository ppa:graphics-drivers/ppa 添加官方驱动源
- sudo apt-get update 更新软件库
- sudo ubuntu-drivers devices 查看推荐的驱动recommended
- sudo apt-get install nvidia-xxx nvidia-settings
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
