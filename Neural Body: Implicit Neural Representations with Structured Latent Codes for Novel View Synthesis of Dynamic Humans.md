论文题目: Neural Body: Implicit Neural Representations with Structured Latent Codes for Novel View Synthesis of Dynamic Humans    
论文出处:CVPR 2021  
简单总结：
1.由Google提出的NeRF需要非常稠密视角来训练视角合成网络，所以比较适用于静态物体。而作者提出的Neural Body就是通过整合时序信息来获得足够多的3D shape observation，这里整合时序信息的实现用的是latent variable model，具体来说，就是通过定义一组latent code，从同一组latent code中生成不同帧的场景，这样就把不同帧观察到的信息和同一组latent code关联在了一起。经过训练，就能把视频各个帧的信息整合到这组latent code中，也就整合了时序信息。  
2.Neural Body用隐式神经表示动态人体。  
3.Neural Body用非常稀疏的视角输入,从稀疏多视图视频(sparse multi-view video)合成动态人体的视角。
<img src="https://user-images.githubusercontent.com/84011398/201474667-0eee5c98-7d4b-4435-842a-8f27dd8823ef.png" width = "600" />
<img src="https://user-images.githubusercontent.com/84011398/201474087-94b8c7f2-d59d-4279-a592-6b84754e7622.png" width = "400" />


## abstract
作者提出了Neural Body，首先整合视频帧，假设在不同帧共享同一组隐变量，因此可以自然地把不同帧观察到的信息和一组隐变量关联在了一起。可变形网格还为网络提供几何指导，以更有效地学习 3D 表示  

## introduction
1.Free-viewpoint videos自由视点视频，依靠密集的相机阵列或深度传感器，因此复杂的硬件导致昂贵的成本和受限制的使用场景。  
2.传统的基于重建的方法，由于稀疏视角导致人体部分被遮挡，会导致重建的人体不完整。  
3.NeRF可以概括为用一个MLP神经网络去隐式地学习一个静态3D场景，因此可以用密度和颜色的隐式场来实现对3D场景的表示。但是NeRF需要非常稠密视角来训练视角合成网络，因此并不适用于Sparse View稀疏视角。  
4.Lombardi(人名)采用了整合视频帧的方法，但是使用了多个不同的latent code作为同一个网络的输入，每一帧独立获得隐变量，因此缺乏约束   
5.Neural Body将给定的稀疏视角图像合成一个新的视角，且使用同一个latent code作为网络的输入，产生视频中不同帧的隐式表示  


## relative work
1.Image-based rendering  
2.Human performance capture  
3.Neural representation-based methods   


## method
3.1 generalization  
<img src="https://user-images.githubusercontent.com/84011398/201480534-a223623e-b236-4d67-8015-d358f3aab072.png" width = "700" />  
3.1.1 论文使用neural radiance field (NeRF)来表示一个3维场景。对于一个场景的任意一点，NeRF取出3维坐标和视角方向，输入到一个MLP中，输出color和density。  
<img src="https://user-images.githubusercontent.com/84011398/201481338-4b9cfd96-e83d-45ba-ad32-8aa3f7c14c40.png" width = "500">  

3.1.2 对于某一帧，论文定义了一组离散的local latent codes, Neural Body将local latent codes输入到一个3维卷积神经网络，得到一个latent code volume，这样就能通过插值得到空间中的任意一点的latent code。可以使用SMPL模型等对某一帧图像预测一个人粗糙的human mesh,然后将latent codes摆放到这个人身上,然后Neural Body把latent code送入NeRF的网络中，就能得到相应的color和density    


3.1.3 为了从同一组latent codes中生成不同帧的场景，则将这些latent codes摆放在一个可驱动人体模型的mesh vertex上，也就得到了structured latent codes。    
<img src="https://user-images.githubusercontent.com/84011398/201481867-b1e7f3eb-310b-4480-8177-c1491aadef5b.png" width = "300"> 

3.1.4 通过人体模型驱动structured latent codes的空间位置  
<img src="https://user-images.githubusercontent.com/84011398/201482239-6b49abcb-ca64-41b0-a315-c3ff4f672fe2.png" width = "500">   


3.1.5 从同一组structured latent codes生成不同帧的场景。  
对于不同的视频帧，Neural Body将latent codes摆放为对应的人体姿态，然后输入到NeRF中得到对应帧的3维场景。像NeRF一样，论文用volume rendering从图片中优化网络参数。通过在整段视频上训练，Neural Body实现了时序信息的整合。   
<img src="https://user-images.githubusercontent.com/84011398/201482059-ca8b2d94-2516-4040-b19a-e3a19cd7791b.png" width = "500">   


































