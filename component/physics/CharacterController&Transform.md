角色控制器（CharacterController）
=======
角色控制器 (Character Controller) 主要用于第三人称玩家控制或者是不使用 __刚体__ 物理组件的第一人称玩家控制。

属性
------
|属性：	|功能：|
|----|----|
|slopeLimit	|将碰撞体限制为爬坡的斜率不超过指示值（以度为单位）。|
|StepOffset	|仅当角色比指示值更接近地面时，角色才会升高一个台阶。该值不应该大于角色控制器的高度，否则会产生错误。|
|skinWidth	|两个碰撞体可以穿透彼此且穿透深度最多为皮肤宽度 (Skin Width)。较大的皮肤宽度可减少抖动。较小的皮肤宽度可能导致角色卡住。合理设置是将此值设为半径的 10%。|
|minMoveDistance	|如果角色试图移动到指示值以下，根本移动不了。此设置可以用来减少抖动。在大多数情况下，此值应保留为 0。|
|center	|此设置将使胶囊碰撞体在世界空间中偏移，并且不会影响角色的枢转方式。|
|radius	|胶囊碰撞体的半径长度。此值本质上是碰撞体的宽度。|
|height	|角色的__胶囊碰撞体__高度。更改此设置将在正方向和负方向沿 Y 轴缩放碰撞体。|
|isGrounded	|在上次移动期间 CharacterController 是否接触地面|
|velocity	|该角色的当前相对速度（请参阅注释）。|
|collisionFlags	|在上次 CharacterController.Move 调用期间，该胶囊体的哪个部分与环境发生了碰撞。|
|detectCollisions	|确定其他刚体或角色控制器是否与该角色控制器碰撞（默认情况下始终启用它）。|
|enableOverlapRecovery	|启用或禁用重叠恢复。 启用或禁用重叠恢复。用于在检测到重叠时取消角色控制器从静态对象的穿透。|

公共函数
------
|||
|-----|----|
|Move	|采用绝对移动增量的更复杂移动函数。|
|SimpleMove	|以 speed 移动该角色。|

消息
-----
|||
|-----|----|
|OnControllerColliderHit	|当该控制器在执行 Move 时撞到碰撞体时调用 OnControllerColliderHit。|


详细信息
------
角色控制器只是一个胶囊形状的 __碰撞体__ ，可以通过脚本来命令这个碰撞体向某个方向移动。然后，控制器将执行运动，但会受到碰撞的约束。控制器将沿着墙壁滑动，走上楼梯（如果低于 Step Offset 值），并走上 Slope Limit 设置范围内的斜坡。

**控制器本身不会对力作出反应，也不会自动推开刚体**。

**如果要通过角色控制器来推动刚体或对象，可以编写脚本通过 OnControllerColliderHit() 函数对与控制器碰撞的任何对象施力**。

另一方面，如果希望玩家角色受到物理组件的影响，那么可能更适合使用刚体，而不是角色控制器。

微调角色
------
可以修改 Height 和 Radius 属性来适应角色的网格。对于人形角色，**建议始终使用 2 米左右的高度**。如果轴心点并非刚好在角色的中心，还可以修改胶囊体的 Center 属性。

Step Offset 属性也可能有影响，对于身高 2 米的人，**请确保此值在 0.1 到 0.4 之间**。

Slope Limit **不应太小。通常，使用 90 度的值效果最佳**。由于胶囊体形状的原因，角色控制器将无法爬墙。

不要被卡住
-------
要正确调整角色控制器， __Skin Width__  属性是最重要的属性之一。 **如果角色被卡住，那么很可能是因为 Skin Width 设置过小**。Skin Width 允许对象轻微穿透控制器，但可消除抖动并防止被卡住。

**最好是让 Skin Width 的值至少大于 0.01 并且比 Radius 的值大 10%**。

**建议将 Min Move Distance 保持为 0**。

Tips
-----
- 如果发现角色经常被卡住，请尝试调整 Skin Width。
- 如果是自己编写脚本，则角色控制器可能会影响使用物理组件的对象。
- 对象无法通过物理组件来影响角色控制器。
- 请注意，在 Inspector 中更改角色控制器属性将在场景中重新创建控制器，因此任何现有的触发器触点都将丢失，并且在再次移动控制器之前，不会收到任何 OnTriggerEntered 消息。
- 在查询中使用的角色控制器胶囊体（比如射线投射）可能会略有缩小。因此，在某些极端情况下，即使查询似乎命中了角色控制器的辅助图标，但实际可能未命中。

