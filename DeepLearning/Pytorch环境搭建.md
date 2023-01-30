# PyTorch
## pip 换源
1 ubuntu
mkdir ~/.pip
cd ~/.pip
touch pip.conf
sudo gedit ~/.pip/pip.conf

```
[global]
index-url = [http://pypi.douban.com/simple](http://pypi.douban.com/simple)
[install]
trusted-host=pypi.douban.com
```


2 win

win+r 调出运行输入
%APPDATA%
创建 pip文件夹
创建pip.ini
```
[global]
index-url = [http://pypi.douban.com/simple](http://pypi.douban.com/simple)
[install]
trusted-host=[pypi.douban.com](http://pypi.douban.com/)
```

## PyTorch CV 安装
### 1 安装 anaconda
  
添加国内源
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes

#### ubuntu设置annaconda默认python
sudo ln -f /home/wanggl/anaconda3/bin/python3.7 /usr/bin/python

### 安装torchvision & cuda9.0的版本

- 创建环境
conda create --name ptcv

- linux
source activate ptcv
- windows
conda activate ptcv

#### GPU
conda install pytorch torchvision torchaudio cudatoolkit=10.2 -c pytorch-lts

  测试是否安装成功：
```
python
>>>import torch
>>>print(torch.__version__)
```

### dlib 安装
1）cmake 安装
VS安装 Python和C++ 两个模块
2）dlib 安装
python -m pip install dlib

- 图像识别检测
conda install -c menpo opencv
conda install scipy
conda install matplotlib
conda install yaml
python -m pip install easydict

- tensorboardX
python -m pip install tensorboardX
python -m pip install tensorflow

### cython c 扩展 python

conda install cython
conda install cffi   # cffi是连接Python与c的桥梁

- msgpack

msgpack用起来像json，但是却比json快，并且序列化以后的数据长度更小，言外之意，使用msgpack不仅序列化和反序列化的速度快，数据传输量也比json格式小，msgpack同样支持多种语言

python -m pip install msgpack

## Ubuntu16.04+cuda
### 1.卸载系统里的Nvidia低版本显卡驱动
sudo apt-get purge nvidia*
### 2.把显卡驱动加入PPA

sudo add-apt-repository ppa:graphics-drivers
sudo apt-get update


### 3 GUI additional driver install

cuda9.2 nvidia-driver 396.54
cuda9.0 384.81

### 4.1 9.0 local deb

sudo dpkg -i cuda-repo-ubuntu1604-9-0-local_9.0.176-1_amd64.deb

  

4.2 9.2 local deb

1.  `sudo dpkg -i cuda-repo-ubuntu1604-9-2-local_9.2.148-1_amd64.deb`
2.  `sudo apt-key add /var/cuda-repo-<version>/7fa2af80.pub`
3.  `sudo apt-get update`
4.  `sudo apt-get install cuda`
    

### 5. env config

5.1 cuda9.0

sudo vim ~/.bashrc
export  PATH=/usr/local/cuda-9.0/bin:$PATH
export  LD_LIBRARY_PATH=/usr/local/cuda-9.0/lib64$LD_LIBRARY_PATH　
source ~/.bashrc

5.2 cuda9.2


### 6 cudnn

tar -xzvf cudnn-9.0-linux-x64-v7.tgz
sudo cp cuda/include/cudnn.h /usr/local/cuda/include
sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
sudo chmod a+r /usr/local/cuda/include/cudnn.h
sudo chmod a+r /usr/local/cuda/lib64/libcudnn*