﻿#  余弦退火从启动学习率机制 
【导语】主要介绍 ** 在pytorch 中实现了余弦退火从启动学习率机制，支持 warmup 和 resume 训练。并且支持自定义下降函数，实现多种重启动机制。 觉得好用记得点个 star 哈...

代码： https://github.com/Huangdebo/CAWB

## 1. 多 step 重启动
![设定 cawb_steps 之后，便可实现多步长余弦退火重启动学习率机制;调整epoch_scale可以实现更复杂的变化机制](https://img-blog.csdnimg.cn/8c38d5131f8d4a26a3784c9564112960.jpg?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAd29uZGVyZnVsX2hkYg==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
设定 cawb_steps 之后，便可实现多步长余弦退火重启动学习率机制。每次重启动时，开始学习率会乘上一个比例因子 step_scale。调整  step_scale 和 epoch_scale 等参数，可以实现学习率跳变的时候是上升还是下降。也可以调整中间的 step 不用走完一个退火过程，保持较高的学习率，实现更复杂的学习率变化机制。

## 2. 正常余弦退火机制
![如果 cawb_steps 为 [], 则会实现正常的余弦退火机制](https://img-blog.csdnimg.cn/50ad359e69f2459cbbe3268a1be821e4.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAd29uZGVyZnVsX2hkYg==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
如果 cawb_steps 为 [], 则会实现正常的余弦退火机制，在整个 epochs 中按设定的 lf 机制一直下降

## 3. warmup
![设定 warmup_epoch 之后便可实现学习率的 warmup 机制](https://img-blog.csdnimg.cn/e2e1ad5fe3d0496aa4d674018c1191d8.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAd29uZGVyZnVsX2hkYg==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
设定 warmup_epoch 之后便可实现学习率的 warmup 机制。warmup_epoch 结束后则按设定的 cawb_steps 实现重启动退火机制。

## 4. resume
![设定 last_epoch 便可实现 resume 训练](https://img-blog.csdnimg.cn/a17f69421aa74c44a1acd3ad737094f9.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAd29uZGVyZnVsX2hkYg==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
设定 last_epoch 便可实现 resume 训练，接上之前中断的训练中的学习率。

## 5. 自定义下降函数
![自定义下降函数](https://img-blog.csdnimg.cn/3d16857453d44cabb93294edf7ce1ede.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAd29uZGVyZnVsX2hkYg==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
可通过自定义下降函数，实现多种重启动机制

```python
# lf = lambda x, y=opt.epochs: (((1 + math.cos(x * math.pi / y)) / 2) ** 1.0) * 0.9 + 0.1  
lf = lambda x, y=opt.epochs: (1.0 - (x / y)) * 0.8 + 0.2 
scheduler = CosineAnnealingWarmbootingLR(optimizer, epochs=opt.epochs, step_scale=0.7, 
                                         steps=opt.cawb_steps, lf=lf, batchs=len(data), warmup_epoch=0)
```

## 6. 实验结果
本实验是在 COCO2017中随机选出 10000 图像和 1000 张图像分别作为训练集和验证集。检测网络使用 yolov5s，学习率调整机制分别原版的 cos 和 本文实现的 CAWB。

#### 6.1 学习率:
```python
# yolov5
lf = lambda x: ((1 - math.cos(x * math.pi / steps)) / 2) * (y2 - y1) + y1
scheduler = lr_scheduler.LambdaLR(optimizer, lr_lambda=lf)

# 本文
lf = lambda x, y=opt.epochs: (((1 + math.cos(x * math.pi / y)) / 2) ** 1.0) * 0.65 + 0.35 
scheduler = CosineAnnealingWarmbootingLR(optimizer, epochs=opt.epochs, steps=opt.cawb_steps, step_scale=0.7,
                                         lf=lf, batchs=len(train_loader), warmup_epoch=3, epoch_scale=4.0)
```
![cos 和 cawb 学习率](https://img-blog.csdnimg.cn/dbfa53959c5d4f368ffae7340c8d9050.jpg?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAd29uZGVyZnVsX2hkYg==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)
#### 6.2 map:

#### 6.2.1 cos：
![mAP_0.5 = 0.294;  mAP_0.5:0.95 = 0.161](https://img-blog.csdnimg.cn/9633cce6adf744d097181b81ac4d1fd1.jpg?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAd29uZGVyZnVsX2hkYg==,size_16,color_FFFFFF,t_70,g_se,x_16#pic_center)

mAP_0.5 = 0.294;  mAP_0.5:0.95 = 0.161

#### 6.2.2 CAWB ：
![mAP_0.5 = 0.302; mAP_0.5:0.95 = 0.165](https://img-blog.csdnimg.cn/6fe463388c43475abe495de50da2ee1b.jpg?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAd29uZGVyZnVsX2hkYg==,size_16,color_FFFFFF,t_70,g_se,x_16#pic_center)

mAP_0.5 = 0.302; mAP_0.5:0.95 = 0.165

# 7. 结论
在实验中使用了 CAWB 学习率机制时候，mAP_0.5 和 mAP_0.5:0.95 都提升了一丢丢，而且上升趋势更加明显，增加 epochs 可能提升更大。

改变 CAWB 的参数可以实现更多形式的学习率变化机制。增加学习率突变就是想增加网络跳出局部最优的概率，所以不同数据集可能合适不同的变化机制。小伙伴们在其他数据集上尝试之后，记得来提个 issue 哈...

代码： https://github.com/Huangdebo/CAWB
