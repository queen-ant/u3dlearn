刚体
===========
刚体 (_Rigidbody_)使 __游戏对象__ 的行为方式受物理控制。刚体可以接受力和扭矩，使对象以逼真的方式移动。任何游戏对象都必须包含受重力影响的刚体，行为方式基于施加的作用力（通过脚本），或通过 NVIDIA PhysX 物理引擎与其他对象交互。

操纵变换与刚体的最大区别在于力的运用。刚体可以接受力和扭矩，但变换不能。变换可以被平移和旋转，但这与使用物理引擎不同。

父子关系
------
当对象处于物理控制下时，**对象的移动方式在一定程度上独立于其变换父对象的移动方式**。

如果移动任何父对象，它们会将刚体子对象拉到自己身边。但是，刚体仍然会因重力而降落，并会对碰撞事件作出反应。

脚本控制
-----------
由于刚体组件会接管附加到的游戏对象的运动，因此不应试图借助脚本通过更改变换（_Transform_）属性（如位置和旋转）来移动游戏对象。相反，应该施加 __力__ 来推动游戏对象并让物理引擎计算结果。

要执行此操作，可在对象的刚体上调用 AddForce() 和 AddTorque()。

在某些情况下，可能希望游戏对象具有刚体，并让刚体的运动摆脱物理引擎的控制。例如，可能希望直接从脚本代码控制角色，但仍允许触发器检测角色。

脚本产生的这种非物理运动称为 _运动学_ 运动。刚体组件有一个名为 _IsKinematic_ 的属性，该属性可以让刚体摆脱物理引擎的控制，并允许通过脚本以运动学方式来移动刚体（只受transform影响）。**运动刚体将影响其他对象，但它们自身不会受到物理影响（不受force影响）**

可以通过脚本来更改 _IsKinematic_ 的值，从而为某个对象开启和关闭物理引擎，**但这会产生性能开销，应谨慎使用。**

动画
-----
在某些情况下，主要需要创建布娃娃效果，因此有必要在动画和物理之间切换对象的控制。为此，可将刚体标记为 isKinematic。虽然刚体标记为 __isKinematic__，但不会受到碰撞、力或物理系统任何其他部分的影响。这意味着，必须通过直接操作变换组件来控制对象。运动刚体将影响其他对象，但它们自身不会受到物理影响。例如，附加到运动对象的关节将约束附加到关节的所有其他刚体，并且运动刚体将通过碰撞影响其他刚体。

睡眠机制
----------
当刚体移动速度低于规定的最小线性速度或转速时，物理引擎会认为刚体已经停止。发生这种情况时，游戏对象在受到碰撞或力之前不会再次移动，因此将其设置为“睡眠”模式。

这种优化意味着，在刚体下一次被“唤醒”（即再次进入运动状态）之前，不会花费处理器时间来更新刚体。

在大多数情况下，刚体组件的睡眠和唤醒是透明发生的。**但是，如果通过修改 __变换__ 位置将静态碰撞体（即，没有刚体的碰撞体）移入游戏对象或远离游戏对象，则可能无法唤醒游戏对象。**

这种情况下可能会导致问题，例如，已经从刚体游戏对象下面移走地板时，刚体游戏对象会悬在空中。在这种情况下，可以使用 WakeUp 函数显式唤醒游戏对象。

属性
------
|属性：	|功能：|
|  ----  | ----  |
|Mass	|对象的质量（默认为千克）。|
|Drag	|根据力移动对象时影响对象的空气阻力大小。0 表示没有空气阻力，无穷大使对象立即停止移动。|
|Angular Drag	|根据扭矩旋转对象时影响对象的空气阻力大小。0 表示没有空气阻力。请注意，如果直接将对象的 Angular Drag 属性设置为无穷大，无法使对象停止旋转。|
|Use Gravity	|如果启用此属性，则对象将受重力影响。|
|Is Kinematic	|如果启用此选项，则对象将不会被物理引擎驱动，只能通过 __变换 (Transform)__ 对其进行操作。对于移动平台，或者如果要动画化附加了 HingeJoint 的刚体，此属性将非常有用。|
|Interpolate	|仅当在刚体运动中看到急动时才尝试使用提供的选项之一。|
|- None	|不应用插值。|
|- Interpolate	|根据前一帧的变换来平滑变换。|
|- Extrapolate	|根据下一帧的估计变换来平滑变换。|
|Collision Detection	|用于防止快速移动的对象穿过其他对象而不检测碰撞。|
|- Discrete	|对场景中的所有其他碰撞体使用离散碰撞检测。其他碰撞体在测试碰撞时会使用离散碰撞检测。用于正常碰撞（这是默认值）。|
|- Continuous	|对动态碰撞体（具有刚体）使用离散碰撞检测，并对静态碰撞体（没有刚体）使用基于扫掠的连续碰撞检测。设置为 __连续动态 (Continuous Dynamic)__ 的刚体将在测试与该刚体的碰撞时使用连续碰撞检测。其他刚体将使用离散碰撞检测。用于 __连续动态 (Continuous Dynamic)__ 检测需要碰撞的对象。（此属性对物理性能有很大影响，如果没有快速对象的碰撞问题，请将其保留为 Discrete 设置）|
|- Continuous Dynamic	|对设置为 __连续 (Continuous)__ 和 __连续动态 (Continuous Dynamic)__ 碰撞的游戏对象使用基于扫掠的连续碰撞检测。还将对静态碰撞体（没有刚体）使用连续碰撞检测。对于所有其他碰撞体，使用离散碰撞检测。用于快速移动的对象。|
|- Continuous Speculative	|对刚体和碰撞体使用推测性连续碰撞检测。这也是可以设置运动物体的唯一 CCD 模式。该方法通常比基于扫掠的连续碰撞检测的成本更低。|
|Constraints	|对刚体运动的限制：|
|- Freeze Position	|有选择地停止刚体沿世界 X、Y 和 Z 轴的移动。|
|- Freeze Rotation	|有选择地停止刚体围绕局部 X、Y 和 Z 轴旋转。|

