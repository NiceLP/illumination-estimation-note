# illumination-estimation-note
# 论文笔记

## 光源估计

**(2022 AAAI) Rendering-Aware HDR Environment Map Prediction from a Single Image**

问题提出：

1. 从有限视野（FOV）LDR图像估计HDR全景照明图非常复杂（照明条件、表面材质、场景几何、相机参数）
2. 参数化的光照模型难以准确和充分估计全频率照明（SH解释高频能力弱，SG能表示高质量高光和镜面反射，但描述低频能力弱），直接估计全频率的HDR照明图效果差

<div align=center>
  <img src="images/(2022 AAAI)Rendering-Aware HDR Environment Map Prediction from a Single Image4.png" alt="Different light estimation networks">
</div>

解决方案：

1. 使用两阶段的框架从LDR FOV图像估计HDR全景照明图
2. 使用Transformer估计主光源的SG表示高频照明（锚点与输入图像之间有强空间关系，self-attention可以模拟锚点之间的成对交互），用CNN估计二阶SH表示低频照明
3. 将估计的SG和SH参数转换为高斯照明图和漫射辐照度图（参数可视化），以两幅图作为先验，用GAN生成HDR全景照明图
4. 采用一个可微渲染层，将渲染损失也作为一个损失项监督GAN的生成

<div align=center>
  <img src="images/(2022 AAAI) Rendering-Aware HDR Environment Map Prediction from a Single Image1.png" alt="The structure of SG Regression Module">
  
  <img src="images/(2022 AAAI) Rendering-Aware HDR Environment Map Prediction from a Single Image2.png" alt="The structure of SH Regression Module">
  
  <img src="images/(2022 AAAI) Rendering-Aware HDR Environment Map Prediction from a Single Image3.png" alt="The structure of the generative network">
</div>

**(2020 ECCV) Object-based Illumination Estimation with Rendering-aware Neural Networks**

问题提出：

1. 实际的AR应用中需要实时估计局部场景的照明
2. 从单个图像进行场景的逆渲染难以实现，需要简化照明模型和材质类型
3. 逆渲染的优化需要高计算成本，难以同时保证高精度和实时性能
4. 神经网络需要知道渲染的物理规则，分析物理渲染不适用于实时应用，漫反射和镜面反射规则不同，且与物体材质有关
5. 目前基于场景的估计方法，都是从场景（全景图）中包含的一块有限区域中的信息估计光照，区域中的内容量决定了估计光的质量
6. 前人训练端到端逆渲染网络，但只假设漫反射着色，并因为渲染器限制只能产生低分辨率EM

解决方案：

1. 提出一种从单个对象（物体）的RGBD外观及其周围局部区域的图像（FOV）估计整个场景的照明（HDR Environment Map），端到端直接估计HDR照明图
2. 仍使用基于物理的逆渲染，中间步骤使用深度学习进行加速，输入RGBD图像（可计算法线信息，辐照度图也由深度图计算），使用深度学习计算反射率、漫反射、镜面反射，并将漫反射着色转换为角度照明域（空间域转换为角度域），将镜面着色几何映射到其镜面方向，通过角度融合产生HDR EM
3. 使用物理规则可以将镜面反射等信息投影到照明方向
4. 使用递归卷积层考虑照明环境的时间相干性（连贯性）
5. 使用基于物理的逆渲染和神经网络结合，比纯基于学习的方法估计精度更高
6. 只从场景中单个对象（物体）的着色信息估计HDR EM，不依赖场景中的内容量，不对表面反射进行假设，作为基于场景的估计方法的补充

<div align=center>
  <img src="images/(2020 ECCV) Object-based Illumination Estimation with Rendering-aware Neural Networks1.png" alt="Overview of system">
</div>

结论：

1. 具有高频照明的灯光（主灯光）比低频灯光（近似环境照明）更难估计
2. 基于场景的方法和基于对象的方法结合是一个未来的研究方向

**(2022 EGSR)SkyGAN Towards Realistic Cloud Imagery for Image Based Lighting**

<div align=center>
  <img src="images/(2022 EGSR)SkyGAN Towards Realistic Cloud Imagery for Image Based Lighting2.png" alt="SkyGAN Overview">
</div>

问题提出：

1. 晴空模型（clear sky models）虽然能够表示真实的室外照明，但包含的信息过于简单，渲染出来的图像真实感不足
2. 体积云模拟计算成本高，希望将云整合到天空中，直接通过HDR EM渲染出来
3. 拍摄出的天空是静态的，受拍摄时的位置和天气条件的影响，希望能够以参数的形式改变天空的外观
4. 一般的天空/大气模型通过物理的解析，只能得到晴朗无云的天空

