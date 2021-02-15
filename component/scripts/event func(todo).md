事件函数
======
Unity 中提供了大量其他事件函数；可在 MonoBehaviour 类的脚本参考页面中找到事件函数的完整列表以及详细的事件函数用法说明。以下是一些最常见和最重要的事件。

常规更新事件
-----
在渲染帧之前以及计算动画之前都会调用 Update 函数。
```
void Update() {
    float distance = speed * Time.deltaTime * Input.GetAxis("Horizontal");
    transform.Translate(Vector3.right * distance);
}
```

在每次物理更新之前都会调用一个称为 FixedUpdate 的单独事件函数。**由于物理更新和帧更新不会以相同频率进行，所以如果将物理代码放在 FixedUpdate 函数而不是 Update 中，此代码将产生更准确的结果**。
```
void FixedUpdate() {
    Vector3 force = transform.forward * driveForce * Input.GetAxis("Vertical");
    rigidbody.AddForce(force);
}
```

为场景中的所有对象调用 Update 和 FixedUpdate 函数之后以及计算所有动画之后，如果能进行其他更改，也会很有用。

一个例子是摄像机应该聚焦于目标对象；必须在目标对象移动后才能调整摄像机的方向。另一个例子是脚本代码应该覆盖动画的效果（例如，让角色的头部看向场景中的目标对象）。可使用 LateUpdate 函数来处理这些类型的情况。
```
void LateUpdate() {
    Camera.main.transform.LookAt(target.transform);
}
```

初始化事件
------
**在第一帧之前或开始对象的物理更新之前**需要调用 Start 函数。**场景加载时会为场景中的每个对象**调用 Awake 函数。

请注意，虽然各种对象的 Start 和 Awake 函数的调用顺序是任意的，**但在调用第一个 Start 之前，所有 Awake 都要完成**。

这意味着 Start 函数中的代码可以利用先前在 Awake 阶段执行的其他初始化。

GUI 事件
------
Unity 有一个系统用于渲染场景中主要操作的 GUI 控件，并响应对这些控件的点击。此代码的处理方式与正常的帧更新有些不同，因此应将此代码置于定期调用的 OnGUI 函数中。
```
void OnGUI() {
    GUI.Label(labelRect, "Game Over");
}
```
此外，还可以检测场景中出现的游戏对象上发生的鼠标事件。使用此功能可以定位武器或显示当前在鼠标指针下的角色的相关信息。

借助一系列 OnMouseXXX 事件函数（例如 OnMouseOver、OnMouseDown）可以让脚本对用户的鼠标操作做出反应。

物理事件
------
物理引擎将通过调用该对象的脚本上的事件函数来报告对象的碰撞情况。

**在进行接触、保持接触和中断接触时，将调用 OnCollisionEnter、OnCollisionStay 和 OnCollisionExit 函数**。

**对象的碰撞体配置为触发器（即，碰撞体只检测某物何时进入而不进行物理反应）时，将调用相应的 OnTriggerEnter、OnTriggerStay 和 OnTriggerExit 函数**。

如果在物理更新期间检测到多次接触，可能连续多次调用这些函数，因此会将一个参数传递给函数，从而提供碰撞的详细信息（位置、进入对象标识等）。

```
void OnCollisionEnter(otherObj: Collision) {
    if (otherObj.tag == "Arrow") {
        ApplyDamage(10);
    }
}
```

事件函数的执行顺序
======
脚本生命周期流程图
------
<img src="https://docs.unity3d.com/cn/2018.4/uploads/Main/monobehaviour_flowchart.svg">

加载第一个场景
------
场景开始时将调用以下函数（为场景中的每个对象调用一次）。

- Awake：始终在任何 Start 函数之前并在实例化预制件之后调用此函数。（如果游戏对象在启动期间处于非活动状态，则在激活之后才会调用 Awake。）

- OnEnable：（仅在对象处于激活状态时调用）在启用对象后立即调用此函数。在创建 MonoBehaviour 实例时（例如加载关卡或实例化具有脚本组件的游戏对象时）会执行此调用。

- OnLevelWasLoaded：执行此函数可告知游戏已加载新关卡。

请注意，**对于添加到场景中的对象**，在为任何对象调用 Start 和 Update 等函数之前，会为 __所有__ 脚本调用 Awake 和 OnEnable 函数。当然，在游戏运行过程中实例化对象时，不能强制执行此调用。

Editor
------
- Reset：调用 Reset 可以在脚本首次附加到对象时以及使用 Reset 命令时初始化脚本的属性。

在第一次帧更新之前
------
- Start：仅当启用脚本实例后，才会在第一次帧更新之前调用 Start。
**对于添加到场景中的对象**，在为任何脚本调用 Update 等函数之前，将在所有脚本上调用 Start 函数。当然，在游戏运行过程中实例化对象时，不能强制执行此调用。

帧之间
------
- OnApplicationPause：在帧的结尾处调用此函数（在正常帧更新之间有效检测到暂停）。在调用 OnApplicationPause 之后，将发出一个额外帧，从而允许游戏显示图形来指示暂停状态。

更新顺序
-------
跟踪游戏逻辑和交互、动画、摄像机位置等的时候，可以使用一些不同事件。常见方案是在 Update 函数中执行大多数任务，但是也可以使用其他函数。

- FixedUpdate：调用 FixedUpdate 的频度常常超过 Update。如果帧率很低，可以每帧调用该函数多次；如果帧率很高，可能在帧之间完全不调用该函数。在 FixedUpdate 之后将立即进行所有物理计算和更新。在 FixedUpdate 内应用运动计算时，无需将值乘以 Time.deltaTime。这是因为 FixedUpdate 的调用基于可靠的计时器（独立于帧率）。

- Update：每帧调用一次 Update。这是用于帧更新的主要函数。

- LateUpdate：每帧调用一次 LateUpdate __（在 Update__ 完成后）。LateUpdate 开始时，在 Update 中执行的所有计算便已完成。LateUpdate 的常见用途是跟随第三人称摄像机。如果在 Update 内让角色移动和转向，可以在 LateUpdate 中执行所有摄像机移动和旋转计算。这样可以确保角色在摄像机跟踪其位置之前已完全移动。
