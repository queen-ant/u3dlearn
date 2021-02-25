# 关节
所有使用关节的对象都必须使用 __刚体__

## 固定关节（fixed joint）

将两个物体永远以相对的位置固定在一起，即使发生物理改变，它们之间的相对位置也将不变。

可使用 Break Force 和 Break Torque 属性来设置关节强度的限制。如果这些值小于无穷大，并对该对象施加大于这些限制的力/扭矩，则其固定关节将被破坏并将摆脱其约束的束缚。

|属性：	|功能：|
|----|----|
|Connected Body	|对关节所依赖的刚体的引用（可选）。如果未设置，则关节连接到世界。|
|Break Force	|为破坏此关节而需要施加的力。|
|Break Torque	|为破坏此关节而需要施加的扭矩。|
|Enable Collision	|选中此复选框后，允许关节连接的连接体之间发生碰撞。|
|Enable Preprocessing	|禁用预处理有助于稳定无法满足的配置。|

## 弹簧关节（spring joint）

将两个物体以弹簧的形式绑定在一起，挤压它们会得到向外的力，拉伸它们将得到向里的力。


|属性：	|功能：|
|----|----|
|Connected Body	|包含弹簧关节的对象连接到的刚体对象。如果未指定对象，则弹簧将连接到空间中的固定点。|
|Anchor	|关节在对象的局部空间中所附加到的锚点。|
|Auto Configure Connected Anchor	Unity |是否应该自动计算连接锚点的位置|
|Connected Anchor	|关节在连接对象的局部空间中所附加到的锚点。|
|Spring	|弹簧的强度。|
|Damper	 |阻尼，弹簧为活性状态时的压缩程度。|
|Min Distance	|弹簧不施加任何力的距离范围的下限。|
|Max Distance	|弹簧不施加任何力的距离范围的上限。|
|Tolerance	|更改容错。允许弹簧具有不同的静止长度。|
|Break Force	|为破坏此关节而需要施加的力。|
|Break Torque	|为破坏此关节而需要施加的扭矩。|
|Enable Collision	|是否应启用两个连接对象之间的相互碰撞|
|Enable Preprocessing	|禁用预处理有助于稳定无法满足的配置。|

弹簧就像一块弹性物，试图将两个锚点一起拉到完全相同的位置。拉力的强度与两个点之间的当前距离成比例，其中每单位距离的力由 Spring 属性设定。为了防止弹簧无休止振荡，可以设置 Damper 值，从而根据与两个对象之间的相对速度按比例减小弹簧力。值越高，振荡消失的速度越快。

可以手动设置锚点，但如果启用 __Auto Configure Connected Anchor__ ，Unity 将自动设置连接锚点，以保持它们之间的初始距离（即，在定位对象时在 Scene 视图中设置的距离）。

**Min Distance 和 Max Distance 值用于设置弹簧不施加任何力的距离范围**。例如，可以使用该距离范围允许对象进行少量的独立移动，但当对象之间的距离太大时将它们拉到一起。

## 链条关节（hinge joint）

将两个物体以链条的形式绑在一起，当力量大于链条的固定力矩时，两个物体就会产生相互的拉力。

|属性：	|功能：|
|----|----|
|Connected Body	|对关节所依赖的刚体的引用（可选）。如果未设置，则关节连接到世界。|
|Anchor	|连接体围绕摆动的轴位置。该位置在局部空间中定义。|
|Axis	|连接体围绕摆动的轴方向。该方向在局部空间中定义。|
|Auto Configure Connected Anchor	|如果启用此属性，则会自动计算连接锚点 (Connected Anchor) 位置以便与锚点属性的全局位置匹配。这是默认行为。如果禁用此属性，则可以手动配置连接锚点的位置。|
|Connected Anchor	|手动配置连接锚点位置。|
|Use Spring	|弹簧使刚体相对于其连接体呈现特定角度。|
|Spring	|在启用 Use Spring 的情况下使用的弹簧的属性。|
|-        Spring	|对象声称移动到位时施加的力。|
|-       Damper	|此值越高，对象减速越快。|
|-        Target Position	|弹簧的目标角度。弹簧朝着该角度拉伸（以度为单位）。|
|Use Motor	|电机使对象旋转。|
|Motor	|在启用 Use Motor 的情况下使用的电机的属性。|
|-        Target Velocity	|对象试图获得的速度。|
|-        Force	|为获得该速度而施加的力。|
|-        Free Spin	|如果启用此属性，则绝不会使用电机来制动旋转，只会进行加速。|
|Use Limits	|如果启用此属性，则铰链的角度将被限制在 Min 到 Max 值范围内。|
|Limits |  在启用 Use Limits 的情况下使用的限制的属性。|
|-        Min	|旋转可以达到的最小角度。|
|-        Max	|旋转可以达到的最大角度。|
|-        Bounciness	|当对象达到了最小或最大停止限制时对象的反弹力大小。|
|-        Contact Distance	|在距离极限位置的接触距离内，接触将持续存在以免发生抖动。|
|Break Force	|为破坏此关节而需要施加的力。|
|Break Torque	|为破坏此关节而需要施加的扭矩。|
|Enable Collision	|选中此复选框后，允许关节连接的连接体之间发生碰撞。|
|Enable Preprocessing	|禁用预处理有助于稳定无法满足的配置。|

铰链将在 Anchor 属性指定的位置旋转，并围绕指定的 Axis 属性移动。不需要为关节的 Connected Body 属性分配游戏对象。**仅当希望关节的 _变换_ 依赖于附加对象的变换时，才应为 Connected Body 属性分配游戏对象**

可以将多个铰链关节串在一起以形成链条。为链条中的每个链接添加一个关节，并将下一个链接作为 Connected Body 附加。

**Spring、Motor 和 Limits 属性可用于微调关节的行为。Spring 和 Motor 是互斥的。同时使用这两者会导致不可预测的结果。**

## 角色关节（character joint）

可以模拟角色的骨骼关节。

## 可配置关节（configurable joint）

可以模拟任意关节的效果。

# 尝试1：实现弹弓效果
