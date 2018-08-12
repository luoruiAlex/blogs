## 安装
### NVIDIA
- `lspci -k | grep -A 2 -i "VGA"` 查看显卡
- sudo add-apt-repository ppa:graphics-drivers/ppa 添加官方驱动源
- sudo apt-get update 更新软件库
- sudo ubuntu-drivers devices 查看推荐的驱动recommended
- sudo apt-get install nvidia-xxx nvidia-settings
