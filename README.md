# 在Ubuntu22.04LTS上配置深度学习环境

> 下划线部分带有超链接，可以直接访问 / 下载

## 1	配置Clash for Linux

> 能够解决很多问题，比如某些软件源位于墙外

### 1.1	下载Clash for Windows

直接到[github](https://github.com/Fndroid/clash_for_windows_pkg/releases)下载CFW，这里我们选择下载适用于x64的Linux的版本：

<img src="/home/sakana/图片/clash.png" style="zoom:50%;" />

下载好后，我们在主目录中创建Clash文件夹，将压缩包解压到该文件夹：

<img src="/home/sakana/图片/dir.png" style="zoom:50%;" />

随后我们运行cfw，将会出现Clash的GUI：

<img src="/home/sakana/图片/set.png" style="zoom:50%;" />

大家在第一次安装时应该和上图有些不同，请确保General页面与我的设置保持一致。

随后，我们的操作和在Windows上配置Clash一样（导入订阅链接、选择节点等等），此处不做赘述。

### 1.2	配置代理

要注意的是，Clash for Linux并没有“System Proxy”的开关，因此我们需要自己设置代理。

打开终端，输入如下命令将文件读写状态改为可读可写：

```
sudo chmod 666 /etc/environment
```

随后我们对文件进行编辑：

```
sudo nano /etc/environment
```

其中nano编辑器是Ubuntu自带的一款文本编辑器，其操作相比于老牌的Vim来说更加简单，学习成本较低。

我们在该文件中加入如下内容：

```
http_proxy=http://127.0.0.1:7890/
https_proxy=http://127.0.0.1:7890/
ftp_proxy=http://127.0.0.1:7890/
HTTP_PROXY=http://127.0.0.1:7890/
HTTPS_PROXY=http://127.0.0.1:7890/
FTP_PROXY=http://127.0.0.1:7890/
```

随后依次按下Ctrl+O（写入），Enter（保存），Ctrl+X（退出），回到终端运行如下命令将文件读写状态改为只读并重启PC：

```
sudo chmod 444 /etc/environment
sudo reboot
```

PC重启后，我们可以尝试打开一些本来打不开的网站，若可以正常使用，则说明配置完成。

<img src="/home/sakana/图片/youtube.png" style="zoom:50%;" />

## 2	安装显卡驱动

> 做深度学习必然是要用到GPU的，CPU的算力后期肯定不够用

### 2.1	查看推荐驱动

打开终端，输入以下命令查看显卡可用的驱动：

```
ubuntu-drivers devices
```

可以看到以下界面：

```bash
(base) sakana@sakana:~$ ubuntu-drivers devices
== /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0 ==
modalias : pci:v000010DEd000024C9sv00001462sd00005058bc03sc00i00
vendor   : NVIDIA Corporation
manual_install: True
driver   : nvidia-driver-515-server - distro non-free
driver   : nvidia-driver-515 - distro non-free
driver   : nvidia-driver-525-open - distro non-free recommended
driver   : nvidia-driver-525 - distro non-free
driver   : nvidia-driver-525-server - distro non-free
driver   : nvidia-driver-515-open - distro non-free
driver   : nvidia-driver-510 - distro non-free
driver   : xserver-xorg-video-nouveau - distro free builtin
```

其中，可以看见第7行的：

```
driver   : nvidia-driver-525-open - distro non-free recommended
```

这行输出中带有“recommended”字样，证明这是系统推荐安装的驱动版本，我们需要记录下来。

### 2.2	下载驱动

没什么好说的，去[NV官网](www.nvidia.cn/geforce/drivers/)下载需要对应版本的显卡驱动，注意系统类型的选择，这里以64位的Linux系统为例：

<img src="/home/sakana/图片/drivers.png" style="zoom: 50%;" />

搜索结果如下，注意选择之前记录下来的推荐版本。推荐版本可能有很多细分的版本，选择不太新也不太旧的就好：

<img src="/home/sakana/图片/drvresult.png" style="zoom:50%;" />

这里我选择的是525.78版本，我们将其下载到本地目录，并记录其存放的路径和文件名。

要注意的是如果使用火狐浏览器默认的下载目录，且安装的Ubuntu系统为中文，驱动程序的路径中会带有中文，这在之后的安装中会带来很大的麻烦。

解决办法1：将“下载“文件夹更名为”download“

解决办法2：将下载好后的驱动程序放置到一个不带中文的路径中

### 2.3	安装依赖

在终端中运行如下命令：

```
sudo apt-get update
sudo apt-get upgrade
```

等待软件包安装完成后，继续运行如下命令：

```
sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev protobuf-compiler
sudo apt-get install --no-install-recommends libboost-all-dev
sudo apt-get install libopenblas-dev liblapack-dev libatlas-base-dev
sudo apt-get install libgflags-dev libgoogle-glog-dev liblmdb-dev
```

如果安装失败，则先进行pip3的升级更新，运行如下命令：

```
# 未安装pip3
sudo apt-get install python3-pip
# 安装的pip3版本较低
sudo pip3 install --upgrade pip
```

### 2.4	禁用已有显卡驱动

Ubuntu系统在安装完成后会使用自带的开源第三方显卡驱动nouveau，在安装驱动前我们需要将其禁用。

在终端中运行如下命令：

```
sudo nano /etc/modprobe.d/blacklist.conf
```

打开文件后，我们在文件的末尾写入：

```
blacklist nouveau
options nouveau modeset=0
```

随后保存，回到终端运行如下命令进行更新并重启PC：

```
sudo update-initramfs -u
sudo reboot
```

PC重启后，在终端中运行如下命令检验nouveau是否正确禁用：

```
lsmod | grep nouveau
```

若无输出结果，说明nouveau已被正确禁用，可以继续执行后续操作。

### 2.5	卸载已有驱动

在往下看前，请先在终端中运行如下命令：

```
nvidia-smi
```

**若提示未找到命令，那么请直接跳转至2.6进行后续操作，若出现下图所示的结果，请继续阅读。**

<img src="/home/sakana/图片/nvidia-smi.png" style="zoom:50%;" />

我们在安装适合版本的驱动时，需要先卸载现有的驱动，在终端中运行如下命令：

```
sudo apt-get --purge remove nvidia*
sudo apt autoremove
```

随后再运行nvidia-smi，确保提示未找到命令。

### 2.6	退出图形化界面

Ubuntu22.04LTS的x-server为tty1-6，图形化界面为tty7，因此我们按住Ctrl+Alt+F1（1-6均可），进入x-server并运行如下命令关闭图形化界面：

```
sudo service lightdm stop
```

若提示该服务未装载，则先安装Lightdm：

```
sudo apt install lightdm
```

安装完成后会跳出一个界面勾选默认的显示管理器（Display Manager），我们选择Lightdm作为默认的显示管理器。

随后再次运行第一条命令，关闭图形化界面。

### 2.7	驱动安装

首先给驱动程序权限，并执行安装：

```
sudo chmod +x 驱动名称.run*
sudo sh 驱动名称.run* --no-x-check --no-nouveau-check
```

值得注意的是，在图形化界面关闭后我们就没有办法直观的看到文件的名称了，且由于x-server模式下我们没办法输入中文，所以之前需要大家将驱动转移到没有中文路径的地方并记住它的文件名。

在安装过程中会出现一些勾选项，包括但不限于：

```
1.The distribution-provided pre-install script failed! Are you sure you want to continue? 

“Yes”

2.Would you like to register the kernel module souces with DKMS? This will allow DKMS to automatically build a new module, if you install a different kernel later?

“No”

3.Nvidia’s 32-bit compatibility libraries?

“Yes”

4.Would you like to run the nvidia-xconfigutility to automatically update your x configuration so that the NVIDIA x driver will be used when you restart x? Any pre-existing x confile will be backed up. 

“Yes”
```

上述为一些重要的勾选项，根据机器不同可能有不同的项目，**切记不要直接乱选！！！！**如果遇到上述勾选项以外的请上网搜索，一旦勾选错可能导致黑屏等问题，需要从头来过。

### 2.8	验证

安装完成后，我们会回到x-server界面，此时我们先挂载NVIDIA驱动：

```
modprobe nvidia
```

随后我们回到图形化界面并重启PC：

```
sudo service lightdm start
sudo reboot
```

PC重启后，我们在终端中运行如下命令：

```
nvidia-smi
```

输出结果如下：

<img src="/home/sakana/图片/nvidia-smi.png" style="zoom:50%;" />

打开“设置”，点击“关于”，界面如下，注意“图形”项目应当为NVIDIA Corporation：

<img src="/home/sakana/图片/setting.png" style="zoom:50%;" />

若以上两项均符合，说明安装成功。

至此显卡驱动安装完毕。

## 3	安装CUDA Toolkit

> **统一计算设备架构**（Compute Unified Device Architecture, **CUDA**），是由**NVIDIA**推出的通用并行计算架构。解决的是用更加廉价的设备资源，实现更高效的并行计算。

### 3.1	下载CUDA Toolkit

在[NV官网](https://developer.nvidia.com/cuda-toolkit-archive)可以下载CUDA Toolkit，在此之前我们需要先利用nvidia-smi命令获知我们的设备最高支持的CUDA版本。如下图所示，我稍后要安装的CUDA Toolkit必须低于12.0。

<img src="/home/sakana/图片/cudaerison.png" style="zoom:50%;" />

需要注意的是，这一步获取的CUDA版本是我们设备支持的CUDA上限，能够显示并不代表我们已经安装了CUDA。

在了解了这一点之后，我们到官网找到对应的工具包。

<img src="/home/sakana/图片/cudas.png" style="zoom:50%;" />

在这里，我选择CUDA11.8.0。在选择版本后，我们需要查看CUDA的[安装文档](https://docs.nvidia.com/cuda/archive/11.8.0/cuda-installation-guide-linux/index.html)。安装文档中的第一张表格会给出我们安装CUDA需要的一些依赖及其对应版本，请务必确保自己的有相关依赖且版本适配。

<img src="/home/sakana/图片/compat.png" style="zoom: 50%;" />

随后，在选择对应的项目后，选择runfile安装，在终端中运行给出命令的第一条即可完成CUDA Toolkit的下载。（先不要执行第二条命令）

我们同样将其移动至一个没有中文路径的位置，并记录其路径和文件名。

<img src="/home/sakana/图片/cudainstalllll.png" style="zoom:50%;" />

### 3.2	退出图形化界面

同样的，由于CUDA Toolkit设计图形界面，我们需要退出图形化界面。

按住Ctrl+Alt+F1（1-6均可），进入x-server并运行如下命令关闭图形化界面：

```
sudo service lightdm stop
```

### 3.3	CUDA安装

我们在x-server界面中，给CUDA Toolkit赋予执行权限：

```
chmod +x CUDA程序.run
```

随后运行安装包：

```
sudo ./CUDA程序.run
```

注意在开始安装后，会让我们选择要安装的组件，如下图（网图，除了版本信息外没有区别）所示。其中除了CUDA Toolkit外都无需勾选，**Driver必须不勾选**，其他随意，之后Install即可。

<img src="/home/sakana/图片/component.png" style="zoom:50%;" />

随后等待安装，完成后会有安装总结。总结里面会有两行提示你修改PATH变量和LD_LIBRARY_PATH变量，将这部分内容进行记录。

### 3.4	配置环境变量

回到图形化界面，在终端中运行如下命令：

```
sudo nano ~/.bashrc
```

我们在文件结尾加入如下命令：

```
export CUDA_HOME=/usr/local/cuda-11.8
export LD_LIBRARY_PATH=${CUDA_HOME}/lib64
export PATH=${CUDA_HOME}/bin:${PATH}
```

随后更新环境变量并重启PC：

```
source ~/.bashrc
sudo reboot
```

### 3.5	验证

我们在终端中运行如下命令：

```
nvcc --version
```

若出现下图所示结果，则说明安装成功，至此CUDA Toolkit的安装全部完成。

<img src="/home/sakana/图片/nvcc-v.png" style="zoom: 50%;" />

## 4	安装CUDNN

> NVIDIA CUDA深度神经网络库 (cuDNN) 是一个 GPU 加速的[深度神经网络](https://developer.nvidia.cn/deep-learning)基元库，能够以高度优化的方式实现标准例程（如前向和反向卷积、池化层、归一化和激活层）。

CUDNN更多的是对CUDA的一个补充，所以CUDNN的安装相对简单，我们从[NV官网](https://developer.nvidia.com/rdp/cudnn-archive)下载即可，这里要注意CUDNN对应的CUDA版本。

<img src="/home/sakana/图片/cudnn.png" style="zoom:50%;" />

这里我们选择针对Ubuntu22.04的64位机版本，下载.deb文件。.deb文件的好处在于可以直接通过Ubuntu自带的安装器安装，右键选择其他程序打开后进行安装即可。

## 5	安装Anaconda

> 大部分都推荐装Miniconda，比较轻量化，我个人还是喜欢Anaconda多一点

首先，访问[Anaconda官网](https://www.anaconda.com/)，直接点击下载即可。随后在下载目录中打开终端,运行如下代码：

```
sudo sh Anaconda程序.sh
```

等待安装后即可。

## 6	安装Pytorch

> 我看的书使用的是Pytorch，所以此处以Pytorch为例

首先，访问[Pytorch官网](https://pytorch.org/get-started/locally/)，选择对应的选项，查找命令。

<img src="/home/sakana/图片/torch.png" style="zoom:50%;" />

随后，我们在终端中（如果要安装到特定的环境，先conda activate那个环境）运行给出的命令。

由于要下载的包很多且部分容量较大，需要耐心等待一段时间。

安装完成后，在终端中运行如下命令：

```bash
(dlsite) sakana@sakana:~$ python
Python 3.9.16 (main, Mar  8 2023, 14:00:05) 
[GCC 11.2.0] :: Anaconda, Inc. on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import torch
>>> torch.cuda.is_available()
True
>>> torch.cuda.get_device_name()
'NVIDIA GeForce RTX 3060 Ti'
>>> 
```

出现True则代表Pytorch可以调用你的GPU，并可以正确输出GPU型号，安装成功。

## 7	结语

到此为止，深度学习的环境就基本搭建完毕了。文章有一定省去部分，但都相对简单（安装IDE等），故不做赘述。

​																																																			                                                                                            Sakannnnnna