解决方案：

1. 通过天空分析模型从一组真实照片中拟合与太阳位置相对应的晴空图像
2. 将晴空图像经过编码器编码到潜在空间，作为生成器的输入信号，生成器生成一对晴天和阴天图像（包含云雾等信息），晴天图像与拟合出来的晴空图像计算Loss，阴天图像放入鉴别器判断真假

<div align=center>
  <img src="images/(2022 EGSR)SkyGAN Towards Realistic Cloud Imagery for Image Based Lighting1.png" alt="SkyGAN architecture">
</div>

**(2021 CVPR)HDR Environment Map Estimation for Real-Time Augmented Reality**

问题提出：

1.  在实时AR系统中应用从LDR图像预测HDR EM（iPhone XS中9ms以内），AR应用中由于相机相对于虚拟对象移动，需要频繁估计新的照明，并且需要保证照明的时间连贯性
2.  直接端到端生成HDR EM，单个模型，不需要深度图进行训练

解决方案：

1. 使用四通道输入（RGB和二进制掩码），生成网络输入全景大小的图像，已知区域以外的像素用随机信号填充
2. 使用外观特征提取器一次性提取整个数据集的特征（ORB描述符和平均颜色），并对数据集特征进行聚类，为不同类型分配语义标签（如房间类型），将聚类损失加入鉴别器，提供监督分类信息
3. 不同光源和场景有较大强度变化，所以采用对数对HDR图像进行强度归一化
4. 通过渲染图像的损失监督生成器成本太高，提出一种新的投影损失（ProjectionLoss）

<div align=center>
  <img src="images/(2021 CVPR)HDR Environment Map Estimation for Real-Time Augmented Reality1.png" alt="Overview of EnvMapNet">
  
  <img src="images/(2021 CVPR)HDR Environment Map Estimation for Real-Time Augmented Reality2.png" alt="Overview of Network">
</div>

**(2022 VR)An improved augmented-reality method of inserting virtual objects into the scene with transparent objects**

问题提出：

1.  当真实场景中存在透明物体时，透明物体的折射率和粗糙度的差异会影响光照估计和虚实融合的效果

2.  现有的方法只考虑在真实场景中仅有不透明物体时插入虚拟对象，没有考虑场景中的透明物体对虚拟对象的影响，具有不同折射率和粗糙度的透明物体对其周边虚拟物体的渲染产生不同效果
3.  先前的研究考虑到了镜面和透明对象的材质，但没有考虑粗糙度，使得虚拟对象融合的不够真实
4.  对透明物体的最佳折射率求解只能在间隔采样后的有限采样点下进行

解决方案：

1. 将微表面模型和半球照明模型加入反向路径追踪，同时求解不透明物体和透明物体的材质和照明
2. 输入多视角图像，利用现有方法重建几何模型，将透明物体和不透明物体进行分离
3. 对几何模型设计了两步优化方法来优化场景中物体的照明和材质。第一步，使用反向路径追踪和半球照明模型联合优化不透明物体的照明和材质；第二步，将透明物体添加到场景中，基于第一步中照明和材质的估计结果，一起优化场景中的所有材质和照明

<div align=center>
  <img src="images/(2022 VR)An improved augmented-reality method of inserting virtual objects into the scene with transparent objects1.png" alt="Overall framework">
</div>

**(2022 ECCV)StyleLight HDR Panorama Generation for Lighting Estimation and Editing**

问题提出：

1. LDR图像能够显示的亮度范围有限，无法显示超出范围的亮度
2. FOV图像只包含部分场景，不同数量、形状、方向的模糊光源会影响整个场景的光照分布
3. FOV图像之外的场景光照取决于场景中物体材质、几何等信息，FOV图像无法提供

解决方案：

1. 将LDR FOV生成LDR全景图，和LDR全景图生成HDR全景图集成到一个框架内
2. LDR和HDR共享同一个生成器，在生成的输出端采用色调映射（Tone Mapping）进行LDR和HDR的转换
3. 使用两个鉴别器分别鉴别LDR和HDR
4. 提出一种焦点遮蔽GAN反演方法，从LDR的合成分支找出潜在编码，然后通过HDR的合成分支生成对应的HDR EM，可以通过潜在编码编辑光照

<div align=center>
  <img src="images/(2022 ECCV)StyleLight HDR Panorama Generation for Lighting Estimation and Editing1.png" alt="Overview of StyleLight">
</div>



