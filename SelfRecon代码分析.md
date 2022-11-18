# 整体框架  
整个项目是多个小模块的组合，要理解论文代码就需要理解每个小模块  
## 目录结构
### asset:资源  
### dataset
### FastMinv
### MCAcc
### MCGpu
### model
### smpl_pytorch
### utils 
### generate_boxs.py:  
- 模块功能:目标检测是指生成矩形边界框将物体框出来 并预测物体的类别，而generate_boxs.py的作用就是计算每张图片中人体对应的边界框  
- 目标检测的步骤:         
&emsp;(a.生成多个锚框  
&emsp;(b.用交并比IoU评价锚框的优劣，IoU越大越好  
&emsp;(c.训练锚框，得到锚框所含目标的类别 和 真实物体边界框相对锚框的偏移量offset  
&emsp;(d.输出预测边界框   

### generate_normals.py: 
- 模块功能：生成法线，法线贴图???  
- crop_image(img, rect):图像裁剪,根据generate_boxs.py中生成的矩形边界框对图像进行裁剪  
- def get_item(self, index):获取图片的相关信息，返回(图片名, 图片, 图片背景, 高度, 宽度, 人体的边界框)
- 从第168行到第206行都不知道在干啥？另外,也没读懂在哪里生成法线了


### infer.py:   
### people_snapshot_process.py:
- 模块功能：读取people_snapshot_public数据集中的数据,经过预处理后保存到本地文件中
- 读取masks.hdfs文件
- 读取视频的每一帧
- 读取相机参数
- 读取 SMPL模型的三个输入参数(betas、pose、trans)


### texture_mesh_extract.py:   
### testure_mesh_prepare.py:  
### train.py:   
- 三种粒度(coarse粗、medium中、fine细),主要是传到set_hierarchical_config()方法的Seg3dLossless中去了,有啥用呢？？  
- 从配置文件config.conf读取各种参数     
- optNet实例包含了SDF模型、LBS模型、CompositeDeformer(MLPTranslator + skinner)、相机camera、raster、render、shader    
- 调用set_hierarchical_config方法，分级地将config.conf配置文件的中train参数、loss参数传给optNet用于模型训练,将对应的batch_size传给dataloader    
- 使用Adam优化器+动态调整学习率的方法训练模型   


### install.sh:
### 数据集people_snapshot_public分析   
目录结构图如下  
<img src="https://user-images.githubusercontent.com/84011398/202379223-5175e285-fdc2-4fba-abd5-bb1afe122418.png" width ="500" />   
- camera.pkl是相机参数  
&emsp;camera_k:畸变矩阵     
&emsp;camera_rt:旋转矩阵    
&emsp;camera_t:平移矩阵   
&emsp;camera_c:主点坐标   
&emsp;camera_f:相机焦距    
&emsp;height:图像高度   
&emsp;width:图像宽度     
  
- consensus.obj 打开consensus是个3D人体模型  
<img src="https://user-images.githubusercontent.com/84011398/202399866-99670f75-52ff-44ae-93fc-9c2df2696246.png" width ="300" />   

  
- consensus.pkl    
&emsp;batas字段:(10,)    
&emsp;v_personal字段(6890, 3)     
tip: consensus.obj中展示的人体mesh是由SMPL生成的SMPL的输入为【体型参数beta：10个, 姿态参数theta：72个, 相机参数cam：3个】,输出为【具有6890个顶点的人体mesh】  

- female-3-casual.mp4 是一段自旋转人体视频。  
- keypoints.hdf5 存放关键点，数据形状是(648, 54)。  不过是什么关键点有54个？？
- masks.hdf5 数据形状是(648, 1080, 1080)  mask作用是啥，提取目标区域？  
&emsp;即有648张大小为(1080,1080)的掩膜。由camera.pkl可知，每张图像大小为(width=1080, height=1080),所以每张mask的大小也为(1080,1080)  
- reconstructed_poses.hdf5 【betas(10,)  ---  pose(649, 72)  --- trans(649, 3)】      
&emsp;对应了SMPL的输入为【体型参数beta：10个, 姿态参数theta：72个, 相机参数cam：3个】    
- tex-female-3-casual.jpg 纹理图

  
### Real-time 2D Multi-Person Pose Estimation on CPU: Lightweight OpenPose(新引入)  
- 作用:基于CPU实时推理检测一个骨架(由关键点和关键点之间的连接组成)来识别图像中每个人的人体姿势。这个姿势可能包含多达18个关键点:耳朵、眼睛、鼻子、脖子、肩膀、肘部、手腕、臀部、膝盖和脚踝  
<img src="https://user-images.githubusercontent.com/84011398/202466644-3cfffc45-8c08-415e-8760-c9fcd241440e.png" width="500">  


### PIFuHD: Multi-Level Pixel-Aligned Implicit Function for High-Resolution 3D Human Digitization(新引入)
准确的预测人体三维模型需要大范围的空间信息，精细的表面信息需要更高维度的特征信息。由于当前显卡的显存大小限制，之前的方法偏向于使用低分辨率的图片作为输入以让网络得到大范围的空间信息（大感受野），但是代价就是生成低精度（低分辨率）的三维模型。  
为了解决这个问题，作者提出一种可以端到端训练的Multi-level Architecture（两阶段）。在粗略估计阶段，输入是低分辨率的图片主要用于大致形状的预测；在精细重建阶段，输入是高分辨率的图片，联合粗略阶段的特征，输出高精度的三维预测。
   
   
   

   
   



