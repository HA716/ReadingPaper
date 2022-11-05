# SelfRecon复现记录
## 一定要找准真正报错的位置才能顺利解决问题，所以需要认真阅读错误提示
## 配置环境
### **1.配置CUDA、cuDNN**
1.1 确定显卡驱动、CUDA、cuDNN等之间的对应关系  

1.1.1 查看服务器NVIDIA驱动版本           
![image](https://user-images.githubusercontent.com/84011398/199973292-34e19c5b-966e-4b7d-a730-acb65f7ef85b.png)         
                   
1.1.2 不同版本cuda对应的NVIDIA驱动版本。可安装的最高CUDA版本为11.6，因此略低于11.6的cuda版本都是支持的。由于服务器原本的cuda运行版本是11.1，而论文要求的是cuda_11.3，所以需要自己重新安装cuda_11.3。            
![image](https://user-images.githubusercontent.com/84011398/199973462-dd3b711e-583c-43b7-8e36-098233345b24.png)      
                          
1.1.3 cuDNN是一个SDK，是一个专门用于神经网络的加速包，注意，它跟我们的CUDA没有一一对应的关系，即每一个版本的CUDA可能有好几个版本的cuDNN与之对应，但一般有一个最新版本的cuDNN版本与CUDA对应更好。在NVIDIA官网上可以找到实验要求的cuDNN_8.2.0  
![image](https://user-images.githubusercontent.com/84011398/199975994-313505c5-f81f-4d37-a945-159c9c34bc74.png)    

1.1.4 下载并安装CUDA_11.3、cuDNN_8.2.0  
参考https://blog.csdn.net/hizengbiao/article/details/88625044    

1.1.5 拓展:在多个版本的CUDA间切换，只需要在切换环境时直接在.bashrc文件里更改之前配置环境时加入的路径代码(export PATH、export LD_LIBRARY_PATH)即可。如下图所示
```
 ·切换后，通过nvcc -V和which nvcc可以检测CUDA是否切换成功。  
 ·切换后，通过cat /home/usernameXXX/GPU/CUDA_11.3/temp/cuda/include/cudnn_version.h | grep CUDNN_MAJOR -A 2 检测cuDNN是否切换成功   
```
![image](https://user-images.githubusercontent.com/84011398/200005933-c37c42b4-77ce-4202-988b-7a7ba4d05505.png)   










### **2. 克隆环境**
```
conda env create -f environment.yml
conda activate SelfRecon
```
2.1 安装过程中，Anaconda镜像找不到指定版本的pytorch源,所以需要自行下载，下面这个网站能找到各种版本的pytorch包。  
```
https://anaconda.org/pytorch/pytorch/files?version=1.10.2
```

2.2 出现问题：在environment.yml中的"pip:"部分均无法顺利安装，只能手动安装
```
pip install cycler==0.11.0
pip install fonttools==4.31.2
pip install fvcore==0.1.5.post20220305
pip install h5py==3.6.0
pip install iopath==0.1.9
pip install kiwisolver==1.4.0
pip install matplotlib==3.5.1
pip install opencv-python==4.5.5.64
pip install openmesh==1.2.1
pip install packaging==21.3
pip install portalocker==2.4.0
pip install pycocotools==2.0.4
pip install pyhocon==0.3.59
pip install pyparsing==2.4.7
pip install python-dateutil==2.8.2    
pip install pyyaml==6.0
pip install tabulate==0.8.9
pip install termcolor==1.1.0
pip install torch-scatter==2.0.9
pip install tqdm==4.63.0
pip install trimesh==3.10.5
pip install yacs==0.1.8
```
1.2 问题：pip install openmesh==1.2.1 报错 Command errored out with exit status 1: ... check the logs for full command output。  
已解决：对于一些pip安装不上的包，可以通过下载包对应的whl文件(https://www.lfd.uci.edu/~gohlke/pythonlibs/#lxml)， 然后通过pip install + whl文件名进行安装


1.3 问题：执行install.sh时报错subprocess.CalledProcessError: Command ‘[‘ninja‘, ‘-v‘]‘ returned non-zero exit status 1.  
踩坑：一定不能将anaconda环境下的  env/SelfRecon/python3.8/site-packages/torch/utils/cpp_extension.py文件将['ninja','-v']改成['ninja','--v'] 或者['ninja','--version'],将代码['ninja','-v'] 改成 ['ninja','--v'] 或者 ['ninja','--version'],因为这并没有解决根本问题，而且修改会导致其他错误，后续编译也不会通过，根本原因不是ninja -v无法识别!!!!!!!   

原因分析：ninja未正确安装或者torch版本不匹配。最后发现，是ninja未安装，所以执行下述命令安装ninja即可
```
pip install ninja
```  

1.4 继续报错g++：找不到XXX.o文件，查看文件目录之后发现确实缺失这个文件，原因是ninja编译失败了
![image](https://user-images.githubusercontent.com/84011398/197696347-169cf6b0-9205-48e6-ad96-51d9525090f6.png)
已解决:原因是踩了1.3的坑(不能改成['ninja','--v'] 或者['ninja','--version'],因为这不是根本原因)

1.5 报错：  
![image](https://user-images.githubusercontent.com/84011398/200109157-8fdf1057-2b94-4b8d-b30d-540662664d94.png)  
解决方法:
报了torch api中cloneable.h文件的错误，经过尝试，将cloneable.h文件中46行，58行，70行三句
```
copy->parameters_.size() == parameters_.size()

copy->buffers_.size() == buffers_.size()

copy->children_.size() == children_.size()
```  
分别改成
```
copy->parameters_.size() == this -> parameters_.size()

copy->buffers_.size() == this -> buffers_.size()

copy->children_.size() == this -> children_.size()
```  
保存后再次安装成功。至此，install.sh算是跑通了，泪目。。。


