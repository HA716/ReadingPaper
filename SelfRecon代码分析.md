## 整体框架  
整个项目是多个小模块的组合，要理解论文代码就需要理解每个小模块  
### 目录结构
#### asset:资源  
#### dataset
#### FastMinv
#### MCAcc
#### MCGpu
#### model
#### smpl_pytorch
#### utils 
#### generate_boxs.py:  
#### generate_normals.py:  
#### infer.py:   
#### people_snapshot_process.py:
- 模块功能：读取people_snapshot_public数据集中的数据,经过预处理后保存到本地文件中
- 读取masks.hdfs文件
- 读取视频的每一帧
- 读取相机参数
- 读取 SMPL模型的三个输入参数(betas、pose、trans)


#### texture_mesh_extract.py:   
#### testure_mesh_prepare.py:  
#### train.py:   
#### install.sh:
#### 数据集people_snapshot_public分析   
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

  



   
   
   

   
   



