# 整体框架  
整个项目是多个小模块的组合，要理解论文代码就需要理解每个小模块  
## 论文
### Abstract 
- 作者将 3D 人体的 隐式表达和显式表达相结合 ，使用符号距离场 (隐式表达) 表示标准姿态空间下的人体，在视频的每一帧提取人体网格 (显式表达)，借助学习到的前向变形场 (Forward Deformation Field) 将标准姿态空间下的人体网格变形到当前姿态下，并通过极小化轮廓损失、光滑损失等能量项来优化人体网格的形状，以得到具有更多细节的人体网格。和当前已有方法相比，文章方法不需要提前获取目标人体的模板网格 (对比 LiveCap, DeepCap 等方法)，也不需要大量带纹理的人体模型用于网络训练 (对比 PIFu、PIFuHD 等方法)，只需通过自监督的方法便可以从穿着任意服装的人体视频重建出带有众多细节的人体网格。总体来说，文章方法是一种 基于优化的方法 ，需要针对每个个体进行人体模型的优化、重建。
### Method
文章方法的流程如下图所示,作者会同时维护显式和隐式两种人体几何表达，在每一帧使用前向变形场 (Forward Deformation Field) 来生成每一帧的显式人体网格。变形场由两个部分组成 :        
- 第一部分 使用一个可学习的 MLP 来表达每一帧的 人体非刚性变形 (Non-rigid Deformation)，以为标准姿态下的人体网格增加更多细节
- 第二部分 是蒙皮变形场 (Skinning Deformation Field)，用于将标准姿态下的网格变形到当前帧姿态；   
       
        
对于显式人体网格，作者使用可微的  Mask 损失、变形约束损失和骨架光滑损失来控制人体形状;  
对于隐式人体曲面，作者使用非刚性光线投射 (Non-rigid Ray Casting) 来计算光线和变形后的隐式曲面的交点，再使用Implicit Rendering Network估计该点沿着输入视角方向的颜色，最后使用颜色信息和交点特性来增加几何细节。   
<img src="https://user-images.githubusercontent.com/84011398/202778679-22352929-abee-417d-b34d-8caa59f35f0c.png" width="1000">       


#### Pipeline
对于一段含有 帧的人体自转视频，作者首先采用 VideoAvatar[2] 的方法来生成 SMPL 人体模型的初始形状 和每一帧的姿态参数 ，再预定义一个姿态参数来生成初始的标准姿态空间的人体网格 ，并使用 来初始化隐式和显式表达。

#### Canonical Implicit SDF
作者将标准姿态空间下的模板人体网格 表示为符号距离场 (SDF) 的零等值面，SDF 由包含可学习权重 的 MLP 表示：作者通过 IGR[5] 方法使用初始人体网格 来初始化。 

#### Deformation Fields
作者使用骨骼动画来驱动大范围的人体运动，为了表达服装的非刚性变形，作者增加了一个变形场来刻画非刚性变形。因此变形场包含两部分变形，分别是Non-rigid Deformation Field和Skinning Transformation Field。这样通过将(Non-rigid Deformation Field和Skinning Transformation Field)相结合，可以得到最终的变形场D，其将第i帧的条件变量hi和 SMPL 的姿态参数θi作为输入，从而将标准姿态空间下的人体网格增加细节后变换到第i帧图像姿态所对应的空间中      
- Non-rigid Deformation Field：  
使用包含可学习权重Φ的 MLP 来表达非刚性变形场d。对于第i帧,MLP 将可优化的条件变量hi 作为输入,使用与第i帧相对应的非刚性变形 对标准姿态空间下的人体网格顶点进行变形。效果是人体的服装、皮肤软组织等会发生形状的改变。   
- Skinning Transformation Field
给定第i帧的姿态参数θi，需要定义标准姿态空间到当前姿态的蒙皮变形场W。效果是得到一个姿态改变之后的人体，也就是说标准姿态空间的点 变换到了 当前帧空间。    

#### Differentiable Non-rigid Ray-casting
- 作者使用可变形 SDF 渲染方法，使用显式网格来帮助找到射线与变形后的零等值面的交点。首先从标准姿态空间Sη中提取一个标准人体模板网格T,通过变形场D可以将标准网格T变换得到第i帧图像对应网格Ti，理论上Ti是当前空间Si的近似，因此从相机c沿着方向v发出一条射线，这条射线与Ti的交点x 就是 这条射线与当前空间Si的交点的近似。通过重心坐标插值(Barycentric Interpolation)找到当前空间Si的点x在标准空间Sη中对应的点p,此时的点p就是 点x经过D^-1变形的结果的近似。
<img src="https://user-images.githubusercontent.com/84011398/202881405-21e94060-b252-40a4-b1b5-d5418eea55e1.png" width="500">    
- 对于第i帧的光线、相机位置c、视点方向v、变形场Di的参数ψi和f的参数η唯一地确定了p，因此可以把p作为表示这些参数的函数。    
- Ray-casting(光线投射) 是指将三维场景渲染为二维图像。从成像屏幕上每一个像素点出发，沿着视线方向发射出一条光线，当这条光线穿过体数据时，沿着光线方向等距离采样，最后可以计算出这条光线对应的屏幕上像素点的颜色值    