输入（Input）
=====
使用 Input 设置（顶部菜单：Edit > Project Settings，然后选择 Input 类别）可定义项目的输入轴和游戏操作。

要添加更多输入轴，请增大 Size 属性中的值。

### 虚拟轴
每个项目在创建时都具有以下默认输入轴：
- Horizontal 和 Vertical 映射到 w、a、s、d 键和箭头键。
- Fire1、Fire2 和 Fire3 分别映射到 Control 键、Option (Alt) 键和 Command 键。
- Mouse X 和 Mouse Y 映射到鼠标移动的增量。
- Window Shake X 和 Window Shake Y 映射到窗口的移动。

每个输入轴提供以下属性列表：

|属性	|功能|
|----|----|
|Name	|输入指代游戏启动器中的轴并通过脚本引用的字符串。|
|Descriptive Name	|输入游戏启动器中显示的 Positive Button 功能的详细定义。|
|Descriptive Negative Name	|输入游戏启动器中显示的 Negative Button 功能的详细定义。|
|Negative Button	|输入向轴发送负值的按钮的名称。|
|Positive Button	|输入向轴发送正值的按钮的名称。|
|Alt Negative Button	|输入向轴发送负值的辅助按钮的名称。|
|Alt Positive Button	|输入向轴发送正值的辅助按钮的名称。|
|Gravity	|设置输入重新居中的速度。仅当 Type 设置为 key / mouse button 时，此属性才适用。|
|Dead	|任何小于此数字的正值或负值都视为零。适用于游戏杆。|
|Sensitivity	|对于键盘输入，值越大，响应越快。较小的值将更加平滑。对于鼠标增量，该值会缩放实际鼠标增量。|
|Snap	|启用此选项可在接收相反输入后立即将轴值重置为零。仅当 Type 设置为 key / mouse button 时，此属性才适用。|
|Invert	|启用此选项可使正值按钮向轴发送负值，反之亦然。|
|Type	|选择此轴期望的输入类型。|
|- _Key / Mouse Button_	|任何一种按钮。|
|- _Mouse Movement_	|鼠标增量和滚轮。|
|- _Window Movement_	|用户移动窗口。|
|- _Joystick Axis_	|模拟游戏杆|
|Axis|轴选择来自设备（游戏杆、鼠标、游戏手柄等）的输入轴。默认为 X 轴。|
|Joy Num	|选择应该使用哪个游戏杆。默认为从所有游戏杆检索输入。注意：这仅用于输入轴而不是按钮。|

**在 Input 设置中设置的所有轴都有两个用途**：
- 允许在脚本中按轴名称引用输入。若要使用输入来进行任何类型的**移动行为，请使用 Input.GetAxis**。 **请将 Input.GetButton 仅用于事件等操作。不要将它用于移动操作。**
- 允许游戏玩家根据自己的喜好自定义控制方式。

**轴的值介于 –1 到 1 之间。中性位置为 0**。 这是游戏杆输入和键盘输入的情况。

但是，Mouse Delta 和 Window Shake Delta 是鼠标或窗口在最后一帧中移动的程度。这意味着，**当用户快速移动鼠标时，它可以大于 1 或小于 –1**。

**可以使用相同的名称创建多个轴。获取输入轴时，将返回绝对值最大的轴**。这样就可以将多个输入设备分配给一个轴名称。例如，为键盘输入创建一个轴，用相同名称为游戏杆输入创建一个轴。如果用户正在使用游戏杆，则输入将来自游戏杆，否则输入将来自键盘。这样一来，不必在编写脚本时考虑输入的来源。

**键的名称遵循以下约定**：
- 普通键：“a”、“b”、“c”…
- 数字键：“1”、“2”、“3”…
- 箭头键：“up”、“down”、“left”和“right”
- 键盘键：“[1]”、“[2]”、“[3]”、“[+]”和“[equals]”
- 修饰键：“right shift”、“left shift”、“right ctrl”、“left ctrl”、“right alt”、“left alt”、“right cmd”、“left cmd”
- 鼠标按钮：“mouse 0”、“mouse 1”、“mouse 2”…
- 游戏杆按钮（任何游戏杆）：“joystick button 0”、“joystick button 1”、“joystick button 2”…
- 游戏杆按钮（特定游戏杆）：“joystick 1 button 0”、“joystick 1 button 1”、“joystick 2 button 0”…
- 特殊键：“backspace”、“tab”、“return”、“escape”、“space”、“delete”、“enter”、“insert”、“home”、“end”、“page up”和“page down”
- 功能键：“f1”、“f2”、“f3”…

