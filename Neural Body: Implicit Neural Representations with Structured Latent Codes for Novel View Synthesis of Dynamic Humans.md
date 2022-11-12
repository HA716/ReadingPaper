论文题目: Neural Body: Implicit Neural Representations with Structured Latent Codes for Novel View Synthesis of Dynamic Humans    
论文出处:CVPR 2021  
简单总结：
由Google提出的NeRF需要非常稠密视角来训练视角合成网络，所以比较适用于静态物体。而作者提出的Neural Body就是通过整合时序信息来获得足够多的3D shape observation，这里整合时序信息的实现用的是latent variable model，具体来说，就是通过定义一组隐变量，从同一组隐变量中生成不同帧的场景，这样就把不同帧观察到的信息和一组隐变量关联在了一起。经过训练，就能把视频各个帧的信息整合到这组隐变量中，也就整合了时序信息。

# abstract
作者提出了Neural Body，首先整合视频帧，假设在不同帧共享同一组隐变量，因此可以自然地把不同帧观察到的信息和一组隐变量关联在了一起。可变形网格还为网络提供几何指导，以更有效地学习 3D 表示
# introduction
1.Free-viewpoint videos自由视点视频，依靠密集的相机阵列或深度传感器，因此复杂的硬件导致昂贵的成本和受限制的使用场景。  
2.传统的基于重建的方法，由于稀疏视角导致人体部分被遮挡，会导致重建的人体不完整。  
3.NeRF可以概括为用一个MLP神经网络去隐式地学习一个静态3D场景，因此可以用密度和颜色的隐式场来实现对3D场景的表示。但是NeRF需要非常稠密视角来训练视角合成网络，因此并不适用于Sparse View稀疏视角  
4.

# relative work
# method