#### Implicit Rendering Network
论文 IDR 提出可以使用一个 MLP 来估计渲染方程，并能在一定程度上将光照和材质解耦，对于刚性物体，通过Differentiable Non-rigid Ray-casting后，对于第i帧的光线、相机位置c、视点方向v、变形场Di的参数ψi和f的参数η唯一地确定了点p，因此可以把p作为表示这些参数的函数。因此就可以将零等值面上的点的位置、法向、视角方向和全局几何特征向量作为输入，估计该点沿着输入视角方向的颜色。  
<img src="https://user-images.githubusercontent.com/84011398/202884059-f34c1b40-7ece-41fe-97f1-257699d6cdea.png" width="500">   


#### Loss Function
- Explicit Loss      
&emsp;(1.Mask Loss: 当前姿态下的人体网格渲染的 Mask 与图片中人体 Mask 的 IoU 损失；       
&emsp;(2.Deformation Regularization Loss: 对非刚性变形的约束，即标准姿态空间下的人体增加非刚性变形后进行蒙皮变换后的网格与直接进行蒙皮变换的网格的顶点应该比较接近；    
&emsp;(3.Skeleton Smoothness Loss ：人体骨骼的连续性约束    
   
- Implicit Loss     
&emsp;(1.Color Loss：通过 Non-rigid Ray-casting 计算的点经过 Implicit Rendering Network 渲染的颜色与输入的图片上的采样点的颜色应该一致   
&emsp;(2.Normal Loss：作者使用 PIFuHD 预测图片对应的法向图，对于法向图也有一致性约束    
&emsp;(3.Rigidity Loss：对变形场 的约束，使其尽可能保持刚性以减少扭曲          
&emsp;(4.Eikonal Loss：  

#### Explicit/Implicit Consistency
- 在每次显式迭代结束后，作者加入一个一致性损失，以使得隐式 SDF 与更新后的显式网格保持一致。
- 在每一步优化，作者先极小化显式损失来计算更新后的显式人体网格并保留X的梯度，再计算隐式损失和一致性损失并累积到X'的梯度，最后使用 Adam 算法计算出来的梯度来更新变量X。    


### Avatar Generation
借助变形场，可以提取出拓扑一致的人体网格序列。使用 VideoAvatar 的方法可以计算得到人体网格的纹理图，从 Skinning Transformation Field 可以提取出蒙皮权重，这样便能得到一个可驱动的人体 Avatar，使用 SMPL 的姿态参数便可以轻松驱动 Avatar。Figure 8 展示了生成的 Avatar 及驱动的结果。  
<img src="https://user-images.githubusercontent.com/84011398/202769420-eedf2319-1b80-40a7-a7cf-5240374fdf3d.png" width="500">     
   
### 结果
- 下图展示了 VideoAvatar 与文章方法的重建结果的误差比较，两种方法都是从自转一圈的人体视频进行重建，可以看到文章方法重建的误差更小、精度更高。   
<img src="https://user-images.githubusercontent.com/84011398/202771327-215706e7-4515-40c9-952f-a3c8dc9984c2.png" width="500">   
  
- 下图展示了文章方法与 PaMIR、NeuralBody 和 VideoAvatar 重建结果的比较，文章的结果更加准确、细节更加丰富。     
<img src="https://user-images.githubusercontent.com/84011398/202771675-1ab3be79-a46d-4723-9774-51c95226fecb.png" width="500">   

- 下图展示了从智能手机拍摄的视频进行人体重建的结果，SelfRecon 能够较为准确地重建出人体模型，贴上纹理之后渲染的图片与输入图片非常相似。     
<img src="https://user-images.githubusercontent.com/84011398/202772102-24be15cd-6c67-4d7c-9123-cd37e62ae5a3.png" width="500">   

- 下面视频展示了文章方法在 People-Snapshot 数据集上的优化结果，右侧是每一帧的人体网格，中间一列是渲染的带纹理人体模型的结果，与左侧的输入视频较为一致。 
<img src="https://user-images.githubusercontent.com/84011398/202772515-ca4ed700-c81f-4fe7-81cf-62582666a112.png" width="500">    








## 目录结构
### asset:资源  
### dataset
### FastMinv
### MCAcc
- 由Monoport和ImplictSegCUDA 来构建Marching Cubes加速
### MCGpu
- 由MarchingCubes_CppCuda提供GPU MC模块(MCGpu)
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
- 模块功能：生成渲染的网格和图像  
- 


### people_snapshot_process.py:
- 模块功能：读取people_snapshot_public数据集中的数据,经过预处理后保存到本地文件中   
- 读取masks.hdfs文件
- 读取视频的每一帧
- 读取相机参数
- 读取 SMPL模型的三个输入参数(betas、pose、trans)

### testure_mesh_prepare.py:  
- 模块功能：生成用于纹理提取的数据  
### texture_mesh_extract.py: 
- 模块功能：提取纹理

### train.py:   
- 设置三种分辨率resolutions,即三种粒度(coarse粗、medium中、fine细),主要是传到set_hierarchical_config()方法的Seg3dLossless中去了,有啥用呢？？  
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
   
   
   

   
   