请注意，执行 Update 前，不会重置 Input 标志。**建议在 Update 循环中进行所有的 Input 调用**。

**游戏启动器**提供所有已定义的轴，并显示每个轴的名称、详细描述和默认按钮。在此处，可以更改轴中定义的任何按钮。因此，最好使用轴（而不是单个按钮）来编写脚本，因为玩家可能希望自定义游戏按钮。

### KeyCode
Event.keyCode 返回的键代码。这些代码直接映射到键盘上的物理键。

键代码可以用于通过 Input.GetKeyDown（**在用户释放键并再次按键之前，它不会返回 true，需要按下状态返回true应使用GetKey**） 和 Input.GetKeyUp 检测键按下和键松开事件。但处理输入时，建议改用 Input.GetAxis 和 Input.GetButton，因为 这允许最终用户对键进行配置。

### 移动设备

iOS 和 Android 设备能够跟踪多根手指同时触摸屏幕的操作。 您可以通过访问 Input.touches 属性数组来访问最后一帧期间每根手指触摸屏幕的状态数据。

设备移动时，其加速度计硬件将报告设备沿三维空间中的三个主轴的线性加速度变化。 您可以使用该数据检测设备的当前方向（相对于地面）以及该方向上的任何即时更改。

硬件将沿每个轴的加速度直接报告为重力值。 值 1.0 表示沿给定轴约 +1g 的荷载，值 -1.0 表示 -1g。 当将设备竖直握住（主按钮位于底部）举到您正前方时， 右侧为正 X 轴，上方为正 Y 轴，指向您的方向为正 Z 轴。

您可以读取 Input.acceleration 属性来获取加速度计读数。 也可以使用 Input.deviceOrientation 属性获取设备在三维空间中的方向的离散估值。 如果您想在用户旋转设备（以不同方式把握设备时）时创建游戏行为，则检测方向变化将非常有用。

注意，每帧可以轮询加速度计硬件数次。 要访问自上一帧以来的所有加速度计样本，可以读取 Input.accelerationEvents 属性数组。 这在重建玩家动作、将加速度数据输入预测器或实现其他精确运动分析时非常有用。


坐标系与空间变换
======
坐标系
------
### 坐标系种类
Unity 中使用到的坐标系分为以下四种

- 世界坐标系 Word Space

即世界空间使用的坐标系，基本单位 unit，x 正方向：左向右， y 正方向：下向上，z 正方向：屏外向屏内。

任何物体使用 Transform.position 即可获得世界坐标值。

场景中根物体使用的就是世界坐标，可在 Inspector 查看世界坐标值。

**对于非根物体则以父物体位置为原点位置使用本地坐标系 Local Space，即相对父物体位置，该物体 Inspector 数值为本地坐标值，可使用 Transform.localposition 获取本地坐标值**

- 屏幕坐标系 Screen Space

基本单位像素，屏幕左下角为（0，0），右上角为（Screen.width，Screen.height），即实际运行屏幕下的游戏窗口像素值，z 为相机世界坐标单位值。

Input.mousePosition 获取的鼠标坐标，Input.GetTouch(0).position 获取触摸坐标。

- 视口坐标系 Viewport Space

左下角为（0，0），右上角为（1，1），z 为相机世界坐标单位值。

适合用于坐标系转换。

- UGUI 坐标系 UGUI Space

基本单位像素，屏幕左上角为（0，0），右下角为（Screen.width，Screen.height）。

### 坐标系转换
世界坐标系 Space.World 与自身坐标系 Space.Self
```C#
// 本地→世界    
transform.TransformPoint(position); //TransformDirection、TransformVector类似
TransformVector
// 世界→本地  
transform.InverseTransformPoint(position);
// 世界→屏幕  
camera.WorldToScreenPoint(position);  
// 世界→视口  
Camera.main.WorldToViewportPoint(position)
// 屏幕→视口  
camera.ScreenToViewportPoint(position);
// 视口→屏幕  
camera.ViewportToScreenPoint(position);
// 视口→世界  
camera.ViewportToWorldPoint(position);
```
空间变换
-----
**Vector3.forward、Vector3.back、Vector3.left、Vector3.right、Vector3.up、Vector3.down 数值是固定的，而transform.forward、 transform.right、transform.up 数值不定，依据物体自身旋转变化，如 transform.forward 为物体 z 轴在世界坐标系中所指方向。**

