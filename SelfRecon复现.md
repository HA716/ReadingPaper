# SelfRecon复现记录
## 一定要找准真正报错的位置才能顺利解决问题，所以需要认真阅读错误提示
## 配置环境
### **1.配置CUDA、cuDNN**
确定显卡驱动、CUDA、cuDNN等之间的对应关系  

1.1 查看服务器NVIDIA驱动版本   
<img src="https://user-images.githubusercontent.com/84011398/199973292-34e19c5b-966e-4b7d-a730-acb65f7ef85b.png" width="500">          
                   
1.2 不同版本cuda对应的NVIDIA驱动版本。可安装的最高CUDA版本为11.6，因此略低于11.6的cuda版本都是支持的。由于服务器原本的cuda运行版本是11.1，而论文要求的是cuda_11.3，所以需要自己重新安装cuda_11.3。    
<img src="https://user-images.githubusercontent.com/84011398/199973462-dd3b711e-583c-43b7-8e36-098233345b24.png" width="500">           
                          
1.3 cuDNN是一个SDK，是一个专门用于神经网络的加速包，注意，它跟我们的CUDA没有一一对应的关系，即每一个版本的CUDA可能有好几个版本的cuDNN与之对应，但一般有一个最新版本的cuDNN版本与CUDA对应更好。在NVIDIA官网上可以找到实验要求的cuDNN_8.2.0      
<img src="https://user-images.githubusercontent.com/84011398/199975994-313505c5-f81f-4d37-a945-159c9c34bc74.png" width="500">      

1.4 下载并安装CUDA_11.3、cuDNN_8.2.0  
参考https://blog.csdn.net/hizengbiao/article/details/88625044    

1.5 拓展:在多个版本的CUDA间切换，只需要在切换环境时直接在.bashrc文件里更改之前配置环境时加入的路径代码(export PATH、export LD_LIBRARY_PATH)即可。如下图所示
```
 ·切换后，通过nvcc -V和which nvcc可以检测CUDA是否切换成功。  
 ·切换后，通过cat /home/usernameXXX/GPU/CUDA_11.3/temp/cuda/include/cudnn_version.h | grep CUDNN_MAJOR -A 2 检测cuDNN是否切换成功   
```  
<img src="https://user-images.githubusercontent.com/84011398/200005933-c37c42b4-77ce-4202-988b-7a7ba4d05505.png" width="700">       






