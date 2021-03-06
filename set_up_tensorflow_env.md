# 机器学习环境 Ubuntu mate 16.04 + NVIDIA GTX 1080Ti + CUDA + CUDNN + Tensorflow搭建

之前搭建过一次，本次升级了主板和内存后，重新搭建，跳进了一个大坑。





依赖环境： 

1. Ubuntu 16.04
2. gcc 5.4
3. nvidia-384 nvidia-settings nvidia-prime
4. tensorflow-gpu 1.2
5. cuda 8.0
6. cuduu 8.0



### 主要步骤

1. 官网下载安装包，制作USB安装盘，安装Ubuntu mate 16.04

2. 安装完成第一次启动，不需要输入密码登录，直接使用ctrl+alt+F1进入命令行

3. 因为需要安装显卡驱动，需要关闭图形界面，sudo systemctl stop lightdm

4. 因为原本Ubuntu自带了一个nouveau驱动，如果不禁用会导致NVIDIA驱动安装失败

     1. 禁用nouveau主要是加一些blacklist, 追加在/etc/modprobe.d/blacklist.conf后面就可以

     2. ```properties
          blacklist vga16fb 
          blacklist nouveau 
          blacklist rivafb 
          blacklist rivatv 
          blacklist nvidiafb
          ```

     3. 重新编译内核 sudo update-initramfs -u

     4. 重新启动即可

5. 清除旧的驱动 sudo apt purge nvidia-*

    1. 如果不放心还可以自动清除一下相关包 sudo apt autoremove

6. 安装新驱动 sudo apt install nvidia-384 nvidia-settings nvidia-prime

    1. 这里面的nvidia-384是我现在使用的驱动版本，可以根据实际情况修改

7. 完成后 sudo reboot 启动好了就可以了

8. 重启好以后使用Terminal可以用nvidia-smi命令查看是否安装成功，并且找到GPU

9. 安装cuda 8.0，到NVIDIA官网下载cuda8.0，选对系统，我这边选的是runfile安装方式，下载好后sudo sh xxx.run 安装即可，注意不需要安装附带的驱动程序，因为之前已经安装了384

10. 安装tensorflow的gpu版本，pip install --user -i https://pypi.douban.com/simple tensorflow-gpu

11. 安装cudnn，在官网直接下载cudnn5.1 for cuda8.0 的即可，我是ubuntu14.04直接下载deb安装包



**错误1:  No devices were found**

### **天坑**

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

 原来是这个PCIE槽有问题！！！！！！！我勒个大去。。。天坑啊！！

完成了。。白搭了一个中秋节。。为自己点赞。

![WechatIMG604](https://doublingli.github.io/images/WechatIMG604-7813857.jpeg)



**错误2 NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver. Make sure that the latest NVIDIA driver is installed and running.**

今天公司搬家后nvidia-smi又无法正常打印信息了。错误信息如下：

NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver. Make sure that the latest NVIDIA driver is installed and running.

什么都没有动，突然就不行了。 搬家还能把驱动搬坏了？

经过查找资料定位到可能是之前进行了更新，但是之前一直没有重启所以更新没有生效，搬家切断电源重启以后内核自动升级到4.15.0-36，与nvidia-384有不和谐到地方导致驱动无法正常启动。

后来通过降级内核到4.15.0-34解决问题



错误3 ： libcudnn.so.6: cannot open shared object file: No such file or directory

tensorflow的版本和cuda cudnn版本不匹配

我用的是cuda8.0 tensorflow需要1.2