### 非刚体平移
- Transform.positon 世界位置坐标

- Transform.localPostion 本地位置坐标

- Transform.Translate(Vector3 translation, Space relativeTo = Space.Self)，如果 relativeTo 被省略或设置为 Space.Self，则会相对于变换的**本地轴**来应用该移动

- Transform.Translate(float x, float y, float z, Space relativeTo = Space.Self)，如果 relativeTo 被省略或设置为 Space.Self，则会相对于变换的**本地轴**来应用该移动

- Transform.Translate(Vector3 translation, Transform relativeTo) 如果 relativeTo 为 null，则相对于**世界坐标系**应用移动。

- Transform.Translate(float x, float y, float z, Transform relativeTo) 如果 relativeTo 为 null，则相对于**世界坐标系**应用移动。
```C#
void Update()
{
     // 以每秒1个单位速度延世界坐标系y轴正方向移动
     transform.Translate(Vector3.up * Time.deltaTime, Space.World);
     // 以每秒1个单位速度延主相机本地坐标系x轴正方向移动
     transform.Translate(Vector3.right * Time.deltaTime, Camera.main.transform);
}
```
- public static Vector3 MoveTowards (Vector3 current, Vector3 target, float maxDistanceDelta) 返回Vector3 新位置

以一定速度向目标移动直至到达目标位置，对比插值法可限制最大速度
```C#
// maxDistanceDelta 最大移动距离
void Update()
{
     // 当前帧移动距离
     float step =  speed * Time.deltaTime; 
     // 由当前位置向目标位置移动距离 step 得出新位置，超过终点则返回终点值  
     transform.position = Vector3.MoveTowards(transform.position, target.position, step);
}
```

- public static Vector3 SmoothDamp (Vector3 current, Vector3 target, ref Vector3 currentVelocity, float smoothTime, float maxSpeed= Mathf.Infinity, float deltaTime= Time.deltaTime)

**实现平滑移动**，可控制速度，向量通过某个类似于弹簧-阻尼的函数（它从不超过目标）进行平滑。**最常见的用法是用于平滑跟随摄像机**。

|参数||
|----|-----|
|current	|当前位置。|
|target	|尝试达到的目标。|
|currentVelocity	|当前速度，此值由函数在每次调用时进行修改。|
|smoothTime	|达到目标所需的近似时间。值越小，达到目标的速度越快。|
|maxSpeed	|可以选择允许限制最大速度。|
|deltaTime	|自上次调用此函数以来的时间。默认情况下为 Time.deltaTime。|

```C#
// 通过使用自身修改的速度指针计算当前位置
private Vector3 velocity = Vector3.zero;
void Update()
{
     // 定义目标物体的后上方为目标位置
     Vector3 targetPosition = target.TransformPoint(new Vector3(0, 5, -10));
     // 以每帧变化的速度 velocity 由当前位置向目标位置移动
     transform.position = Vector3.SmoothDamp(transform.position, targetPosition, ref velocity, 0.3F);
}
```

- public static Vector3  Vector3.Lerp(Vector3 a, Vector3 b, float t)

**线性插值**，`return new Vector3(a.x+(b.x-a.x)*t,a.y+(b.y-a.y)*t,a.z+(b.z-a.z)*t);`，也就是说，t为0，返回a；t为1，返回b；t=0.5时返回a、b中间的点。**当t大于1时，返回的还是b；当t小于0，返回的是a**。

- public static Vector3  Vector3.LerpUnclamped(Vector3 a, Vector3 b, float t)

与Lerp不同在于t小于0和大于1时没有限制。

- public static Vector3  Vector3.Slerp(Vector3 a, Vector3 b, float t)

**球面插值**，返回的向量的方向通过a 和 b 的角度之间进行插值， 其 magnitude 在 a 和 b 的大小之间进行插值。参数 t 限制在范围 [0,1] 内。

- public static Vector3  Vector3.SlerpUnclamped(Vector3 a, Vector3 b, float t)

与Lerp不同在于t小于0和大于1时没有限制。

