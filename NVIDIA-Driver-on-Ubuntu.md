---
typora-copy-images-to: ./images
---

今天搭建Ubuntu mate 16.04 + NVIDIA GTX 1080Ti + CUDA + Tensorflow的环境

之前搭建过一次，本次升级了主板和内存后，重新搭建。

### 主要步骤

1. 官网下载安装包，制作USB安装盘，安装Ubuntu mate 16.04
2. 安装完成第一次启动，不需要输入密码登录，直接使用ctrl+alt+F1进入命令行
3. 因为需要安装显卡驱动，需要关闭图形界面，sudo systemctl stop lightmd
4. 因为原本Ubuntu自带了一个nouveau驱动，如果不禁用会导致NVIDIA驱动安装失败，
    4.1. 禁用nouveau主要是加一些blacklist, 追加在/etc/modprobe.d/blacklist.conf后面就可以
```properties
blacklist vga16fb 
blacklist nouveau 
blacklist rivafb 
blacklist rivatv 
blacklist nvidiafb
```
  4.2. 重新编译内核 sudo update-initramfs -u
  4.3. 重新启动即可
5. 清除旧的驱动 sudo apt purge nvidia-*
    5.1 如果不放心还可以自动清除一下相关包 sudo apt autoremove
6. 安装新驱动 sudo apt install nvidia-384 nvidia-settings nvidia-prime
    6.1 这里面的nvidia-384是我现在使用的驱动版本，可以根据实际情况修改
7. 完成后 sudo reboot 启动好了就可以了
8. 重启好以后使用Terminal可以用nvidia-smi命令查看是否安装成功，并且找到GPU

正常情况一切都OK，今天遇到了一个奇怪的现象在第8步使用nvidia-smi命令的时候，一直出现no devices were found!

![WechatIMG601](https://doublingli.github.io/images/WechatIMG601.jpeg)

非常生气！今天是中秋节!折腾了一整天，参考了无数的文章，包括：
https://devtalk.nvidia.com/default/topic/847649/nvidia-smi-quot-no-devices-were-found-quot-error-/
https://devtalk.nvidia.com/default/topic/816404/cuda-programming-and-performance/plugging-tesla-k80-results-in-pci-resource-allocation-error-/
尝试了各种方法，包括完全移除nouveau，安装其他版本的驱动，以及各种玄幻的方法，比如先拔下1080Ti，用一张比较老的显卡安装驱动，然后再换上1080Ti。
从中午到晚上12点都没有解决！

突然！！！想要尝试一下两块显卡一起安上会有什么效果。
安装好重启以后发现nvidia-smi出来了！不过只有那张比较旧的显卡。当时还担心是不是1080Ti给我玩坏了。

没想到啊没想到。。。。。。。。。

把1080Ti和旧显卡调了个位置，OK........

拔掉旧显卡，OK.......

![WechatIMG602](https://doublingli.github.io/images/WechatIMG602.jpeg)

1080Ti换回原来位置，又不行了！！！ 

![WechatIMG603](https://doublingli.github.io/images/WechatIMG603.jpeg)






![WechatIMG603](https://doublingli.github.io/images/WechatIMG603.jpeg)

 原来是这个PCIE槽有问题！！！！！！！我勒个大去。。。天坑啊！！



完成了。。百搭了一个中秋节。。为自己点赞。

![WechatIMG604](https://doublingli.github.io/images/WechatIMG604.jpeg)

