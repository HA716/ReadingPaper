# SelfRecon复现记录
## 一定要找准真正报错的位置才能顺利解决问题，所以需要认真阅读错误提示
## 配置环境
### **1. 克隆环境**
```
conda env create -f environment.yml
conda activate SelfRecon
```
出现问题：在environment.yml中的"pip:"部分均无法顺利安装，只能手动安装
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
又出现问题：pip install openmesh==1.2.1 报错 Command errored out with exit status 1: ... check the logs for full command output
(当时解决的时候没记录到)


问题：执行install.sh时报错subprocess.CalledProcessError: Command ‘[‘ninja‘, ‘-v‘]‘ returned non-zero exit status 1.
解决方法：将anaconda环境下的  lib/python3.6/site-packages/torch/utils/cpp_extension.py文件将['ninja','-v']改成['ninja','--v'] 或者['ninja','--version']

