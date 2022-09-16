# 《魂》开发日志
## 游戏世界观简介
<br><br>
## 开发配置
### Addons
#### Ultimate Character Controller
[商店页面（付费）](https://assetstore.unity.com/packages/tools/game-toolkits/ultimate-character-controller-99962)<br>
**选用理由:** 角色控制系统，提供了第一、三人称的基本角色控制，包括人物的动画状态机，raddoll，IK，物理碰撞等等关键技术，还有其他可以选用的系统，比如：技能系统、背包系统，状态系统，能够帮助个人开发者节省掉大量的时间，花更多的时间来实现自己的战斗逻辑即可。
#### DebugPlus
[商店页面（免费）](https://assetstore.unity.com/packages/tools/integration/debugplus-144985)<br>
**选用理由:** 开发战斗逻辑时需要绘制一些辅助线，Gyzmo来帮助调试，而Unity内置的 Debug.drawline 系列使用起来并不方便。具体使用方法详见商店页面。
#### POLYGON Fantasy Kingdom - Low Poly 3D Art by Synty
[商店页面（付费）](https://assetstore.unity.com/?q=polygen%20fantasy%20kingdom&orderBy=1)<br>
**选用理由:** 游戏素材库。

## 2022/9/16
### 游戏开发场景
Blender 里简单拉个盒子<br>
<video src="https://user-images.githubusercontent.com/11385187/190580573-48de148e-2e42-4b4e-aad4-faa9784883d4.mp4"></video>

到 [Mixamo](https://www.mixamo.com/)下载一个自己喜欢的人物模型，导入到Unity中，其中 Rig 选择 Humanoid, 生成人形骨骼 Avatar.<br>
![模型导入设置](https://user-images.githubusercontent.com/11385187/190583035-45623580-c00e-4aa6-840c-0285e650d7c3.png)

_Note:如果是第一次从Mixamo导入模型，则 Avatar Definition 选择 Create From This Model, 之后如果再导入 Mixamo 的模型，就可以选 Copy From Other Avatar, 然后选择第一次生成的Avatar即可。之所以可以这样是因为Mixamo上的人形模型的骨架结构都是一样，而如果你从别的网站比如 AccuRig、或者自己制作的模型 导入的话，由于两个模型的骨架结构不一样，就需要重新创建Avatar。_

按照 Ultimate Character Controller 的 开发文档配置下工程。把角色放入到场景中能自由行走就基本ok了。<br>

<video src="https://user-images.githubusercontent.com/11385187/190589946-3dfd756a-5223-4444-914e-68b3c09b5939.webm"></video>

OK，这块就是俺的魂之地！我想传火。
