# ToDo

1. 继续完成anybimanual剩余任务的评估
2. 之前可视化存在一些问题，并且根据peract2作者提供的信息，可视化会延长eval时间，准备有一些能看到的数据结果之后再继续这个内容

- [ ] update：整理好目前的结果，checkpt折线图，anybimanual框架
- [ ] 考虑fail cases，bimanual *v.s.* single arm
- [ ] 改进方向：3DGS+anybimanual
- [ ] 自己找双臂相关，这些单臂工作双臂上面用，单臂上做也可以

# Anybimanual失败案例原因分析（4.21）

！可视化下成功率比原来还低

把测试任务分为两类，需要精细夹取物体（如将盘子夹起来）和不需要精细夹取物体（如按压按钮）



- 需要精细夹取物体的任务：

  需要精细夹取物体的任务多数成功率为0，主要有两个失败原因：

  1. 双臂的协同性很差，经常进行抓取点的争抢
  2. 当双臂都在各自需要抓取的位置时，抓取的性能很差，几乎很难成功完成任务所需要的抓取

- 不需要精细夹取物体的任务：

  成功率总体上高一些，但也不高，并且也有部分很低，主要失败原因：

  1. 视觉上，无法准备判断夹爪的位置导致机械臂没有到达指定的位置，以及无法正确判断颜色（判断颜色也可能是指令理解的问题）
  2. 指令理解上，如在push box任务中，机械臂定位到了将要推到的位置，而非box



改进方向的考虑（3DGS+anybimanual）：

​		目前还在了解3DGS的相关内容，需要精细夹取物体的任务中很多任务都因为协同性导致失败，感觉可能即便在视觉上做改进也无法解决这部分的问题



# Anybimanual复现（4.9）

## anybimanual框架：

