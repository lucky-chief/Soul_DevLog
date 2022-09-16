# 《魂》开发日志
## 游戏世界观简介
<br><br>
## 开发配置
### Addons
#### Ultimate Character Controller
[商店页面（付费）](https://assetstore.unity.com/packages/tools/game-toolkits/ultimate-character-controller-99962)<br>
**选用理由:** 角色控制系统，提供了第一、三人称的基本角色控制，包括人物的动画状态机，raddoll，IK，物理碰撞等等关键技术，还有其他可以选用的系统，比如：技能系统、背包系统，状态系统，能够帮助个人开发者节省掉大量的时间，花更多的时间来实现自己的战斗逻辑。
#### DebugPlus
[商店页面（免费）](https://assetstore.unity.com/packages/tools/integration/debugplus-144985)<br>
**选用理由:** 开发战斗逻辑时需要绘制一些辅助线，Gyzmo来帮助调试，而Unity内置的 Debug.drawline 系列使用起来并不方便。具体使用方法详见商店页面。
#### POLYGON Fantasy Kingdom - Low Poly 3D Art by Synty
[商店页面（付费）](https://assetstore.unity.com/?q=polygen%20fantasy%20kingdom&orderBy=1)<br>
**选用理由:** 游戏素材库。

## 2022/9/15
### 游戏开发场景
Blender 里简单拉个盒子<br>
<video src="https://user-images.githubusercontent.com/11385187/190580573-48de148e-2e42-4b4e-aad4-faa9784883d4.mp4" controls="controls"></video>

到 [Mixamo](https://www.mixamo.com/)下载一个自己喜欢的人物模型，导入到Unity中，其中 Rig 选择 Humanoid, 生成人形骨骼 Avatar.<br>
![模型导入设置](https://user-images.githubusercontent.com/11385187/190583035-45623580-c00e-4aa6-840c-0285e650d7c3.png)

_Note:如果是第一次从Mixamo导入模型，则 Avatar Definition 选择 Create From This Model, 之后如果再导入 Mixamo 的模型，就可以选 Copy From Other Avatar, 然后选择第一次生成的Avatar即可。之所以可以这样是因为Mixamo上的人形模型的骨架结构都是一样，而如果你从别的网站比如 AccuRig、或者自己制作的模型 导入的话，由于两个模型的骨架结构不一样，就需要重新创建Avatar。_

按照 Ultimate Character Controller 的 开发文档配置下工程。把角色放入到场景中能自由行走就基本ok了。<br>

<video src="https://user-images.githubusercontent.com/11385187/190589946-3dfd756a-5223-4444-914e-68b3c09b5939.webm" controls="controls"></video>

OK，这块就是俺的魂之地！我想传火。

## 2022/9/16
### 锁定敌人逻辑（参照黑暗之魂III）
**Goals**<br>
* 正确选取视野正前方离角色最近的怪
* 锁定目标时，相机LookAt目标，相机视野无法改变
* 锁定目标时，人物的移动逻辑（比如左右移动变为左右侧步，后退）
* 能够按方向（左右）切换目标

好的，有了目标，说干就干！！<br>
#### 正确选取视野正前方离角色最近的怪
先在场景里摆几个稻草人作为我的目标，比如这样
![image](https://user-images.githubusercontent.com/11385187/190598242-c1f93fca-0bc0-4449-b2b4-43790d31c692.png) <br>
4个怪，不能再多了。多了我打不过~~~~~~~<br>
先来个“狂草”<br>
![image](https://user-images.githubusercontent.com/11385187/190602920-ee0f84ee-8644-49fd-8431-c9e7faa7085f.png)

* 粉红框框是屏幕
* 绿色是个球
* A/B/C是目标
* me是玩家
* cam是相机， v向量是相机的local z方向
* h向量是相机的水平前方

**步骤**<br>
1. 以玩家为中心向四周发射一定半径的球形的物理射线，得到若干目标数组a
2. 在目标数组中选择离玩家最近的目标b
   - 如果b不在屏幕内，剔除该目标，回到2
   - 如果 Vector3.Dot(<b,玩家>, h) < 0,说明目标在视野后方，剔除该目标，回到2

嗯，这思路感觉对对的，就差那几行代码了~~~<br>

实现 UCC 的 Ability 类，把锁定敌人作为玩家的一个能力，然后七里哗啦就开始撸代码了。 <br>

*正在求取h向量*<br>
*在解决 使用 Physics.SphereCastNonAlloc，返回的结果 m_RaycastHits[] 每个元素的 distance 和 point 都是0 的问题*<br>
*查看相关文档*<br>
*正在测试*<br>
Ten Thousands Later ~~~~~~ <br>
一个名叫 LockEnemy 的能力就被做好了，看看效果<br>

**常规**
<video src='https://user-images.githubusercontent.com/11385187/190614549-dba48580-81e6-4dd7-9d94-1fb769753ed9.mp4' controls="controls"></video>

**屏幕外 + 视野后方**

<video src='https://user-images.githubusercontent.com/11385187/190618082-81e973a7-458b-4385-be98-9615f879a3d0.mp4' controls="controls"></video>


效果看起来也是对对的，传火成功。。。