```C#
// Animates the position in an arc between sunrise and sunset. 运动结果是一个拱形

using UnityEngine;
using System.Collections;

public class ExampleClass : MonoBehaviour
{
    public Transform sunrise;
    public Transform sunset;

    // Time to move from sunrise to sunset position, in seconds.
    public float journeyTime = 1.0f;

    // The time at which the animation started.
    private float startTime;

    void Start()
    {
        // Note the time at the start of the animation.
        startTime = Time.time;
    }

    void Update()
    {
        // 弧的中心（将要进行插值的两个向量的起点）
        Vector3 center = (sunrise.position + sunset.position) * 0.5F;

        // 下移中心使弧平滑
        center -= new Vector3(0, 1, 0);

        // 围住弧的两个向量（一个扇形），这里两个向量等长所以是一个圆弧
        Vector3 riseRelCenter = sunrise.position - center;
        Vector3 setRelCenter = sunset.position - center;

        // The fraction of the animation that has happened so far is
        // equal to the elapsed time divided by the desired time for
        // the total journey.
        float fracComplete = (Time.time - startTime) / journeyTime;

        transform.position = Vector3.Slerp(riseRelCenter, setRelCenter, fracComplete);
        transform.position += center;
    }
}
```

### 刚体平移

### 四元数

#### 欧拉角与万向锁
https://krasjet.github.io/quaternion/bonus_gimbal_lock.pdf

#### 四元数描述旋转
https://krasjet.github.io/quaternion/quaternion.pdf

#### 定义
四元数q=a+bi+cj+dk，也可表示为q={a,u},u=bi+cj+dk

i^2=j^2=k^2=ijk=-1

ij=k,jk=i,ki=j

ji=-k,kj=-i,ik=-j

|q|=a^2+b^2+c^2+d^2

#### 基本性质
复数乘法满足交换律，四元数乘法不满足

q*=a-bi-cj-dk

qq*=|q|^2

q^-1 = q*/|q|^2

q1\*q2\* = (q2q1)*

#### 旋转公式
q={cos(t/2),sin(t/2)·u}，有|u|=1，于是|q|=1

则向量v以u为转轴，按右手系旋转t角度，v'=qvq*=qvq^-1

先旋转q1再旋转q2，可复合为q2·q1，是左乘

两个不同的单位四元数 𝑞 与 −𝑞 对应的是同一个旋转

#### Unity中的Quaternion
- **Unity中的Quaternion乘法操作是左侧的四元数先应用旋转，即复合额外旋转写成```transform.rotation *= extraRotation.rotation;```**
- **在Unity中transform.rotation是四元数Quaternion，transform.eulerAngles是欧拉角，而Inspector界面上Transform组件的Rotation是欧拉角。Inspector上的Rotation中的X Y Z值范围是(-180,180)，对于脚本中超出的部分会自动进行计算映射到范围内**
- **Quaternion.Angle返回的不是四元数的夹角，是四元数表示的两个三维空间中的旋转变换的夹角，实际四元数的夹角是该旋转变换夹角的一半。**
- **Quaternion.Lerp线性插值，比起球面插值Quaternion.Slerp高效一些，但是在大弧度的情况下不能保证均匀的角速度。而在弧度非常小时，因为浮点计算的误差，sin(θ)可能会被近似为0.0，从而导致除以0的错误，此时应该用Lerp。**
- **Quaternion.LookRotation，LookAt与LookRotation的参数都相似，但前者是将游戏对象的z轴指向参数所表示的那个点，而后者是将游戏对象的z轴指向参数所表示的向量的方向。**


#### 为什么没有三元数

复数的基是 1和 i 生成的4阶群 {1 i -1 -i}

根据拉格朗日陪集定理, 子群的阶要能整除父群的阶,

所以能兼容4阶子群的最小父群也要8阶,

这个就是四元数群g = {1 i j k -1 -i -j -k}

如果要完整兼容复数空间，8阶和4阶中间没有三元数的空间了

### 非刚体旋转

### 刚体旋转