连续碰撞检测（(CCD)）
-------
**连续碰撞检测是一种阻止快速移动的碰撞体相互穿过的功能**。

使用正常 (Discrete) 碰撞检测时，如果对象在一个帧中位于某个碰撞体的一侧，而在下一帧中已经穿过了碰撞体，便属于彼此穿过的情况。要解决此问题，可在快速移动对象的刚体上启用连续碰撞检测。

将碰撞检测模式设置为 Continuous 可防止刚体穿过任何静态（即非刚体）网格碰撞体。

设置为 Continuous Dynamic 也会防止刚体穿过任何其他支持的刚体（即，碰撞检测模式设置为 Continuous 或 Continuous Dynamic 的刚体）。 **盒型碰撞体、球形碰撞体和胶囊碰撞体支持连续碰撞检测**。

请注意，连续碰撞检测的目的是作为一种安全机制，可在对象会相互穿过的情况下捕获碰撞，**但不会提供精确物理碰撞的结果；所以如果遇到快速移动对象的问题，仍然可以考虑在 TimeManager 检视面板中降低固定时间步长值以使模拟更准确**。

**Unity 提供以下 CCD 方法**：

- 基于扫掠的 CCD
- 推断性 CCD

### 基于扫掠的 CCD

基于扫掠的 CCD 采用撞击时间 (TOI) 算法，通过**扫掠对象的前向轨迹来计算对象的潜在碰撞**（采用对象的当前速度）。如果沿对象移动方向有接触，该算法会计算撞击时间并移动对象直至达到该时间。该算法可从该时间开始执行子步骤，即计算 TOI 之后的速度，然后重新扫掠，代价是需要经历更多的 CPU 周期。

然而，**因为此方法依赖于线性扫掠，所以会忽略物体的角运动，这在对象迅速旋转时会引起穿隧效应**。

例如，弹球机上的弹球杆固定在一端，围绕一个固定点旋转。弹球杆只做角运动，不做线性运动。因此，很容易打不中弹球：
<img src="https://docs.unity3d.com/cn/2018.4/uploads/Main/SpeculativeCCD1.gif">
已启用 Continuous Dynamic 属性的细杆游戏对象。绕轴心点快速旋转时，此杆不会与球体接触。

已启用 Continuous Dynamic 属性的细杆游戏对象。绕轴心点快速旋转时，此杆不会与球体接触。

此方法的另一个问题是性能问题。**如果附近有大量启用 CCD 的高速对象，CCD 的开销将由于进行额外的扫掠而很快增加**，因此物理引擎不得不执行更多的 CCD 子步骤。

### 推断性 CCD
推断性 CCD 的工作原理是基于对象的线性运动和角运动增大一个对象的粗筛阶段(Broad-Phase)轴对齐最小包围盒 (AABB, axis-aligned minimum bounding box)。该算法是一种推测性的算法，因为会选取下一物理步骤中的所有潜在触点。然后将所有触点送入解算器，因此可确保满足所有的触点约束，使对象不会穿过任何碰撞。

下图显示了从 t0 开始移动的球体如何获得预期的 t1 位置（如果其路径中没有墙）。通过使用目标姿势将 AABB 扩大，推测性算法选取与 n1 和 n2 法线之间的两个触点。然后，该算法告诉解算器遵循这些触点，使球体不会穿过墙壁。
<img src="https://docs.unity3d.com/cn/2018.4/uploads/Main/SpeculativeCCD2.png">

基于当前速度并扩大的 AABB 有助于检测沿运动轨迹的所有潜在触点，使解算器能够防止对象穿过去。

推测性 CCD 的成本通常低于基于扫掠的方法，因为只在碰撞检测阶段（而不在求解和积分阶段）计算触点。此外，**由于推测性 CCD 根据对象的线性运动和角运动来扩展粗筛阶段 AABB，因此能发现基于扫掠的 CCD 可能遗漏的触点**。

<img src="https://docs.unity3d.com/cn/2018.4/uploads/Main/SpeculativeCCD3.gif">

