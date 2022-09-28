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
* [x] 正确选取视野正前方离角色最近的怪
* [x] 锁定目标时，相机LookAt目标，相机视野无法改变
* [x] 锁定目标时，人物的移动逻辑（比如左右移动变为左右侧步，后退）
* [ ] 能够按方向（左右）切换目标

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
Ten Thousand Years Later ~~~~~~ <br>
一个名叫 LockEnemy 的能力就被做好了，看看效果<br>

**常规**
<video src='https://user-images.githubusercontent.com/11385187/190614549-dba48580-81e6-4dd7-9d94-1fb769753ed9.mp4' controls="controls"></video>

**屏幕外 + 视野后方**

<video src='https://user-images.githubusercontent.com/11385187/190618082-81e973a7-458b-4385-be98-9615f879a3d0.mp4' controls="controls"></video>


效果看起来也是对对的，传火成功。。。<br>
给LockEnemy 一个可序列化的字段 m_MaxLockDistance, 表示最大可锁定距离<br>
![image](https://user-images.githubusercontent.com/11385187/190884290-4bec95ad-30b3-473f-969c-33ae6f5e636a.png)
参照 **黑魂III**的感觉， 设置8差不多。<br>

在 **黑魂III**里，还有一个小细节，就是如果没有锁定到目标，相机视野会校正到人物的正前方。我也要~~~~~<br>
看了下UCC的源码，发现ViewType类里有个虚方法 **Reset**
```
/// <summary>
/// Resets the ViewType's character rotation and springs.
/// </summary>
/// <param name="characterRotation">The rotation of the character.</param>
public virtual void Reset(Quaternion characterRotation) { }
```
尝试下，在我的 LockEnemy 里添加如下代码
```
//如果没有转中目标，重置视野到玩家正前方
if(m_LockedTarget == null) {
   m_CameraController.ActiveViewType.Reset(m_Transform.rotation);
}
```
看看效果
<video src='https://user-images.githubusercontent.com/11385187/190884503-5ac57125-8f10-4983-890a-4b57e4f1c1d5.mp4' controls='controls'></video>
效果看起来是对了，但是太硬了，一帧就转过来了。

## 2022/9/17
### 锁定目标时，相机LookAt目标，相机视野无法改变
今天是礼拜六啊，睡了个饱觉（睡到10:30），要不是北京顺义的防空警报一直响个不停我还能睡~~~~~~~<br>
** 开始简单的以为锁定目标时就是把相机的targe(follow + lookat)设置成目标，但发现其实不对。相机的运动逻辑应该是 follow target 仍然是玩家自己，而 look target 是目标。** <br>
看了UCC代码，有一个类叫 AimAssist, 就是瞄准助手，能够帮助做这个目标锁定功能，丰富一下我的 LockEnemy 代码。<br>
Ten Thousand Years Later ~~~~~~ <br>
<video src="https://user-images.githubusercontent.com/11385187/190845662-1081c12b-74fa-4061-9549-496564258141.mp4" controls="controls"></video>

在实现的过程中，发现这个 AimAssist 设置target之后，用户的look输入还是会影响到相机的视野，其实在 **黑魂3**里，锁定怪之后，相机是不受用户的输入影响的，即如果操作手柄右摇杆相机视野无变化。更改 PlayerInput.LookSensitivity = 0 也无济于事，现在我的处理方案是：
```
m_PlayerInput.enabled = false;
```
** 简单粗暴，还未发现什么副作用~~~~ ** <br>

另外还有一个体验上的问题，就是锁定的时候玩家头部，身体的IK需要单独设置一下，能够让玩家模型在战斗是的头部一直盯着目标，体验上看起来更真实。于是在 锁定敌人开始是做了如下处理：
```
 if (m_CharactorIK) {
   m_CharactorIK.LookAtHeadWeight = 1f;
   m_CharactorIK.LookAtBodyWeight = 0.1f;
 }
```
IK的target，UCC框架已经帮忙处理了，我只要设置下权重即可，被照顾的感觉真好。。<br>
头部IK拉满，身体稍稍给点，否则太夸张体验会不好。看看效果：
<video src="https://user-images.githubusercontent.com/11385187/190846854-ee9b5742-967a-423b-a755-d21bca4fe106.mp4" controls="controls"></video>

可以看到在高处锁定敌人时，他的头在朝下看。这就很有感觉。。。。。

好的，锁定敌人相机运动逻辑就这样先~~~~~

## 2022/9/18
### 锁定目标时，人物的移动逻辑（比如左右移动变为左右侧步，后退）
这个的话，UCC就有提供功能。就是切换一下人物的移动方式即可<br>
![image](https://user-images.githubusercontent.com/11385187/190882733-ee7b4997-4f8e-4806-a3aa-5d468658cee5.png)
<br>
现在在人物身上我挂了两种移动方式，Adventure 是普通的行走方式， Combat 就是战斗时的移动方式。在成功锁定目标后，意味着进入了战斗模式，只要把人物移动方式切换为 Combat 即可。<br>
那为了做到这点，我现在的做法是添加一个人物状态叫 Combat, 在UCC的 Ability 的 inspector 面板中有一个字段是 **State**， 即表示成功触发能力后需要切到的状态<br>
![image](https://user-images.githubusercontent.com/11385187/190882870-32114c9f-5929-4653-a708-037599a2ec7b.png)
![image](https://user-images.githubusercontent.com/11385187/190882879-14f99e46-071d-4a06-acd5-88802716894e.png)
<br>
在人物状态里有一个 preset的字段， 这个是用来切到这个状态是要改变哪些属性的值，对于我的情况就是要把人物的 movementtype 改为 combat <br>
![image](https://user-images.githubusercontent.com/11385187/190883059-c105bbbf-90f3-4688-bd50-9c30d77ee6f8.png)
<br>
好的，这样的话，人物的移动逻辑应该ok了。然后到相机，UCC里，相机其实也有不同的类型叫 ViewTypes <br>
![image](https://user-images.githubusercontent.com/11385187/190882969-fcb60443-a2c7-4a6b-be61-76a8a4f70cf0.png)
<br>
这个可以方便地控制不同情况下不同的视野，比如 钻管道时， 相机的FOV 可能需要很小，就可以添加一个类似状态，把这个状态下相机的FOV调成自己需要的值。剩下的就是当钻管道场景触发时，把相机viewtype设置成对应的值即可,非常方便<br>
回到我的情况就是在锁定目标时需要把viewtype设置成 **Third Person Combat**， 这个状态我是给相机的 position smoothing 字段设置了值，目的是人物在战斗是，相机是**软跟随**的。<br>
![image](https://user-images.githubusercontent.com/11385187/190883366-f2305ab4-5d7c-4e91-b18c-1ab8502f323b.png)
 <br>
 
代码中，需要在锁定目标的Ability开始后切换相机的viewtype <br>
```
var combatVT = m_CameraController.GetViewType<Combat>();
if (combatVT != null) {
   m_CameraController.SetViewType(combatVT.GetType(), true);
}
```
当然也别忘了在退出锁定目标时要设置回来
```
var adventureVT = m_CameraController.GetViewType<Adventure>();
if (adventureVT != null) {
   m_CameraController.SetViewType(adventureVT.GetType(), true);
}
```

<video src='https://user-images.githubusercontent.com/11385187/190883620-79df3d84-6566-47d2-acb5-b85826f1bfb0.mp4' controls='controls'></video>

## 2022/9/20
### 按方向（左右）锁定目标
昨天一直再看UCC的代码，示例，文档，想搞懂玩家是如何装备新的武器的机制。搞得都快放弃了:joy:，还好最后冷静想想，随意放弃不是**不死人**的传火精神，坚持了下来，也略微窥探了UCC的切换装备的做法，回归主线吧~~~~~~<br>
锁定目标切换在选取目标的基础上在添加下规则就好。
1. 玩家按下特定健触发切换目标，比如z健向左选择目标，x健向右选择
2. 排除自己
3. 利用向量叉乘来判断目标在当前目标的左边or右边

记录下LockEnemy Ability关键部分代码
```C#
private void SelectTarget(bool rightSide = true)
 {
     Transform _target = null;
     if (m_LockedTarget) {
         _target = m_LockedTarget;
     }

     Vector3 lookDir = m_CharacterLocomotion.LookSource.LookDirection(true); //相机的 local z 方向，一般不是水平的
     Vector3 right = Vector3.Cross(Vector3.up, lookDir); //得到 水平右
     Vector3 forword = Vector3.Cross(right, Vector3.up).normalized; //得到 水平前

     int hitCount = Physics.SphereCastNonAlloc(m_Transform.position, m_MaxLockDistance, forword, m_RaycastHits, m_MaxLockDistance, m_CharacterLayerManager.EnemyLayers);

     //使用 Physics.SphereCastNonAlloc，返回的结果 m_RaycastHits[] 每个元素的 distance 和 point 都是0。
     //不知道是否是Unity的bug。官方该API的文档也未见说明
     //所以在此遍历，重新给 distance 和 point 赋值。
     for (int i = 0; i < hitCount; i++) {
         m_RaycastHits[i].distance = Vector3.Distance(m_Transform.position, m_RaycastHits[i].collider.bounds.center);
         m_RaycastHits[i].point = m_RaycastHits[i].transform.position;
     }

     if (hitCount > 0) {
         for (int i = 0; i < hitCount; ++i) {
             var closestRaycastHit = QuickSelect.SmallestK(m_RaycastHits, hitCount, i, m_RaycastHitComparer);
             var closestRaycastHitTransform = closestRaycastHit.transform;

             //如果是当前已锁定的目标，则不在锁定范围
             if(_target && _target == closestRaycastHitTransform) {
                 continue;
             }

             //如果在屏幕外，则不在锁定范围
             var viewPortPoint = m_Camera.WorldToViewportPoint(closestRaycastHit.point);
             if (viewPortPoint.x < 0f || viewPortPoint.x > 1f || viewPortPoint.y < 0f || viewPortPoint.y > 1f) {
                 continue;
             }

             var v = closestRaycastHit.point;
             v.y = m_AimAssist.transform.position.y;
             v = (v - m_AimAssist.transform.position).normalized;
             float dot = Vector3.Dot(forword, v);
             //如果在视野后方，则不在锁定范围
             if (dot < 0f) {
                 continue;
             }
             //要选择当前锁定目标的右边的
             if (_target && rightSide) {
                 if(Vector3.Cross(forword, v).y < 0) {
                     continue;
                 }
             }

             if (_target && !rightSide) {
                 if (Vector3.Cross(forword, v).y > 0) {
                     continue;
                 }
             }

             m_LockedTarget = closestRaycastHitTransform;
             break;
         }
     }
 }
```
效果

<video src='https://user-images.githubusercontent.com/11385187/191199645-2c1c3e49-44a0-4c5e-8d93-4888fd5fb1d4.mp4' controls='controls'></video>
<br>
这憨货石头人和这斧子就是我昨天的成果，下一个目标就是尝试下不同武器的切换和不同的动作表现。:speak_no_evil:

## 2022/9/27
### 不同武器的切换和不同的动作表现
<video src='https://user-images.githubusercontent.com/11385187/192451208-c496f322-2e49-4a68-b9bd-c5703bcfe65a.mp4' controls='controls'></video>
<br>
参照**黑魂III**不同武器的表现，分两类：
* 装备不同武器后的基本动画（idle + walk + run）只是复写了两个手臂的动作，其他部位仍旧是用的人物的基本动画。 比如装备冰狗的大锤的时候，就是右手把锤子抗在肩上，别的部位不变。
* 攻击时，全部用的武器的全套动作。一般人物的位移也是用的动作里的，Rotation看武器情况用不用动画里的，通常是不用的，即玩家在攻击过程可以改变攻击的朝向。
### 连续攻击（combos）
几乎所有的ARPG都能连击，即在一定的时间内连续按下攻击键，能触发combos。<br>
UCC也提供了Combos的机制，摸索了几天，略懂。在此记录一下。<br>
第一步需要把所有的连击动作在Animator里串起来，如视频中的剑的攻击有3段
![image](https://user-images.githubusercontent.com/11385187/192463186-e2656239-e3ad-4f9b-b6ac-abf44ee28b96.png)
<br>
设置好各个Transition的条件，需要每个State都要有Transition，否则动作很容易卡住。
第二步在每个攻击动作加上两个帧事件<br>

![image](https://user-images.githubusercontent.com/11385187/192480189-780b674b-3849-47bd-b968-5dfd8072f15d.png)
![image](https://user-images.githubusercontent.com/11385187/192480275-c3ca853c-ed19-441b-88da-768473ee0ab4.png)
<br>
调了两种类型的武器单手剑和双手斧子，发现如果想要Combos连贯些或者说是更容易按出来一些，根据UCC 触发Combos的机制，需要调整**OnAnimatorItemUse**和**OnAnimatorItemUseComplete**的帧事件的位置，然后试下手感，然后调，然后试。。。。。直到你满意或者崩溃为止~~~
第三步在武器上的MeleeWeapon的组件里有个**Animator Audio**的字段，设置上攻击段的参数<br>
![image](https://user-images.githubusercontent.com/11385187/192481262-ea054490-f660-40f5-a5fe-ad90e4d6c8a3.png)
### Combos 触发原理
用一个例子说明<br>
![image](https://user-images.githubusercontent.com/11385187/192820899-3da48573-882f-43c8-850a-1feeed338a17.png)
<br>
如上图，有3段攻击，假设第一段攻击的 **OnAnimatorItemUse** 在 **1s** 时刻触发， **OnAnimatorItemUseComplete** 在 **1.5s**时刻触发。有下列情况：
* 第一段攻击在0s时刻开始
* 0.6s 时刻收到用户攻击输入， 但是由于第一段攻击的 **OnAnimatorItemUse** 没有触发，所以此次的攻击输入不能打断第一段的攻击，于是第一段攻击继续
* 1.2s 时刻收到用户攻击输入， 由于此时第一段攻击的 **OnAnimatorItemUse** 已经触发，且 **OnAnimatorItemUseComplete** 没有触发，所以能触发Combos，第一段攻击被打断，进入到第二段攻击。
* 但如果在1.6s 时刻收到用户攻击输入，由于**OnAnimatorItemUseComplete**已经触发了，所以也无法触发Combos
* 其他段攻击以此类推


## 2022/9/28
### 翻滚（Roll）
翻滚是魂游戏的标配了，所以我也想（必须）有。<br>
先记录下我的视频<br>

<video src="https://user-images.githubusercontent.com/11385187/192824055-3a65e1dc-fe30-48d6-9281-74920482f6f9.mp4" controls="controls"></video>
<br>

翻滚也分两种时态
* 冒险时，即如同冒险时行走一样，任何翻滚都是向玩家的前方翻滚
* 战斗时，此时会有左右前后之分

据此的话，动画状态就需要添加两种时态
![image](https://user-images.githubusercontent.com/11385187/192825227-c893739f-eb5d-4e3f-8aa5-7d6b4b33f151.png)
由于翻滚是用到了全身骨骼，所以放到了 **full body layer**。然后战斗时由于涉及到了多个方向的翻滚动作，且因变量是 **HorizontalMovement** 和 **ForwardMovement**， 故而做成了一个二维的 BlendTree。
![image](https://user-images.githubusercontent.com/11385187/192826131-d01729c6-3785-46ed-b1f9-9662ac30a1c3.png)
![image](https://user-images.githubusercontent.com/11385187/192826189-09cdae06-c7b1-4645-a7a3-9a2c666ec3ce.png)

接下来就是撸代码了，需要实现一个 Roll 的 Ability。。。。<br>
*为什么在使用 Roll 能力时会自动结束掉 LockEnemy 能力？？？？*<br>
*调试代码。。。原因定位到了，原来时 UCC 在使用一个 Ability 时会尝试强制停止别的已激活的 Ability。*<br>
*于是我在 LockEnemy 中复写了 CanForceStopAbility 方法，强行返回 false， 即不让停止。。。。*<br>
*在使用 Roll 能力时会自动结束掉 LockEnemy 能力解决掉了，但感觉好不优雅，于是寻求别的方法*<br>
*发现UCC自带的Aim和我的LockEnemy差不多，但它是一个ItemAbility，于是乎把 LockEnemy 继承自 ItemAbility， 果然也是可以的*<br>

