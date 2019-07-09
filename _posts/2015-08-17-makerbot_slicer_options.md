---
title: MakerBot Slicer 配置文件
tags: 技术 3D打印 MakerBot
---  

来自[MakerBot Slicer Options](https://www.makerbot.com/support/new/04_Desktop/Knowledge_Base/Using_Custom_Slicing_Profiles)的中文翻译。
在使用MakerBot的时候，遇到一些需要调整的参数。Makerbot的参数调整，是通过自定义Profile文件的方式进行的。
<!--more-->
*如果发现翻译错误，恳请告知 topcaver[AT]gmail.com*

###Layers (分层)
层设置定义在打印过程中，挤出耗材的丝的高度和宽度。包括一组配置项：

######**"layerHeight"**: 层高（毫米）
定义每一层的高度。


######**"layerHeightMinimum" / "layerHeightMaximum"** 最小层高/最大层高（毫米）
在你配置品质（Quality）项目时，这个配置决定了使用哪个基础配置模版（basic profiles）。在使用自定义配置切片时，这些配置不起作用。

######**"layerWidthMaximum" / "layerWidthMinimum"** 最大/最小 层宽度（毫米）
在必要时，覆写"layerWidthRatio"配置，限制挤出的宽度。默认情况下，它们被设置为相同的值，确保无论层高如何设置，层宽都能保持一致。

######**"layerWidthRatio"** 层宽比（小数，比例）
层宽除以层高的比例。如果计算出的宽度，介于"layerWidthMinimum"和"layerWidthMaximum"设置的值之间，那么压挤出的宽度等于层高乘以层宽率。

######**"computeVolumeLike2_1_0"** (True，False)
这个选项仅出现于从早期版本升级而来的配置文件中。它允许你不需要修改参数，也可以在当前Makerbot中使用旧的配置文件。

###Positioning （定位）
定位设置给予切片器关于3D打印机各组件的初始位置。包括：

######**"bedZOffset"** Z轴偏移（毫米）
Makerbot的Z轴偏移，是存储在主板上的。bedZoffset允许更改这一设置。

######**"startX" / "startY" / "startZ"** 起始X/Y/Z（毫米）
这三个设置确定了打印开始时的开始位置。如果你使用自定义的start.gcode文件移动喷头到了不同的位置，必须要修改这项设置以匹配在start.gcode里面的位置信息。

######**"defaultExtruder"** 默认喷头（整数［0，1］）
当你在MakerBot Desktop中使用MakerBot Slicer的时候，你可能要为不同的物品分配各自不同的喷头。当你手动执行MakerBot Slicer时，这个设置将决定使用哪个喷头建造你的模型。在双喷头的机器中，0代表右侧的喷头，1代表左侧的喷头。单喷头的机器，0代表这个喷头。

###Travel Movement （空载快速移动）
空载移动，描述在喷头不挤出塑料的时候的移动速度。这种移动速度比打印时的挤料移动要快得多。
######**"rapidMoveFeedRateXY"** XY轴快速移动速率（毫米/秒）
######**"rapidMoveFeedRateZ"** Z轴快速移动速率（毫米/秒）

###Speed （速度）
设置3D打印喷头的各项移动速率，打印过程中的减速和加速。

######**"minLayerDuration"** 最小层耗时（秒）
设置每层最短时间。如果某层完成时间小于这个设置值，喷头将会降低速度，直到打印一层完成时的时间等于这个设置。这可以让很小的层充分冷却，避免之后一层叠加在还是融化状态的塑料上。

######**"minSpeedMultiplier"** 最小速度乘数（小数［0，1］）
此设置乘以挤出设置中的基础速率，等于最小的打印速度。在最小层耗时的限制下，喷头的移动速度也不会低于这个速度。Dynamic Speed设置的降速不会被这个设置影响。

######**"doDynamicSpeed" 使用动态速度**（True，False）
动态速度可以在急转曲线上通过降速获取更好的效果。当doDynamicSpeed设置为true时，切片程序会通过下面的参数降低速率。

######**"dynamicSpeedCurvatureThreshold"** 动态速度曲率门限值（度/毫米）
"DynamicSpeedCurvatureThreshold"结合"dynamicSpeedDetectionWindow"设置以确定在何种曲率条件下触发动态降速。如果在"dynamicSpeedDetectionWindow"中设置的距离内，平均变化角度大于"dynamicSpeedCurvatureThreshold"中设置的角度变化数，切片器降低喷头移动速度。

######**"dynamicSpeedDetectionWindow"** 动态速率监测窗口（毫米）
略，见上。

######**"dynamicSpeedSlowdownRatio"** 动态速度减速率（小数，比率）
动态速率减速时相对当前基础速度的比率。

######**"doDynamicSpeedGradually"** 使用动态速率平滑变速（True, False）
当触发动态速度降速时，如果此项设置为True，降速和提速过程将缓慢进行。如果设置为False，这一变化将由固件的加速度控制。

######**"dynamicSpeedTransitionWindow"** 动态速度过渡窗口（毫米）
doDynamicSpeedGradually设置为“True”时，此项配置决定速度平滑需要的长度距离。

######**"dynamicSpeedTransitionShape"** 动态速度过渡特征（小数［0，1］）
此项设置控制动态减速的过渡阶段。设置越靠近0，加速曲线更类似一个梯形；设置越靠近1，加速曲线越接近于S曲线。

######**"doDynamicSpeedOutermostShell"** 在最外端面使用动态速度（True，False）

######**"doDynamicSpeedInteriorShells"** 在内部面使用动态速度（True，False）

######**"doSplitLongMoves"** 分离长距离移动（True，False）
如果在动态速度检测窗口中，有一段既包含大曲率的曲线也包括长直线的情绪，平均变化曲率可能不到减速门限，不触发动态减速。此项配置就为了适应此种情况，将长距离的移动分解成不同的移动路径。此项设置为True时，每一个长移动将按照下面的"splitMinimumDistance"进行分割。如果分割后的移动距离也不会触发动态减速，那么这些线段将被重新组合。

提示：长路径分割成短路径，将使导出的打印文件变大。

######**"splitMinimumDistance"** 切分最短距离（毫米）
######**"doSplitLongMoves"** 设置为True时，长距离移动会被分割成指定的长度。

###Temperature （温度）
设置基准的喷头、平台和机器内空间温度
######**"extruderTemp0" / "extruderTemp1"** 喷头0/1的温度 (摄氏度)
分别设置两个喷头的温度。如果是单喷头的机器，喷头就是喷头0。如果是双喷头机器，喷头0是右侧喷头，喷头1是左侧喷头。如果使用单喷头打印，没有用的那个设置将会被忽略。温度的单位是摄氏度。

######**"platformTemp"**  平台温度（摄氏度）
设置热床平台的温度。如果3D打印机没有可加热的平台，这个设置将被忽略。

######**"chamberTemp"** 打印腔内加热（摄氏度）
设置打印机腔内空间温度。如果没有加热腔，此设置将被忽略。

###Shells （壳面）
壳面是每一层的外轮廓。每个附加壳面会略微嵌入前一个壳面。
######**"numberOfShells"** 壳面的数量 （整数，［1，无穷］）
每一层打印的壳面数量。如果模型不足以容纳足够的壳面数，切片程序将会尽可能打印最多的壳面数量。

######**"infillShellSpacingMultiplier"** 填充壳面空隙乘数（小数，［0，1］）
指定内壳面和内填充之间的重叠部分。0.0表示内填充将完与内壳面完全重叠，1.0表示内填充线仅仅接触到内壳面。

######**"insetDistanceMultiplier"** 插入距离乘数（小数，层宽的乘数）
定义相邻的两个壳面的距离。数值时挤出料宽度的乘数。设置小于1，会产生潜入，内壳面，重叠；大于1，将会产生缝隙。MakerBot建议有一些略微的重叠。

######**"shellsLeakyConnections"** 壳面漏空连接 （True，False）
如果设置为False，在壳面之间移动时，喷头将继续挤出塑料；如果设置为“true”，喷头将不再挤出并回缩塑料。此项可以避免打印出多余的塑料。

###Roofs and Floors （顶面和底面）
顶部和底部设置，会影响实体层的顶部和底部。

######**"roofThickness"** 顶面厚度（毫米）
顶层厚度设置会影响你的模型的实体顶层的厚度。这项设置生效时，顶层厚度和层高和层数无关。如果"roofLayerCount"设置生效，此项设置将失效。

######**"roofLayerCount"** 顶面层数（整数，［0，无穷］）
生效时将覆盖"roofThickness"的设置，使用指定的层数做顶。如果不想使用这个设置，修改设置的名称为："roofLayerCount_disabled"

######**"roofAnchorMargin"** 顶层锚边距（毫米）
如果当前打印的层，大于下一个要打印的层，切片程序将会加入实体填充的内窄边框来支持下一层的打印，避免出现缝隙。此项设置决定了这个窄边框的宽度。也可以设置为0关闭这一特性。

######**"floorThickness"** 底面厚度 （毫米）
设置模型实体底部的高度，这项设置和层高和层数无关。如果"floorLayerCount"生效，本项配置将失效。

######**"floorLayerCount"** 底面层数 （整数，［0，无穷］）
生效时将覆盖"floorThickness"的设置，使用指定的层数做底面。如果不想使用这个设置，修改设置的名称为："floorLayerCount_disabled"

###Resolution （分辨率）
分辨率设置定义使用何种精度将3D模型转化为喷头打印路径。

######**"coarseness"** 粗糙度（小数，［0，0.1］）
在创建打印喷头的路径前，用来简化每层轮廓。切片程序将相邻几段几乎共线的线段连接在一起。设置为0，将没有线段被简化合并，更多的细节将被打印出来。值越高，细节表现将越少。设置到0.1，所有的线段都将被连接，不推荐这样设置。

###Infill （填充）
填充设置将影响打印物品的内部结构。

######**"sparseInfillPattern"** 稀疏填充模式（字符串 [linear, hexagonal, moroccanstar, catfill, sharkfill]）
填充的解构模式，包括：线性、六角形、五角星（moroccanstar），猫填充，[鲨鱼填充](http://www.makerbot.com/blog/2013/08/05/makerware-2-2-2-sharkfill-for-shark-week)。

######**"infillDensity"** 填充密度 （小数，［0，1］）
设置为0到1之间的一个小数。0会打印一个空心物品。1会打印一个实心物品。

######**"infillOrientationOffset"** 填充方向偏移 （角度）
仅在填充模式设置为“线性（linear）”时生效。第一层填充将旋转一定角度。

######**"infillOrientationInterval"** 填充方向间隔 （角度）
仅在填充模式设置为“线性（linear）”时生效。每一层填充将依次旋转一定角度。

######**"infillOrientationRange"** 填充方向范围（角度）
仅在填充模式设置为“线性（linear）”时生效。当超过此范围后，将被重置为初始角度。

######**"gridSpacingMultiplier"** 网格空隙乘数 （小数，［0，1］）
相邻的底面和顶面或者实体填充需要稍有重叠，形成一个连续的表面。切片程序使用挤出宽度和本设置乘数确定重叠的量。这个结果会让挤出料之间的空隙比实际的小。

######**"solidFillOrientationOffset"** 实体填充方向便宜（角度）
第一层实体表面旋转特定的角度。

######**"solidFillOrientationInterval"** 实体填充间隔（角度）
每个实体层依次旋转特定的角度。

######**"solidFillOrientationRange"** 实体填充方向范围（角度）
超出范围之后，实体填充角度将被重置到"solidFillOrientationOffset"

######**"adjacentFillLeakyConnections"** 相邻填充漏空连接（true，false）
当设置为False时，喷头将继续挤出塑料填充相邻两个填充线。若为False，喷头在移动时将不再挤出并回缩塑料。哪些填充线时相邻的，取决于“adjacentFillLeakyDistanceRatio”设置。

###Backlash Compensation （反向间隙补偿）
在3D打印机的坐标轴上有一些空隙时。当喷头改变方向时，一小部分的移动将会这些空隙抵消掉。导致实际移动距离，少于预期。反向间隙将影响精度。这些设置将补偿反向间隙。

######**"doBacklashCompensation"** 使用反向间隙补偿（True，False）
设置为“true”将会补偿反向间隙。设置为“false”将不会补偿。这个特性是一个实验性的，可能会对导致轻微的打印失真。

######**"backlashFeedback"** 反向间隙反馈（小数，［0，1］）
MakerBot切片程序逐步反馈补偿一减少打印失真。他将根据"splitMinimumDistance" 补偿一个特定的反馈距离。每一段由"splitMinimumDistance"设置到长度的反馈补偿的距离，是本项配置的百分比的倒数。例如，如果设置为0.9，"splitMinimumDistance"是0.4mm，那么切片程序将补偿10%反馈误差在第一个0.4mm改变方向时。他将在下一个0.4mm中补偿10%的剩余误差，直到误差距离小于"backlashEpsilon"中的设置。如果"backlashFeedback"设置为0，所有反馈误差将被立即补偿。

######**"backlashEpsilon"** 反向补偿系数（毫米）
当你使用"backlashFeedback"设置时，切片程序将补偿逐渐减少的误差。剩余的误差会越来越小，但是永远不会被补偿完毕。本项配置为了解决这个最后补偿的问题。当剩余误差小于这里设置的距离，切片器将停止补偿剩余误差。

######**"backlashX" / "backlashY"** X/Y轴反向间隙（毫米）
此设置告诉切片程序，在X/Y轴应该补偿多少反向间隙误差。可以这样确定反向间隙的误差：打印20mm精确的盒子（在**File > Examples** 附带的实例中有），然后测量打印出的盒子的宽和高。如果长度（Y）或宽度（X）小于20mm，用20减去实际值，再除以2。这个值就是反向间隙。

###Leaky Connections （漏空连接）
此选项提供了更多的选项，控制喷头在短距离内部空载移动时的行为。

######**"shellsLeakyConnections"** 壳面漏空连接 （True，False）
设置为“false”时，喷头在两个壳面见移动时将会持续喷料。设置为“true”时，喷头将不再挤出并回缩塑料。这可以避免构造出多余的塑料。

######**"adjacentFillLeakyConnections"** 相邻填充漏空连接（True，False）
“false”时，在相邻填充线之间持续喷料。“true”时，保持不再挤出并回缩塑料。哪些填充线是相邻的，取决于“adjacentFillLeakyDistanceRatio”的设置。

######**"supportLeakyConnections"** 支撑漏空连接（True，False）
“false”时，在两个支撑段之间将持续喷料。“true”时不喷料，但是也不回缩。这会导致细小的链接，既可以加固支撑，也便于拆除。

######**"adjacentFillLeakyDistanceRatio"** 相邻填充漏空连接距离乘数 （比例，［1.0，2.0］）
在两个填充线之间的距离小于此值和挤出宽度的乘积时，他们被认为是相邻的。

######**"leakyConnectionsAdjacentDistance"** 漏空连接相邻距离（原料的毫米）
空载移动前后的挤出路径的长度，至少要到设置的距离，切片程序才会将它转化为一个“漏空”连接。这适用于壳面、支撑和填充。


###Bridging （桥接）
桥接作用于两边有支撑而中间没有支撑的结构上。

######**"doBridging"** 使用桥接 （True，False）
“true”时，切片程序将确保挤出材料在稳定的锚接区域。“false”时，下面的设置将失效。

######**"bridgeAnchorMinimumLength" / "bridgeAnchorWidth"** 桥接锚点最小长度/桥接锚点宽度（毫米）
桥接锚点设置确定哪个线段可以被用作稳定的锚点区域。如果锚点区域太窄或太薄，他不能为桥结构提供足够的基础支撑。如果区域比“bridgeAnchorMinimumLength”设置的更窄，或者比"bridgeAnchorWidth"更薄，都不会被当作锚点区域。

######**"bridgeMaximumLength"** 桥接最大长度（毫米）
此设置用来定义桥。任何类似桥结构的长度大于这个设置，切片程序都不会当作桥。太长的无支持桥机构都不会打印的太好，无论桥的这些设置是否生效。

######**"bridgeSpacingMultiplier"** 桥接空隙乘数（乘数）
桥结构的第一层回比其他层更窄一些。因为他们不能被压入下面的层中。此设置来提供机会打印紧密的连线，所以他们会重叠。切片程序通过此设置乘以挤出宽度，并将该值作为预期的宽度。结果是，切片程序挤出的线间距会比实际的小一些。此项是试验性和可能无意义的。如果要让桥第一层正常的间距，请设置为1。

###Exponential Deceleration （指数减速）
允许在移动终端回缩之前，使用缓速挤料。
The exponential deceleration settings allow you to use the oozing plastic at the end of a move before retraction. This group of settings includes:
######**"doExponentialDeceleration"** 使用指数减速 （True，false）
融化态的塑料是液体，所以当你的喷头停止挤料时，塑料还是会继续渗出喷头。指数减速郧西你使用这些渗漏的塑料。首先，它将稍早一点结束寂寥，一遍使用渗漏的料完成后面的打印。然后他缓慢减速喷头，以保证挤出的宽度一致。

######**"exponentialDecelerationRatio"** 指数减速率 （比率，［0.0，1.0］）
在指数减速期间，喷头速度回从初始速度减到 初始速度和此设置的乘积。

######**"exponentialDecelerationSegmentCount"** 指数减速段数 （正整数）
设置几段减速。大的数字会让减速更平滑，但是会增大打印文件的体积。

######**"preOozeFeedstockDistance"** 预滴料距离（耗材毫米）
喷头在指数减速前，多挤出一些料。移动使用少于这个设置时，喷头将正常停止。如果用料大于这个数值，但是少于oozeFeedstockDistance，允许渗出的料将减少。

######**"oozeFeedstockDistance"** 渗出料长度（耗材毫米）
从喷头停止挤料时，到打印行尾时的塑料的数量。

###Spurs （刺？）
影响切片程序如何为非常细的线段创建一个移动路径。刺时一个单层部分，外部轮廓非常接近以至于退化成一根单线的打印结构。

######**"doExternalSpurs"** 外部刺（True，False）
设置为“True”时，以下设置被用于创建单层结果。如果设置为“false”，那些需要仅有一丝宽度的结构，将根本不被打印。

######**"doInternalSpurs"** 内部刺（True，False）
内部刺出现在你的模型的的轮廓内部。当内部打印缩小到一个点大小的时候，在壳面内侧往往会出现内部刺。如果设置为“true”，单丝料宽度的缝隙，将被填充。

提示：这是一个试验特性。

######**"maxSpurWidth"** / "minSpurWidth" 最大/最小刺宽度 （毫米）
宽于最大宽度的刺和小于最小宽度的刺，都不会被处理。更宽的结构将能打印多条丝宽的结构，更窄的刺将不再被打印。

######**"minSpurLength"** 最小刺长度（毫米）
小于这个长度的刺，最后都会被从打印路径中去除。

######**"spurOverlap"** 刺重叠（毫米）
此设置允许几乎接触的刺结合在一起。

###Rafts （底座）
这些设置影响打印到你模型下面的底座。

######"doRaft" 打印底座 （True,False）
是否打印底座。设置为“false”时，底座设置将被忽略。

######"doMixedRaft" 打印混合底座 （True，False）
双头打印机如何打印底座。如果设置为“true”，打印底座和打印模型将使用相同的喷头，这意味着不同部分模型下面的底座，可由不同的喷头打印。设置为“false”时，底座将由"defaultRaftMaterial"决定使用哪个喷头打印。设置为“true”相当于在底座下拉设置菜单中选择“Color-Matched”（匹配颜色）。

######"defaultRaftMaterial" 默认底座材料 （整数，［0，1］）
在"doMixedRaft"设置为False时，0代表右侧喷头，1代表左侧喷头。单喷头打印机，单喷头为0

######"raftBaseAngle" 底座基层角度（角度）
底座基层的角度。默认是45度。这样更细的挤出材料不会掉入底座缝隙里面。

######"raftBaseDensity" 底座基层密度（小数，［0，1］）
每层底座为一组Z字型的线。1.0密度将创建一个实体的底座。更小的密度，会得到更宽的线间距。

######"raftBaseLayers" 底座基层层数 （整数，［0，无穷］）
底座打印多少层

######"raftBaseRunGapRatio" 底座基层运行间隙比（比率）
此设置确定底座Z字折线行之间间隙的大小。底座层挤出的宽度乘以设置值。太大的间隙可能会让部分界面层的材料漏下到底座层。不存在的间隙可能会导致底座翘起弯曲。

######"raftBaseRunLength" 底座基层运行长度（毫米）
底座由一系列Z型折现构成。这项设置决定了每行的宽度。

######"raftBaseThickness" 底座基层厚度（毫米）
底座基层的厚度。

######"raftBaseWidth" 底座基层宽度（毫米）
底座基层的挤出宽度。

######"raftInterfaceAngle" 底座接口层角度 （角度）
######"raftInterfaceDensity" 底座接口层密度（小数，［0，1］）
######"raftInterfaceLayers" 底座接口层层数（整数，［0，正无穷］）
######"raftInterfaceThickness"  底座接口层厚度（毫米）
######"raftInterfaceWidth" 底座接口层宽度（毫米）

######"raftAligned" 底座对齐（True，False）
“true”强制所有底座接口层相同方向打印。“false”每一层底座接口层将旋转90度。

######"raftModelSpacing" 底座模型间距（毫米）
此设置定义了底座最上层和模型最下层之间的垂直距离。正值，将留出一个小的空间，更容易的拆除底座，但模型第一层补胎平滑。负值，将模型打印到底座当中，底座和模型更加牢固。这些设置仅对模型第一层有效。后续的层，将打印在相对底座的相同高度。

######"raftOutset" 底座外延（毫米）
底座延伸出模型多远的距离。

######"raftSurfaceAngle" 底座表面角度（角度）
######"raftSurfaceLayers" 底座表面层数（整数［0，1］）
######"raftSurfaceThickness" 底座表面厚度 （毫米）

###Anchor （锚点）
锚点设置影响那些打印在每次开始时锚点。
The anchor settings affect the anchor that is printed on your build plate at the beginning of each print. This group of settings includes:
######"anchorWidth" 锚点宽度 （毫米）
影响锚点和打印物或平台连接点的挤出宽度。

######"anchorExtrusionAmount" 锚点挤出量（耗材毫米）
确定在起点，挤出多少塑料形成一团锚点

######"anchorExtrusionSpeed" 锚点基础速度 （耗材毫米/秒）
确定在锚点起点，塑料进入喷头的速度。


###Support （支撑）
影响支撑结构的打印。
######"doSupport" 使用支撑（True，False）
支撑结构的开关。当设置为false时，下面所有支撑相关的设置将不起作用。

######"doSupportUnderBridges" 桥接结构下使用支撑 （True，False）
如果设置为False，任何符合"bridgeMaximumLength"设置的桥下面将不再生成支撑。如果设置为“True”，支撑结构都将生成，无论是否桥接。

######"doMixedSupport" 混合支撑（True，False）
取决于你的双头3D打印机如何打印支撑结构。设置为True，支撑将会使用相同的喷头打印。这意味不同部分的支撑结构可能是不同的喷头打印的。如果设置为false，喷头设置中"defaultSupportMaterial"将会用于打印所有支撑材料。

######"defaultSupportMaterial" 默认支撑材料（整数［0，1］）

######"supportAligned" 支撑对齐（True，False）
True，强制支撑相同方向。False，每层挤出的支撑旋转90度。

######"supportLeakyConnections" 支撑漏空连接
false连续挤出塑料，true允许空隙。

######"supportDensity" 支撑密度 （小数，［0，1］）
每层支撑表现为Z字形，1.0的支撑密度将会实体支撑结构，

######"supportExtraDistance" 支撑扩展距离

######"supportAngle" 支撑角度

######"supportModelSpacing" 支撑模型空隙（毫米）
模型轮廓与支撑之间的距离。支撑越靠近模型，越难拆除。支撑远离模型，将提供更少的支撑。

######"supportExcessive" 支撑过载？

###Purge Wall （清除壁）
影响一个无关的外部结构，用来减少双喷头打印时的材料混合。
The purge wall settings affect an extraneous external structure that can be used to reduce mixing of materials during a dual extrusion print. This group of settings includes:
"doPurgeWall"
"purgeWallModelOffset"
"purgeWallXLength"
"purgeWallWidth"
"purgeWallBaseWidth"
"purgeWallBasePatternLength"
"purgeWallBasePatternWidth"
"purgeWallBaseFilamentWidth"
"purgeWallPatternWidth"
"purgeWallBucketSide"

###Fan Controls（风扇控制）
这些设置控制是否将风扇控制命令插入到生成的喷头打印路径上。

######"doFanCommand" 使用风扇命令 （True，False）
“true”插入风扇命令，在打印过程中控制一个冷却风扇。“false”将忽略下面的设置。

######"fanLayer" 风扇层（整数）
指定那一层插入开启活动冷却风扇的命令

######"weightedFanCommand" 风扇速度（整数，［1，100］）
允许设置喷头风扇速度，MakerBot 3D打印机不使用这项设置。

######"doFanModulation" 使用风扇调速器 （True，Flase）
风扇调速器，允许在打印过程中，关闭或降低风速。“true”将在低俗打印和空载移动时关闭风扇；“false”将在打印过程中，使用固定风速。

######"fanModulationWindow" 风扇调速窗口（秒）
在"doFanModulation"设置为true时，每次回缩材料时，将关闭风扇。风扇再次开启的实际将由调速窗口和调速门限共同决定。
每一次回缩之后，切片器将会分析有"fanModulationWindow"定义的时间段内预期移动。当打印移动和空载移动的比例高于"fanModulationThreshold"设定的门限时，风扇将重新开启。

######"fanModulationThreshold" 风扇调速门限（小数，比例）
略，见上。

###Toolpath （喷头路径）
喷头路径设置和描述喷头路径使用的代码有关
######"doPrintProgress" 显示打印进度（True，False）
“true”打印进度将显示在Makerbot的LCD屏幕上。

######"startGcode" / "endGcode" 起始/结束Gcode
MakderBot 3D打印机包括Replicator 2X ，需要一段Gcode文件附加在开始和结束位置，来控制喷头加热和冷却、构造平台和喷头的复位。默认情况下，这将由MakerBot Desktop创建。如果你使用自定义的起始结束Gcode文件，使用这个设置，来指定你需要使用的文件路径。如果需要关闭这个配置，可删除文件路径，或者将名字改为"startGcode_disabled"/"endGcode_disabled"。

你也可以使用标准的起始、结束的Gcode文件。使用Finder（Mac）或资源管理器（windows）打开Things > profiles > [Custom Profile Name]。这里面的 "startGcode" 和 "endGcode" 是默认生成的文件。

第五代MakerBot打印机，不再使用Gcode。这里设置的Gcode将被第五代打印机忽略。

######"commentClose" / "commentOpen" 注释开始/注释结束(字符串)
不同的打印机在Gcode中使用不同的字符来表示注释信息。MakerBot切片程序默认用MakerBot打印机支持的注释字符。

###Extruder Profiles （喷头配置）
此处分为两段配置，允许分别设置两个喷头。如果你的3D打印机有两个喷头，第一段设置将作用于右侧的喷头，第二段设置将作用于左侧的喷头。如果你的打印机只有一个喷头，那么将使用第一个设置，第二个设置将被忽略。
此项配置中的某些组件允许你为不同的喷头设置不同的配置，比如填充和内壳面。

######"extruderProfiles"
######"feedDiameter" 耗材直径（毫米）
根据耗材设置，如果这个值太小，喷头将会挤出过多的塑料，如果这个值过大，喷头将挤出很少的塑料。

######"feedstockMultiplier" 材料乘数（乘数）
由于种种原因，例如材料密度，膨胀，材料进入喷头和挤压出的料的量并不相同。这个数字用来处理这种差异。

######"nozzleDiameter" 喷嘴直径（毫米）
根据喷头的实际值设置

######"retractRate"/"retractRate2" 回缩塑料（毫米/秒）
设置回缩阶段拉回的材料的速度。

######"retractDistance"/"retractDistance2" 回缩距离（毫米）
喷头在空载移动前，将回缩制定的距离，避免拉丝出现。当你设置"retractDistance2"，回缩过程将分成两个阶段。在第一阶段，回缩"retractDistance"指定的距离，在第二阶段，回缩"retractDistance2"指定的距离。如果没有设置"retractDistance2"或者"retractRate2"，回缩将只有一个过程。

######"restartRate"/"restartRate2" 复位速率（毫米/秒）
设置回缩后，喷头复位塑料的速度。设置restartRate2，过程将分成两段。

######"restartExtraDistance" 复位附加距离（毫米）
允许在回缩后复位更多或更少的耗材长度。正数将会多出一些塑料，补偿在回缩时滴落的塑料，但是这可能会有少量疙瘩。负数会少复位一段塑料，完全消除接缝处的小疙瘩，但是这可能导致一些间隙，或者小细节打印失败。这项设置，是设置附加的距离，不是整体距离。

######"restartExtraRate" 复位附加速率（毫米/秒）
可以为复位距离设置不同的复位速率。

######"toolchangeRetractDistance" 喷头切换回缩距离（毫米）
在双头打印机中，当一个喷头切换至不活动时的回缩距离。

######"toolchangeRetractRate" 喷头切换回缩速率（毫米/秒）
######"toolchangeRestartDistance" 喷头切换复位距离（毫米）
######"toolchangeRestartRate" 喷头切换复位速率（毫米/秒）

######"bridgesExtrusionProfile" 桥接挤出配置（字符串）

######"firstLayerExtrusionProfile" 首层挤出配置
######"firstLayerRaftExtrusionProfile" 首层底座配置
######"infillsExtrusionProfile" 填充挤出配置
######"insetsExtrusionProfile" 内壳挤出配置
######"outlinesExtrusionProfile" 外轮廓挤出配置
######"raftBaseExtrusionProfile" 底座挤出配置
######"feedrate" 打印速率
######"temperature" 温度