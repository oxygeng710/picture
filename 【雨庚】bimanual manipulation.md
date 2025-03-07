# peract2：

## 框架：

![image-20250307145239537](https://raw.githubusercontent.com/oxygeng710/picture/main/image-20250307145239537.png)

### 处理步骤

1. **体素化**：将多个RGB-D图像融合成一个3D体素网格，表示机器人工作空间的体积信息，每个体素对应空间中的一个小立方体
2. **特征学习**：使用Perceiver IO transformer对体素网格和语言指令进行编码，捕捉其空间和语义特征
3. **动作预测**：基于特征生成每只机械臂的动作指令，包括：
   - **6自由度（DoF）末端执行器位置**：在3D空间中的位置和方向
   - **抓取器状态**：打开或关闭
   - **运动规划器碰撞标志**：是否需要避障
4. **动作执行**：将预测动作发送给机器人控制器，驱动机械臂运动

## 复现结果：

![a39ef5ce318dffc33a913aabd171f21](https://raw.githubusercontent.com/oxygeng710/picture/main/a39ef5ce318dffc33a913aabd171f21.png)

​	论文中评估结果的方式仅有成功率一项，这里可能需要运行eval.py才能够得到成功率，运行这个时也是有与之前同样的问题

qt.qpa.plugin: Could not load the Qt platform plugin "xcb" in "/share/home/HCI/yugeng/user/CoppeliaSim_Player_V4_1_0_Ubuntu16_04" even though it was found.
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.

Available platform plugins are: eglfs, linuxfb, minimal, minimalegl, offscreen, vnc, xcb.

根据[CoppeliaSim的安装与xrdp远程连接](https://docs.ruotao.tech/#/CoppeliaSim_xrdp)中的内容，需要连接桌面解决，但实际连接桌面后仍然报此项错误。网上有一些解决方法，如检查/share/home/HCI/yugeng/user/CoppeliaSim_Player_V4_1_0_Ubuntu16_04下的platform文件夹中的文件是否完整，查询后并未找到网上残缺部分的内容。此次报错与之前的无法display问题不同，无法直接通过阅读报错的调用去修改代码解决，还在继续查阅相关方法的github中的issue栏寻找解决方案。

# Anybimanual:

## 框架：

![image-20250307161119801](https://raw.githubusercontent.com/oxygeng710/picture/main/image-20250307161119801.png)

1. **语言分支**：
   - 使用预训练的文本编码器解析双臂操作指令，将其转换为具有高层次语义的语言嵌入。
   - 技能管理器通过自适应地协调原始技能，为不同的手臂增强语言嵌入。这个过程通过线性组合和补偿来实现，将组合的技能原语与初始的双臂语言嵌入进行拼接，将其传递给相应的单臂策略。
2. **视觉分支**：
   - RGB-D输入被体素化为体素空间作为观察。
   - 使用3D稀疏体素编码器对体素观察进行编码，以获得信息丰富的体积表示。
   - 视觉对齐器生成空间软掩码，以对齐单臂策略模型的视觉表示，使其与预训练阶段的表示匹配。这有助于减少单臂和双臂系统之间的观察差异，提高策略的可转移性。

## 复现：

之前说训练这个需要peract2的checkpoint，原因是：

![2dbaec00edd494c8597fb37e368c39b](https://raw.githubusercontent.com/oxygeng710/picture/main/2dbaec00edd494c8597fb37e368c39b.png)

还有anybimanual的readme中所写的训练指令：bash scripts/train.sh BIMANUAL_PERACT 0,1 12345 ${exp_name}，以及peract2的github网址中的repository名为peract_bimanual，让我误以为这里需要的checkpoint是peract2的，后面利用peract2的checkpoint训练加载模型时出现mismatch报错，然后重新阅读论文和readme之后确认这里使用的是peract的checkpoint。

目前正在尝试复现论文中提到的最好的方法组合PerAct + AnyBimanual。训练过程与peract2类似，但是多了一个processing files的过程，根据论文内容，可能是文中提到的自动技能发现方法，以无监督的方式从离线双臂操作数据集中自动识别和提取出基本的技能原语。

![image-20250307172100095](https://raw.githubusercontent.com/oxygeng710/picture/main/image-20250307172100095.png)