但是，**推测性 CCD 可能会导致幽灵碰撞**；在这种碰撞中，对象的运动受到推测性触点的影响，而这是不应发生的。这是**因为推测性 CCD 根据最近点算法收集所有潜在触点，所以触点法线不太准确**。这通常会使高速对象沿着细分的碰撞特征滑动并跳起来，但不应该这样。例如，下图中，球体从 t0 开始向右水平移动，积分后的预测位置为 t1。扩大后的 AABB 与框形 b0 和 b1 重叠，而 CCD 在 c0 和 c1 产生两个推测性触点。由于推测性 CCD 使用最近点算法来生成触点， __c0__ 具有非常倾斜的法线，因此解算器会将其视作斜坡。

<img src="https://docs.unity3d.com/cn/2018.4/uploads/Main/SpeculativeCCD4.png">

这种非常倾斜的法线导致 t1 在积分后向上跳动，而不是笔直向前移动：

<img src="https://docs.unity3d.com/cn/2018.4/uploads/Main/SpeculativeCCD5.gif">

**推测性 CCD 还可能导致发生穿隧，因为只会在碰撞检测阶段计算推测性触点**。在触点求解过程中，如果一个对象从解算器获得太多能量，在积分后，**其最终位置可能在初始扩大的 AABB 之外**。如果在紧邻 AABB 的外部发生碰撞，对象会从右边穿出。

例如，下图显示了球体从 t0 向左移动，而球杆顺时针旋转。如果球体从撞击中获得太多能量，最终可能离开扩大的 AABB（红点矩形），落在 t1 处。如果在紧邻 AABB 的外部发生碰撞（如下面的蓝色框所示），球体最终可能会从右边穿出。这是因为解算器只计算扩大的 AABB 的内部触点，在求解和积分阶段不会执行碰撞检测。

<img src="https://docs.unity3d.com/cn/2018.4/uploads/Main/SpeculativeCCD6.png">

尝试1: 高速移动的弹性碰撞
=======
模拟物理过程：https://www.bilibili.com/video/BV1bt41147H5

Hierarchy下有Main Camera、PlaneX、PlaneY

附加Momentum.cs脚本到PlaneX，创建方块。方块使用prefab，collider物理材质Dynamic Friction = 0，Static Friction = 0，Bounciness = 1。
```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Momentum : MonoBehaviour
{
    public GameObject cube;
    private Rigidbody body1;
    private Rigidbody body2;
    void Start()
    {
        GameObject camera = GameObject.Find("Main Camera");
        camera.transform.Translate(5.0f, 1.0f, 1.0f);
        camera.transform.LookAt(Vector3.zero);
        body1 = Instantiate(cube, new Vector3(0, 0.1f, 3.0f), Quaternion.identity).GetComponent<Rigidbody>();
        body2 = Instantiate(cube, new Vector3(0, 0.5f, 4.0f), Quaternion.identity).GetComponent<Rigidbody>();

        body1.mass = 1;
        body2.mass = 2500;

        body1.name = "SmallOne";
        body2.name = "BigOne";

        body1.transform.localScale = new Vector3(0.2f, 0.2f, 0.2f);

        body2.AddForce(new Vector3(0, 0, -3.0f), ForceMode.VelocityChange);
    }

    void Update()
    {

    }

    void FixedUpdate()
    {

    }
}

```
附加脚本CollideCount.cs到方块prefab
```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CollideCount : MonoBehaviour
{
    private int count;
    void Start()
    {
        count = 0;
        
        //刚体阻力设为0
        GetComponent<Rigidbody>().drag = 0;
        GetComponent<Rigidbody>().angularDrag = 0;
        
        //锁定XY轴的移动和XYZ轴的旋转
        GetComponent<Rigidbody>().constraints = RigidbodyConstraints.FreezePositionX | RigidbodyConstraints.FreezePositionY
        | RigidbodyConstraints.FreezeRotationX | RigidbodyConstraints.FreezeRotationY | RigidbodyConstraints.FreezeRotationZ;
        
        //使用连续碰撞检测
        GetComponent<Rigidbody>().collisionDetectionMode = CollisionDetectionMode.ContinuousDynamic;
        
        //固定步长设为1毫秒
        Time.fixedDeltaTime = 0.001f;
    }

    void Update()
    {

    }

    void OnCollisionEnter(Collision other)
    {
        if (other.gameObject.name == "PlaneY" || other.gameObject.name == "BigOne")
        {
            count += 1;
        }
    }

    void OnCollisionExit()
    {
        if (count > 0)
        {
            Debug.Log(count);
        }
        //count最大值为pi*sqrt(m2/m1)，此例中为pi*50
    }

}
```
**1、因为是直线上的动量守恒，所以要使用Rigidbody.constraints来锁定自由度**

**2、因为存在高速运动，所以要设置为CollisionDetectionMode.ContinuousDynamic进行连续碰撞检测，否则会穿过静态碰撞体（此例中使用CollisionDetectionMode.ContinuousSpeculative开启推断性CDD无法得到正确结果）。并且要缩短物理系统工作的固定时间步长，以获得准确的结果（此例中默认步长无法获得正确结果）**