![image-20250331195059424](https://raw.githubusercontent.com/oxygeng710/picture/main/image-20250331195059424.png)

1. **语言分支**：
   - 使用预训练的文本编码器解析双臂操作指令，将其转换为语言嵌入。
   - 技能管理器通过自适应地协调原始技能，为不同的手臂增强语言嵌入。这个过程通过线性组合和补偿来实现，将组合的技能原语与初始的双臂语言嵌入进行拼接，将其传递给相应的单臂策略。
2. **视觉分支**：
   - RGB-D输入被体素化为体素空间作为观察信息。
   - 使用编码器对体素观察进行编码，以获得视觉信息。
   - 视觉对齐器生成掩码，对于左手则将右手遮蔽，使其与预训练阶段的表示匹配。

## 结果：

方法名-LF 为单臂策略通过leader-follower方式转化成双臂策略，

PerAct2为双臂策略，

PerAct2 + Pretraining为直接预训练两个单臂策略peract，

此次复现的方法为PerAct + AnyBimanual，70000表示综合最优的ckpt，best_ckpt指在不同ckpt下能达到的最大成功率，

每个方法进行了25个episode的评估。

注：Bi3DDA为CoRL2024 workshop MRM-D Poster上的内容，其使用多任务的方式进行训练双臂扩散策略；KStar Diffuser为arxiv上的论文，加入了与机器人运动学相关的内容，训练扩散策略，未说明以多任务还是单任务的方式进行训练，仅进行了部分任务的评估。两个方法都未提供代码。

| method                  | pick_laptop | pick_plate | straighten_rope | lift_ball | lift_tray | push_box |
| ----------------------- | ----------- | ---------- | --------------- | --------- | --------- | -------- |
| RVT -LF                 | 0%          | 0%         | 0%              | 12%       | 4%        | 16%      |
| RVT-LF + AnyBimanual    | 0%          | 8%         | 8%              | 16%       | 8%        | 24%      |
| PerAct [50]-LF          | 0%          | 4%         | 8%              | 12%       | 4%        | 16%      |
| PerAct-LF + AnyBimanual | 4%          | 4%         | 4%              | 12%       | 8%        | **24%**  |
| PerAct2                 | 8%          | 16%        | 4%              | **40%**   | 4%        | 0%       |
| PerAct2 + Pretraining   | 4%          | 20%        | 12%             | 12%       | 12%       | 8%       |
| PerAct + AnyBimanual    | **8%**      | **32%**    | **24%**         | 32%       | **12%**   | 16%      |
| 70000ckpt               | 4%          | 4%         | 4%              | 32%       | 8%        | 0%       |
| best_ckpt               | 8%          | 28%        | 8%              | 32%       | 8%        | 24%      |
|                         |             |            |                 |           |           |          |
| Bi3DDA                  | 71%         | 66%        | 50%             | 92%       | 59%       | 74%      |
| KStar Diffuser          | 43%         |            |                 | 98%       |           | 83%      |

| method                  | put_bottle_in_fridge | dual_push_buttons | handover_item | sweep_to_dustpan |
| ----------------------- | -------------------- | ----------------- | ------------- | ---------------- |
| RVT -LF                 | 0%                   | 12%               | 0%            | 0%               |
| RVT-LF + AnyBimanual    | 0%                   | 20%               | 4%            | 4%               |
| PerAct [50]-LF          | 0%                   | 8%                | 0%            | 28%              |
| PerAct-LF + AnyBimanual | 4%                   | 8%                | 0%            | **32%**          |
| PerAct2                 | 0%                   | 12%               | 0%            | 0%               |
| PerAct2 + Pretraining   | 0%                   | 64%               | 4%            | 8%               |
| PerAct + AnyBimanual    | **8%**               | **84%**           | **8%**        | 4%               |
| 70000ckpt               | 0%                   | 52%               | 60%           | 12%              |
| best_ckpt               | 4%                   | 52%               | 72%           | 16%              |
|                         |                      |                   |               |                  |
| Bi3DDA                  | 79%                  | 96%               | 19%           | 98%              |
| KStar Diffuser          |                      |                   |               | 89%              |

| method                  | take_tray_out_of_oven | handover_item_easy |
| ----------------------- | --------------------- | ------------------ |
| RVT -LF                 | 4%                    | 0%                 |
| RVT-LF + AnyBimanual    | 4%                    | 8%                 |
| PerAct [50]-LF          | 0%                    | 4%                 |
| PerAct-LF + AnyBimanual | 0%                    | 8%                 |
| PerAct2                 | 0%                    | 24%                |
| PerAct2 + Pretraining   | 4%                    | 28%                |
| PerAct + AnyBimanual    | **4%**                | **28%**            |
| 70000ckpt               | 0%                    | 12%                |
| best_ckpt               | 4%                    | 28%                |
|                         |                       |                    |
| Bi3DDA                  | 15%                   | 20%                |
| KStar Diffuser          |                       | 27%                |

## 折线图：

如果每个任务一个折线图需要占用大量的位置，并且也无法提取出有用的信息，所以把所有任务都放进同一张折现图中，整体的成功率都较低。

<img src="https://raw.githubusercontent.com/oxygeng710/picture/main/image-20250408160545602.png" alt="image-20250408160545602" style="zoom: 67%;" />

## 失败案例：

task：dual_push_buttons

图1中测试的button穿模导致无法正确识别button，图2中机器臂未正确按压到按钮，还有部分失败案例无法按压对应颜色的按钮

<table>
    <tr>
        <td ><center><img src="https://raw.githubusercontent.com/oxygeng710/picture/main/image-20250408155904426.png">图1 </center></td>
        <td ><center><img src="https://raw.githubusercontent.com/oxygeng710/picture/main/image-20250408162515383.png">图2 </center></td>
    </tr>
</table>
task：handover_item

图3中因为抓取红色物体时，夹爪碰到了蓝色物体，导致未能正常夹起红色物体，图4未准确判断对物体的距离提前关闭了夹爪导致未能夹起（这两种出现次数较少）

图5夹错颜色的物体（是一个重要原因，出现次数较多），图6在交接时未能夹住物体（对成功与失败的判定有问题，即便有时掉落了也算入了成功，在交接时很难稳稳地夹住物体）

<table>
    <tr>
        <td ><center><img src="https://raw.githubusercontent.com/oxygeng710/picture/main/image-20250408221330737.png">图3  </center></td>
	<td ><center><img src="https://raw.githubusercontent.com/oxygeng710/picture/main/image-20250409170733950.png">图4  </center></td>
    </tr>
    <tr>
	<td ><center><img src="https://raw.githubusercontent.com/oxygeng710/picture/main/image-20250409172057250.png">图5  </center></td>
	<td ><center><img src="https://raw.githubusercontent.com/oxygeng710/picture/main/image-20250409180210323.png">图6  </center></td>
    </tr>
</table>

其他失败案例后续继续补充。



# Anybimanual复现（3.24）

| ckpt            | pick_laptop | pick_plate | straighten_rope | lift_ball | lift_tray | push_box |
| --------------- | ----------- | ---------- | --------------- | --------- | --------- | -------- |
| 0               | 0%          | 16%        | 0%              | 0%        | 0%        | 0%       |
| 10000           | 0%          | 0%         | 0%              | 0%        | 0%        | 0%       |
| 20000           | 0%          | 0%         | 0%              | 0%        | 0%        | 0%       |
| 30000           | 0%          | 16%        | 4%              | 0%        | 0%        | 24%      |
| 40000           | 0%          | 0%         | 8%              | 12%       | 0%        | 0%       |
| 50000           | 0%          | 28%        | 4%              | 0%        | 0%        | 0%       |
| 60000           | 4%          | 12%        | 8%              | 0%        | 8%        | 0%       |
| 70000           | 4%          | 4%         | 4%              | 32%       | 8%        | 0%       |
| 80000           | 4%          | 16%        | 0%              | 0%        | 0%        | 0%       |
| 90000           | 4%          | 0%         | 4%              | 0%        | 0%        | 0%       |
| 100000          | 8%          | 4%         | 8%              | 0%        | 0%        | 8%       |
|                 |             |            |                 |           |           |          |
| best_ckpt       | 8%          | 28%        | 8%              | 32%       | 8%        | 24%      |
| **anybimanual** | **8%**      | **32%**    | **24%**         | **32%**   | **12%**   | **16%**  |

| ckpt            | put_bottle_in_fridge | dual_push_buttons | handover_item | sweep_to_dustpan |
| --------------- | -------------------- | ----------------- | ------------- | ---------------- |
| 0               | 0%                   | 20%               | 0%            | 0%               |
| 10000           | 0%                   | 0%                | 4%            | 0%               |
| 20000           | 0%                   | 8%                | 0%            | 0%               |
| 30000           | 0%                   | 24%               | 0%            | 0%               |
| 40000           | 4%                   | 16%               | 8%            | 16%              |
| 50000           | 0%                   | 40%               | 24%           | 0%               |
| 60000           | 0%                   | 48%               | 40%           | 0%               |
| 70000           | 0%                   | 52%               | 60%           | 8%               |
| 80000           | 0%                   | 32%               | 48%           | 0%               |
| 90000           | 0%                   | 24%               | 36%           | 4%               |
| 100000          | 0%                   | 28%               | 72%           | 0%               |
|                 |                      |                   |               |                  |
| best_ckpt       | 4%                   | 52%               | 72%           | 16%              |
| **anybimanual** | **8%**               | **84%**           | **8%**        | **4%**           |

| ckpt        | take_tray_out_of_oven | handover_item_easy |
| ----------- | --------------------- | ------------------ |
| 0           | 0%                    | 0%                 |
| 10000       | 0%                    | 0%                 |
| 20000       | 0%                    | 32%                |
| 30000       | 8%                    | 8%                 |
| 40000       | 0%                    | 4%                 |
| 50000       | 0%                    | 12%                |
| 60000       | 0%                    | 28%                |
| 70000       | 0%                    | 12%                |
| 80000       | 8%                    | 4%                 |
| 90000       | 0%                    | 8%                 |
| 100000      | 0%                    | 4%                 |
|             |                       |                    |
| best_ckpt   | 8%                    | 32%                |
| anybimanual | 4%                    | 28%                |

# 3.19

​	阅读代码解决之前不能正常进行评估的问题。

​	完成了部分任务的评估，与论文一致，在不同ckpt下的评估25个episode的成功率，评估结果如下：

| ckpt   | dual_push_buttons |
| ------ | ----------------- |
| 0      | 20%               |
| 10000  | 0%                |
| 20000  | 8%                |
| 30000  | 24%               |
| 40000  | 16%               |
| 50000  | 40%               |
| 60000  | 48%               |
| 70000  | 52%               |
| 80000  | 32%               |
| 90000  | 24%               |
| 100000 | 28%               |

论文的实验结果：

![18ed81eccfee95ecff5dc15a3ab676d](https://raw.githubusercontent.com/oxygeng710/picture/main/18ed81eccfee95ecff5dc15a3ab676d.png)

​	根据peract2所提供的测试任务，所完成的评估任务dual_push_buttons对应了anybimanual中的press buttons，即便是表现最好的第70000的ckpt成功率也有较大差距，对于这个任务论文中成功率为84%，评估结果最好的为52%。

### 补充：

​	peract2论文的主要贡献提供了双臂控制的测试任务，peract2完成的是single-task agent，而anybimanual的目标是完成一个multi-task agent。所以peract2在anybimanual论文中显示的成功率很低，之前完成了peract2的multi-task agent训练但没有注意到这个问题，进行eval时成功率极低，基本上无法成功。

​	

# 3.7

## peract2：

### 框架：

![image-20250307145239537](https://raw.githubusercontent.com/oxygeng710/picture/main/image-20250307145239537.png)

#### 处理步骤

1. **体素化**：将多个RGB-D图像融合成一个3D体素网格，表示机器人工作空间的体积信息，每个体素对应空间中的一个小立方体
2. **特征学习**：使用Perceiver IO transformer对体素网格和语言指令进行编码，捕捉其空间和语义特征
3. **动作预测**：基于特征生成每只机械臂的动作指令，包括：
   - **6自由度（DoF）末端执行器位置**：在3D空间中的位置和方向
   - **抓取器状态**：打开或关闭
   - **运动规划器碰撞标志**：是否需要避障
4. **动作执行**：将预测动作发送给机器人控制器，驱动机械臂运动

### 复现结果：

![1741341796456](https://raw.githubusercontent.com/oxygeng710/picture/main/1741341796456.png)

​	论文中评估结果的方式仅有成功率一项，这里可能需要运行eval.py才能够得到成功率，运行这个时也是有与之前同样的问题

qt.qpa.plugin: Could not load the Qt platform plugin "xcb" in "/share/home/HCI/yugeng/user/CoppeliaSim_Player_V4_1_0_Ubuntu16_04" even though it was found.
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.

Available platform plugins are: eglfs, linuxfb, minimal, minimalegl, offscreen, vnc, xcb.

根据[CoppeliaSim的安装与xrdp远程连接](https://docs.ruotao.tech/#/CoppeliaSim_xrdp)中的内容，需要连接桌面解决，但实际连接桌面后仍然报此项错误。网上有一些解决方法，如检查/share/home/HCI/yugeng/user/CoppeliaSim_Player_V4_1_0_Ubuntu16_04下的platform文件夹中的文件是否完整，查询后并未找到网上残缺部分的内容。此次报错与之前的无法display问题不同，无法直接通过阅读报错的调用去修改代码解决，还在继续查阅相关方法的github中的issue栏寻找解决方案。

## Anybimanual:

### 框架：

![image-20250307161119801](https://raw.githubusercontent.com/oxygeng710/picture/main/image-20250307161119801.png)

1. **语言分支**：
   - 使用预训练的文本编码器解析双臂操作指令，将其转换为具有高层次语义的语言嵌入。
   - 技能管理器通过自适应地协调原始技能，为不同的手臂增强语言嵌入。这个过程通过线性组合和补偿来实现，将组合的技能原语与初始的双臂语言嵌入进行拼接，将其传递给相应的单臂策略。
2. **视觉分支**：
   - RGB-D输入被体素化为体素空间作为观察。
   - 使用3D稀疏体素编码器对体素观察进行编码，以获得信息丰富的体积表示。
   - 视觉对齐器生成空间软掩码，以对齐单臂策略模型的视觉表示，使其与预训练阶段的表示匹配。这有助于减少单臂和双臂系统之间的观察差异，提高策略的可转移性。

### 复现：

之前说训练这个需要peract2的checkpoint，原因是：

![2dbaec00edd494c8597fb37e368c39b](https://raw.githubusercontent.com/oxygeng710/picture/main/2dbaec00edd494c8597fb37e368c39b.png)

还有anybimanual的readme中所写的训练指令：bash scripts/train.sh BIMANUAL_PERACT 0,1 12345 ${exp_name}，以及peract2的github网址中的repository名为peract_bimanual，让我误以为这里需要的checkpoint是peract2的，后面利用peract2的checkpoint训练加载模型时出现mismatch报错，然后重新阅读论文和readme之后确认这里使用的是peract的checkpoint。

目前正在尝试复现论文中提到的最好的方法组合PerAct + AnyBimanual。训练过程与peract2类似，但是多了一个processing files的过程，根据论文内容，可能是文中提到的自动技能发现方法，以无监督的方式从离线双臂操作数据集中自动识别和提取出基本的技能原语。

![image-20250307172100095](https://raw.githubusercontent.com/oxygeng710/picture/main/image-20250307172100095.png)