尝试1：通过CharacterController组件移动物体
=====
其中柱体附加角色控制器
<img src="https://s16.directupload.net/images/210220/u6nhwm7c.png">

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CharaController : MonoBehaviour
{
    public float speed;
    public float jumpSpeed;
    public float gravity;
    private CharacterController controller;
    private Vector3 moveDirection;
    private Vector3 hitNormal;
    private bool isGlide;
    private CollisionFlags collisionFlags;

    void Start()
    {
        speed = 5.0f;
        jumpSpeed = 5.0f;
        gravity = 20.0f;

        hitNormal = Vector3.zero;
        moveDirection = Vector3.zero;
        isGlide = false;
        collisionFlags = CollisionFlags.None;

        controller = GetComponent<CharacterController>();
        gameObject.transform.localPosition = new Vector3(0, 1, 0);

        GameObject camera = GameObject.Find("Main Camera");
        camera.transform.Translate(GameObject.Find("Plane").transform.position.x - 1.0f, 5.0f, -5.0f);
        camera.transform.LookAt(GameObject.Find("Plane").transform.position);
    }

    void OnControllerColliderHit(ControllerColliderHit hit)
    {
        hitNormal = hit.normal;
        if (hit.rigidbody != null)
        {
            hit.rigidbody.AddForce(-hitNormal, ForceMode.VelocityChange);
        }

    }

    void Update()
    {
        float angle =  Vector3.Angle(hitNormal, Vector3.up);
        if (angle < controller.slopeLimit || collisionFlags == CollisionFlags.Sides)
        {
            isGlide = false;
        }
        else
        {
            isGlide = true;
        }

        if (!isGlide)
        {
            if (controller.isGrounded)
            {
                moveDirection = new Vector3(Input.GetAxis("Horizontal"), 0.0f, Input.GetAxis("Vertical"));
                moveDirection = Mathf.Cos(angle*Mathf.PI/180.0f) * speed * moveDirection;
                if (Input.GetButton("Jump"))
                {
                    moveDirection.y = jumpSpeed;
                }
            }
            else if (Input.GetButton("Horizontal") || Input.GetButton("Vertical")) //跳在空中也能移动
            {
                float y = moveDirection.y; //保存重力影响下的垂直分量，即垂直速度矢量
                moveDirection = new Vector3(Input.GetAxis("Horizontal"), 0.0f, Input.GetAxis("Vertical"));
                moveDirection = speed * moveDirection;

                moveDirection.y = y;
            }
        }
        else
        {
            float y = moveDirection.y;
            moveDirection = hitNormal - hitNormal.y * Vector3.up;
            moveDirection.y = y;
        }

        moveDirection.y = moveDirection.y - (gravity * Time.deltaTime);

        collisionFlags = controller.Move(Time.deltaTime * moveDirection);
    }
}

```
### 总结
**1、CharacterController中Move和SimpleMove的区别：**

Move方法的实际作用和Transform.Translate几乎一样。而且它计算速度是以帧计算的，所以**需要乘每帧时间间隔**。

CharacterController.Move(Vector3.forward * Time.deltaTime * 5)

**返回值**是CollisionFlags对象，可以描述与任何物体碰撞的信息

而SimpleMove，当你使用它来移动你的目标时，它就具备了“重力”效果，**不受Y轴速度影响，只有X轴和Z轴方向的有效**。并且移动的时候，它是以秒为单位，**不用乘Time.deltaTime**。

CharacterController.SimpleMove(Vector3.forward * 5)

**返回值**(BOOL类型)，角色接触地面则返回true，否则返回false

**2、角色控制器的collider是胶囊体，底部是半球，所以无法通过的台阶高度要设置尽量大一些，否则很容易挤上去，高度4倍stepOffset差不多，不应该小于1。**

**3、slopeLimit可以阻止角色通过过陡的斜坡，但是不能阻止跳跃通过**。要阻止跳跃过陡的斜坡，编写代码进行坡度计算然后进行判断即可。

碰撞时调用CharacterController.OnControllerColliderHit(ControllerColliderHit)

**ControllerColliderHit**变量：
|||
|----|----|
|collider	|被该控制器撞击的碰撞体。|
|controller	|撞击该碰撞体的控制器。|
|gameObject	|被该控制器撞击的游戏对象。|
|moveDirection	|在发生碰撞时 CharacterController 移动的方向。|
|moveLength	|该角色在撞到该碰撞体前行进的距离。|
|normal	|在世界空间中我们碰撞的表面的法线。|
|point	|世界空间中的撞击点。|
|rigidbody	|被该控制器撞击的刚体。|
|transform	|被该控制器撞击的变换组件。|

**normal向量与Vector3.up的夹角就是坡度倾角**。本次尝试的代码中，上坡时速度和平地保持一致，并且当坡度超出slopeLimit时会自动下滑且无法跳跃。

**4、对于人形对象的移动，要实现足部贴合如楼梯等地形的效果，需使用FootIk**

尝试2：通过Transform模拟自转和公转
=====

