## 基于GAN模型的去噪算法

用的是GAN实现图像的去噪，算法很多，重点在数据集

![image-20240409202400815](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240409202400815.png)

→时频图

![image-20240409202450270](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240409202450270.png)

用时频图进行训练

然后再训练完成后通过边缘提取的方法提取图片中的波形包络，后续通过二值化，逐步转回到信号图

![image-20240409202738536](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240409202738536.png)

重点在数据集的制作上，网上大多是各种图片的数据集，没有信号的数据集，自己在原信号上加噪声（高斯），作为对比图。