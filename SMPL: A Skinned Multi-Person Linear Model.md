# SMPL
论文题目: SMPL: A Skinned Multi-Person Linear Model
论文出处: ACM ToG 2015
简单总结：


## Abstact    
SMPL是一个人体形状(body)和姿势形状(pose)相关的可学习模型，模型的参数(parameters)从数据中学习，包括静止姿势模板(rest pose template)、混合权重(blend weights)、姿势相关的混合变形(pose-dependent blend shapes)、身份相关的混合变形(identity-dependent blend shapes)以及从顶点到关节位置的回归(a regressor from vertices to joint locations)。与以前的模型不同，姿势相关的混合变形是姿势旋转矩阵元素的线性函数。

## Introduce      
1.作者的目标是创建一个可以代表不同的身体形状的人体模型，身体形状能随姿势自然变形，并表现出与真人一样的软组织运动。并且要求该模型是是快速渲染、易于部署、可兼容的。       
2.模型比较    
<img src="https://user-images.githubusercontent.com/84011398/201516076-6320c191-60b1-4403-9cab-63fa055748fc.png" width="500">   
3.SMPL姿势相关的混合变形是姿势旋转矩阵元素的线性函数 


## Related Work  

<img src="https://user-images.githubusercontent.com/84011398/201516076-6320c191-60b1-4403-9cab-63fa055748fc.png" width="500">  

## Model Formulation

## Training

## SMPL Evaluation








