### 使用pip在自己用户下搭建环境：

- 按ctrl+h可查看隐藏文件夹

1. 把python3.6.7/python2.7.15的安装包拷进来并解压

2. 进入目录Python-3.6.7/python2.7.15，进行配置:
   `（python3）./configure --with-ssl --prefix='/home/username/.local'`
   `（python2）./configure --with-ssl --enable-unicode=ucs4 --prefix='/home/username/.local'`

3. 编译和安装
   `make`
   `make install`

4. 创建虚拟环境（可有可无）
   `python3 -m venv py3venv  --without-pip`
   激活`source ./py3venv/bin/activate`
   退出`deactivate`

5. 更改python默认版本
   `alias python='/home/username/.local/bin/python3.6'`
   `alias python2='/home/username/.local/bin/python2.7'`

6. pip3自带，安装pip2
   `wget https://bootstrap.pypa.io/get-pip.py`
   `python2 get-pip.py`

7. 分别安装python3和python2的tensorflow以及pytorch

   使用`pip/pip2/pip3 --version`查看一下对应哪个python 也可以用alias自己设定。
   
   ```shell
   
      pip3 install numpy
      pip3 install tensorflow-gpu==1.2
   pip2 install numpy
      pip2 install https://mirrors.tuna.tsinghua.edu.cn/tensorflow/linux/gpu/tensorflow_gpu-1.2.0-cp27-none-linux_x86_64.whl
      
   pip3 install https://download.pytorch.org/whl/cu80/torch-0.4.1-cp36-cp36m-linux_x86_64.whl
      pip3 install torchvision
   
      pip2 install https://download.pytorch.org/whl/cu80/torch-0.4.1-cp27-cp27mu-linux_x86_64.whl
      pip2 install torchvision
      #if the above command does not work, then you have python 2.7 UCS2, use this command
      #pip2 install https://download.pytorch.org/whl/cu80/torch-0.4.1-cp27-cp27m-linux_x86_64.whl
   ```
   
8. 进入python2/3，`import tensorflow/torch/torchvision` `tensorflow/torch/torchvision.__version__`查看版本