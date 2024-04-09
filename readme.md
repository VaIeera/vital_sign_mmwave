## Introduction

### EMD经验模态分解

注意：使用时先加载安装工具包里的emd_visu相关工具

![image-20240409164220778](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240409164220778.png)

demo为（https://blog.csdn.net/qq_40061206/article/details/120664537）简单例子，模仿到EMDTest里281行后EMD模块

```matlab
%% EMD
%创建emd对象
x=phi;
t=0:0.000001:0.001023;//x与t长度要相等
imf = emd(x);
[m, n]=size(imf);
%作图
emd_visu(x,t,imf);
```

![image-20240409160816308](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240409160816308.png)

然后做结果分析

- 功率谱计算

```matlab
for i = 1:size(imf, 1)
    [pxx, f] = pwelch(imf(i, :), [], [], [], Fs); % 使用 pwelch 计算功率谱
    % 绘制功率谱图
    figure;
    plot(f, 10*log10(pxx));
    xlabel('Frequency (Hz)');
    ylabel('Power/Frequency (dB/Hz)');
    title(['Power Spectral Density of IMF ', num2str(i)]);
    grid on;
end
```

![image-20240409162329928](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240409162329928.png)

类似上图

> 功率谱是**功率谱密度函数**的简称，它定义为单位频带内的信号功率。 它表示了信号功率随着频率的变化情况，即信号功率在频域的分布状况。 功率谱表示了信号功率随着频率的变化关系。

- 带宽范围中心位置

```matlab
% 假设 imf 是经验模态分解得到的 IMF 分量矩阵，每行是一个 IMF
% 初始化存储带宽范围的向量
bw = zeros(size(imf, 1), 1);

% 计算每个 IMF 分量的带宽范围
for i = 1:size(imf, 1)
    % 使用峭度来估计带宽范围，这里使用了峭度的计算方法，你也可以选择其他的方法
    kurtosis_val = kurtosis(imf(i, :)); % 计算峭度
    bw(i) = sqrt(2 * kurtosis_val); % 带宽范围定义为 2倍峭度的平方根，可以根据需求调整
end

% 计算带宽范围中心位置
bw_center = zeros(size(bw));
for i = 1:length(bw)
    bw_center(i) = mean(bw(i)); % 计算每个分量带宽范围的中心位置
end

% 显示带宽范围中心位置
disp('Bandwidth Center Positions:');
disp(bw_center);
```

输出结果：

![image-20240409163933237](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240409163933237.png)

  

> 功率谱计算、带宽范围估计以及带宽范围中心位置计算在信号处理和分析中具有重要的作用，可以体现信号的多个重要性质：
>
> 1. **功率谱计算**：
>    - **频率成分分析**：功率谱能够帮助分析信号在不同频率上的能量分布情况，从而揭示信号的频率成分和频率特性。
>    - **频谱特征展示**：通过功率谱图可以直观地展示信号的频谱特征，例如频率成分的强度和分布情况。
> 2. **带宽范围估计**：
>    - **频谱宽度评估**：带宽范围的估计可以反映信号的频谱宽度，即信号在频率域上的展宽程度，从而描述信号的频谱特性。
>    - **信号特征分析**：不同带宽范围的信号具有不同的频率特征，通过带宽范围的估计可以帮助分析信号的频率特性和频带分布情况。
> 3. **带宽范围中心位置计算**：
>    - **频谱中心位置**：带宽范围中心位置反映了信号频谱的中心位置，即频率成分的主要集中区域，可以用来描述信号的中心频率。
>    - **频谱分布特征**：通过带宽范围中心位置的计算，可以了解信号频率分布的主要特征，例如频率成分的集中度和分布情况。
>
> 这些分析工具和方法可以帮助我们更全面地理解信号的频率特性、频率成分的分布情况以及信号在频域上的表现，对于信号处理、特征提取、模式识别等领域具有重要意义。通过对功率谱、带宽范围以及中心位置等参数的分析，可以更深入地研究信号的频域特性，为后续的信号处理和分析提供基础和参考。

### VMD变分模态分解

http://khsci.com/docs/index.php/2021/08/19/vmd/

https://zhuanlan.zhihu.com/p/396775790

```matlab
%% VMD
% 1.创建vmd对象
x=phi(1:999);
fs = 1000; % 示例采样频率，根据实际情况调整
t=0:0.000001:0.000998;% 时间向量，1秒内的采样点数根据需要调整
% 2.VMD分解参数设置
alpha=2000;  % alpha   - 惩罚因子
tol=1e-7;    % tol     - 收敛容差，是优化的停止准则之一，可以取 1e-6~5e-6
K=4;         % K       - 指定分解模态数
type = 2;
% type： 如果type没有传入参数，则该函数内将type默认设置为1
%        当type的值为1时，则优先采用MATLAB自带库中的函数进行分解，此时vmd函数的用法与MATLAB自带函数一致；
%        当type的值为2时，则强制采用第三方vmd函数进行分解

% 以下输入参数在使用MATLAB内置函数的时候不需要输入（可以置为nan）
tau=0;      % tau     - time-step of the dual ascent ( pick 0 for noise-slack )
DC=1;       % DC      - true if the first mode is put and kept at DC (0-freq)
init=1;     % init    - 0 = all omegas start at 0
            %           1 = all omegas start uniformly distributed
            %           2 = all omegas initialized randomly
% 3.绘制VMD分解图
imf = pVMD(x,fs, alpha, K, tol, 2, tau, DC, init); %p文件可调用，无法查看源码
% 4.绘制VMD分解图及其频谱图
imf = pVMDandFFT(x,fs, alpha, K, tol, 2, tau, DC, init);
```

![image-20240409171738174](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240409171738174.png)

![image-20240409172808686](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240409172808686.png)

其中，分解模态数K可以根据需要调整，当前场景下K=3可以达到要求，与测试代码相对比，心跳呼吸频谱是一样的

![image-20240409171955556](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240409171955556.png)



![image-20240409172025148](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20240409172025148.png)