### **2. 克隆环境**
```
conda env create -f environment.yml
conda activate SelfRecon
```
2.1 出现问题：安装过程中，Anaconda镜像找不到指定版本的pytorch源,所以需要自行下载，下面这个网站能找到各种版本的pytorch包。  
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
对于一些pip安装不上的包，可以通过下载包对应的whl文件([点击即可下载各种whl文件](https://www.lfd.uci.edu/~gohlke/pythonlibs/#lxml))， 然后通过pip install + whl文件名进行安装

报错：通过本地下载openmesh.whl,然后pip install openmesh.whl 报错 is not a supported wheel on this platform      
原因分析：所下载安装的openmesh.whl和系统python版本不匹配导致的。下载cp39的包，但是SelfRecon的环境是cp38，所以无法安装。     
<img src="https://user-images.githubusercontent.com/84011398/203070835-6d4c21c3-ecb6-4720-b17c-1e2d8e364ab8.png" width="500">    
解决方法：从[conda-forge](https://anaconda.org/conda-forge/openmesh-python/files?version=1.2.1) 中下载 适合python3.8版本的openmesh,然后conda install --use-local XXX.tar.bz2进行本地安装     





2.3 问题：执行install.sh时报错subprocess.CalledProcessError: Command ‘[‘ninja‘, ‘-v‘]‘ returned non-zero exit status 1.  
踩坑：一定不能将anaconda环境下的  env/SelfRecon/python3.8/site-packages/torch/utils/cpp_extension.py文件将['ninja','-v']改成['ninja','--v'] 或者['ninja','--version'],将代码['ninja','-v'] 改成 ['ninja','--v'] 或者 ['ninja','--version'],因为这并没有解决根本问题，而且修改会导致其他错误，后续编译也不会通过，根本原因不是ninja -v无法识别!!!!!!!   

原因分析：ninja未正确安装或者torch版本不匹配。最后发现，是ninja未安装，所以执行下述命令安装ninja即可
```
pip install ninja
```  

2.4 继续报错g++：找不到XXX.o文件，查看文件目录之后发现确实缺失这个文件，原因是ninja编译失败了
<img src="https://user-images.githubusercontent.com/84011398/197696347-169cf6b0-9205-48e6-ad96-51d9525090f6.png" width="700">        
已解决:原因是踩了1.3的坑(不能改成['ninja','--v'] 或者['ninja','--version'],因为这不是根本原因)

2.5 报错：  
<img src="https://user-images.githubusercontent.com/84011398/200109157-8fdf1057-2b94-4b8d-b30d-540662664d94.png" width="700">      
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


2.6 安装pytorch3d。由于服务器上访问不了下面的链接，只能本地下好再传到服务器。
```
wget -O pytorch3d-0.4.0.zip https://github.com/facebookresearch/pytorch3d/archive/refs/tags/v0.4.0.zip
unzip pytorch3d-0.4.0.zip
cd pytorch3d-0.4.0 && python setup.py install && cd ..
```

2.7 安装pytorch3d时报错：  
<img src="https://user-images.githubusercontent.com/84011398/200111765-4a95b157-aacb-4b56-84ac-9affbf276efe.png" width="700">      
分析原因：编译器gcc/g++的版本只有5.4.0，版本太低了       
解决方法：升级编译器版本 或 修改源码让编译器认识      
```
原本：q[idx_top_k] = {pz, f, signed_dist, bary_clip.x, bary_clip.y, bary_clip.z};   
   
修改为：q[idx_top_k]= std::tuple<float, int, float, float, float, float>(pz, f, signed_dist, bary_clip.x, bary_clip.y, bary_clip.z);
```
<img src="https://user-images.githubusercontent.com/84011398/202843012-294b90ca-c9d2-4a94-8080-f5dce97c9b81.png" width="700">
   

2.8 下载作者提供的[SMPL_model](https://mailustceducn-my.sharepoint.com/:f:/g/personal/jby1993_mail_ustc_edu_cn/EqosuuD2slZCuZeVI2h4RiABguiaB4HkUBusnn_0qEhWjQ?e=c6r4KS) ,并将pkl文件全部放入smpl_pytorch/model目录下     
<img src="https://user-images.githubusercontent.com/84011398/203214921-d07e3ec3-f757-40fa-8d47-fdfcd30e0590.png" width="600">  

   
     
### **3.Run on PeopleSnapshot Dataset**   
3.1 下载数据集people_snapshot_public并将数据集放在$ROOT0目录下,其中$ROOT0 = /home/huangbaoren/Code/ROOT/ROOT0       
<img src="https://user-images.githubusercontent.com/84011398/202954658-d6069bd5-a798-4c82-8f2a-cc5eda33afd7.png" width="400">      
  
     
3.2 以female-3-casual为例，运行下面代码提取数据
```
python people_snapshot_process.py --root $ROOT0/people_snapshot_public/female-3-casual --save_root $ROOT0/female-3-casual
```
运行结果:     
<img src="https://user-images.githubusercontent.com/84011398/202960000-02f51a5e-7fc5-409e-937e-e9814858b1ba.png" width="400">       
<img src="https://user-images.githubusercontent.com/84011398/202955555-7c551e29-68c0-4ed4-8679-63330791fbbd.png" width="600">    

   
3.3 提取法线    
- 3.3.1 下载 [PIFuHD](https://shunsukesaito.github.io/PIFuHD/)源码到$ROOT1,下载[Lightweight Openpose](https://github.com/Daniil-Osokin/lightweight-human-pose-estimation.pytorch) 源码到$ROOT2.     
其中，ROOT1 = /home/huangbaoren/Code/ROOT/ROOT1, ROOT2 = /home/huangbaoren/Code/ROOT/ROOT2    

- 3.3.2 将generate_normals.py复制到$ROOT1,将generate_boxes.py复制到$ROOT2  
<img src="https://user-images.githubusercontent.com/84011398/203064479-a25a96f6-62b9-4275-b5ac-9b084b9a2828.png" width="900">   
   
   
- 3.3.3 目标检测，为每张图片的人体生成边界框  
```
cd $ROOT2
python generate_boxs.py --data $ROOT0/female-3-casual/imgs
```

报错：generate_boxs.py找不到checkpoint_iter_370000.pth,确实找不到。       
原因分析：这个checkpoint_iter_370000.pth是通过 [Lightweight Openpose](https://github.com/Daniil-Osokin/lightweight-human-pose-estimation.pytorch)训练出来的，而在[Colab PIFuHD demo](https://colab.research.google.com/drive/11z58bl3meSzo6kFqkahMa35G5jmh2Wgt#scrollTo=UtMjWGNU5STe)中可以直接运行得到这个模型，下载到本地服务器即可            
<img src="https://user-images.githubusercontent.com/84011398/203010598-56885808-f8bc-4395-8036-a5a2af521c72.png" width="600">   
    
解决方法：从[Colab PIFuHD demo](https://colab.research.google.com/drive/11z58bl3meSzo6kFqkahMa35G5jmh2Wgt#scrollTo=UtMjWGNU5STe) 直接运行得到checkpoint_iter_370000.pth这个模型，然后下载到本地服务器，放到$ROOT2目录下即可。         


运行结果：可见在$ROOT0/imgs目录下，为每张图片的人体都生成了对应的边界框。         
<img src="https://user-images.githubusercontent.com/84011398/203014874-e2a4590a-a088-416c-8275-f2ae2cc95ee5.png" width="700">     

   
- 3.3.4 提取法线     
```
cd $ROOT1
python generate_normals.py --imgpath $ROOT0/female-3-casual/imgs
```
报错：找不到checkpoints/pifuhd.pt   
<img src="https://user-images.githubusercontent.com/84011398/203064913-8df2aaf7-2440-4b04-83a5-e424c36d0b99.png" width="700">   
  
原因分析：原因同上(3.3.3),因为pifuhd.pt这是一个训练好的模型，需要自己去下载，在[Colab PIFuHD demo](https://colab.research.google.com/drive/11z58bl3meSzo6kFqkahMa35G5jmh2Wgt#scrollTo=UtMjWGNU5STe)中能找到下载地址(即[pifuhd.pt](https://dl.fbaipublicfiles.com/pifuhd/checkpoints/pifuhd.pt))   

解决方法：从[Colab PIFuHD demo](https://colab.research.google.com/drive/11z58bl3meSzo6kFqkahMa35G5jmh2Wgt#scrollTo=UtMjWGNU5STe)中下载[pifuhd.pt](https://dl.fbaipublicfiles.com/pifuhd/checkpoints/pifuhd.pt),然后放到$ROOT1目录下即可。      


 运行结果：成功生成法线贴图。            
 <img src="https://user-images.githubusercontent.com/84011398/203080966-25b5c713-132e-4a88-b59c-c3a406b76dc1.png" width="700">      
 <img src="https://user-images.githubusercontent.com/84011398/203081525-1461c217-ce6c-4145-8097-0852869f40d4.png" width="700">


- 3.3.5 训练模型
```
CUDA_VISIBLE_DEVICES=0 python train.py --gpu-ids 0 --conf config.conf --data $ROOT/female-3-casual --save-folder result
```
运行结果：训练结果存放在$ROOT/female-3-casual/result,最终得到训练好的optNet。总共有200个epoch,实际上3h训练了10个epoch,预计要60h，但作者说大约只需要花费1天时间训练(啥原因?)。(仅截图部分运行结果)    
- 0h    
<img src="https://user-images.githubusercontent.com/84011398/203215314-53d1314f-b836-4d93-827a-b3ed7aebe915.png" width="700">     
<img src="https://user-images.githubusercontent.com/84011398/203215715-c13517ff-ece0-48d3-8659-41fcc9ce3e4d.png" width="700">     

- 6h   
<img src="https://user-images.githubusercontent.com/84011398/203276697-a37cc8f9-3d63-4917-b043-b758fd0e87af.png" width="700">      
      
- 59h   
<img src="https://user-images.githubusercontent.com/84011398/203804390-3663b332-003e-4f51-b168-8e4e028c9167.png" width="700">     
<img src="https://user-images.githubusercontent.com/84011398/203959459-c5e99fd6-fbad-4fa7-98dd-c93dce57e501.png" width="700">       



### **4.Inference**    
- 4.1 运行下面代码生成渲染的图像和网格    
```
CUDA_VISIBLE_DEVICES=0 python infer.py --gpu-ids 0 --rec-root $ROOT/female-3-casual/result/ --C
```
运行结果：  
<img src="https://user-images.githubusercontent.com/84011398/203961058-05dfc42f-ba15-4a46-9bed-bada0811f614.png" width="900">     


### **5.Texture**     
- 5.1 下载[VideoAvatar](https://graphics.tu-bs.de/people-snapshot)，并将texture_mesh_extract.py复制到VideoAvatar目录下。简化并参数化模板网格tmp.ply，并将结果网格保存到$ROOT/female-3-casual/result/template/uvmap.obj中       
<img src="https://user-images.githubusercontent.com/84011398/203990564-c36bd265-835a-40ec-a97b-8df3a517c253.png" width="500">   
<img src="https://user-images.githubusercontent.com/84011398/204464728-538f5df9-1f9a-4102-8646-a7f0289062f4.png" width="500">   

-5.2 执行下面代码，从tmp.ply中提取点、面、相机等数据，生成用于纹理提取的数据                 
```
CUDA_VISIBLE_DEVICES=0 python texture_mesh_prepare.py --gpu-ids 0 --num 120 --rec-root $ROOT/female-3-casual/result/
```
5.2.1 报错：No module named scipy     
<img src="https://user-images.githubusercontent.com/84011398/204000254-f0015f20-f2a1-4c0f-8636-88d919642f72.png" width="500">        
解决方法：作者在environment.yml中并未提及scipy包，但实际上需要这个包，所以自行安装    
```
pip install scipy
```
       
5.2.2 报错：
<img src="https://user-images.githubusercontent.com/84011398/204002857-e76f1c49-a63d-4c34-a228-178938fa099f.png" width="500">    

原因分析:Readme中提到"you need to simplify and parameterize the template mesh tmp.ply yourself, then save the result mesh as $ROOT/female-3-casual/result/template/**uvmap.obj**. And run the following code to generate the data for texture extraction"。现在报错就是因为并没有生成uvmap.obj文件，即assert语句其后面的条件是错误的，因为路径args.rec_root/template/uvmap.obj不存在，所以会出现上述报错。      

**解决方法**：use meshlab to simplify the mesh + use blender to parameterize the mesh with your manually marked seams         
- 首先，用MeshLab简化模型。([步骤](https://lingyunfx.com/meshlab-remove-face/) import mesh --> 选择XXX.ply --> filters --> Remeshing,Simplification and Reconstruction  --> Simplifiction:Quadric Edge Collapse Decimation(选择planar simplification均匀简化) --> Apply --> (保存新模型)File --> Export Mesh As )。修改Percentage reduction参数，想让模型的面数越少这个值就越小。通常数值范围在 0.99 ~ 0.1， 减面后的面数会约等于当前模型面数乘以这个参数值。这里以推荐参数0.1为例，点击Apply,然后就得到了简化后的模型

- 其次，人体展UV。用Blender手工标记缝合线，参数化模型
(步骤：文件 --> 导入 --> 选择Wavefront(.obj) -->  选择自己需要导入的.obj文件 --> UV Edit --> 边选择模式 --> 根据[b站人体展UV教程](https://www.bilibili.com/video/BV1JA411M7Ba/?vd_source=5518096779c5f1567ef92f6fef425468) 展开人体 --> 导出UV布局图)   
<img src="https://user-images.githubusercontent.com/84011398/204432210-fdce2dd5-08ed-486e-84ee-b80f2cd37876.png" width="700">    
        
      
**运行结果**:生成tex_predata.npz文件        
<img src="https://user-images.githubusercontent.com/84011398/204118258-917986bf-13ff-4a47-998f-d8298508dd63.png" width="500">         


- 5.3提取法线：进入VideoAvatar所在目录执行下面命令         
```
CUDA_VISIBLE_DEVICES=0 python texture_mesh_extract.py --tmp-root $ROOT/female-3-casual/result/template
```


5.3.1 报错No module named 'opendr'
<img src="https://user-images.githubusercontent.com/84011398/204122524-0d8e2c2d-ab7c-45fa-821a-90211f41ea4c.png" width="500">    
解决方法：参考[csdn]{https://blog.csdn.net/qq_40520596/article/details/110955392} 选择手动安装opendr       


5.3.2 再次执行命令，报错     
<img src="https://user-images.githubusercontent.com/84011398/204123084-d82a2364-6f6f-4f16-9338-fa1b719a6e89.png" width="500">     
解决方法：修改源码，在np.load参数列表上加上allow_pickle=True    
```
np.load(xxxx,allow_pickle=True)  
```
        
5.3.3 再次执行命令，成功              
<img src="https://user-images.githubusercontent.com/84011398/212271102-11e07873-bd3d-426b-9278-300b3822d5ae.png" width="700">




























  

