访问组件
=====
组件实际上是类的实例，因此第一步是获取对需要使用的组件实例的引用。这是通过 GetComponent 函数来完成的。通常，希望将组件对象分配给变量，而此操作是使用以下语法以 C# 实现的：
```
void Start () 
{
    Rigidbody rb = GetComponent<Rigidbody>();
}
```
获得对组件实例的引用后，可以像在 Inspector 中一样设置其属性的值：
```
void Start () 
{
    Rigidbody rb = GetComponent<Rigidbody>();
    
    // 改变对象的刚体质量。
    rb.mass = 10f;
}
```
Inspector 中所没有的一项额外功能是可以在组件实例上调用函数：
```
void Start ()
{
    Rigidbody rb = GetComponent<Rigidbody>();
    
    // 向刚体添加作用力。
    rb.AddForce(Vector3.up * 10f);
}
```
另外请注意，完全可以将多个自定义脚本附加到同一对象。**如果需要从一个脚本访问另一个脚本，可以像往常一样使用 GetComponent，只需使用脚本类的名称（或文件名）来指定所需的组件类型**。

**如果尝试检索尚未实际添加到游戏对象的组件，则 GetComponent 将返回 null；如果尝试更改 null 对象上的任何值，将在运行时出现 null 引用错误**。

访问其他对象
======
Unity 提供了许多不同方法来检索其他对象，每种方法都适合特定情况。

将游戏对象与变量链接
------
查找相关游戏对象的最直接方法是向脚本添加公共的游戏对象变量：
```
public class Enemy : MonoBehaviour
{
    public GameObject player;
    
    // 其他变量和函数...
}
```
此变量在 Inspector 中可以像任何其他变量一样显示：
<img src="https://docs.unity3d.com/cn/2018.4/uploads/Main/GameObjectPublicVar.png">

现在可以将对象从场景或 Hierarchy 面板拖到此变量上，对其进行分配。GetComponent 函数和组件访问变量与其他任何变量一样可用于此对象，因此可以使用如下代码：
```
public class Enemy : MonoBehaviour {
    public GameObject player;
    
    void Start() {
        // 在玩家角色背后十个单位的位置生成敌人。
        transform.position = player.transform.position - Vector3.forward * 10f;
    }
}
```
此外，**如果在脚本中声明组件类型的公共变量，则可以拖动已附加该组件的任何游戏对象。这将直接访问组件而不是游戏对象本身**。
```
public Transform playerTransform; 
```
**在处理具有永久连接的单个对象时，将对象与变量链接在一起是最有用的方法**。可以使用数组变量来链接同一类型的多个对象，但仍然必须在 Unity Editor 中（而不是在运行时）进行连接。

在运行时定位对象通常很方便，因此 **Unity 提供了两种基本方法来执行此操作**，如下所述。

寻找子游戏对象
------
可使一组游戏对象成为一个父游戏对象的所有子对象，这种管理多个游戏对象的方式通常会更好。可以使用父游戏对象的变换组件来检索子游戏对象（因为所有游戏对象都具有隐式变换）：
```
using UnityEngine;

public class WaypointManager : MonoBehaviour {
    public Transform[] waypoints;
    
    void Start() 
    {
        waypoints = new Transform[transform.childCount];
        int i = 0;
        
        foreach (Transform t in transform)
        {
            waypoints[i++] = t;
        }
    }
}
```
还可以使用 Transform.Find 函数按名称查找特定子对象： `transform.Find("Gun");`

当对象具有可以在游戏运行过程中添加和删除的子对象时，这种功能可能很有用。可以拾取和放下的武器就是这方面的一个很好的例子。

按名称或标签来查找游戏对象
只要有某种信息可以识别游戏对象，就可以在场景层级视图中的任何位置找到该游戏对象。可使用 GameObject.Find 函数按名称检索各个对象：
```
GameObject player;

void Start() 
{
    player = GameObject.Find("MainHeroCharacter");
}
```
还可以使用 GameObject.FindWithTag 和 GameObject.FindGameObjectsWithTag 函数按标签来查找对象或者对象集合：
```
GameObject player;
GameObject[] enemies;

void Start() 
{
    player = GameObject.FindWithTag("Player");
    enemies = GameObject.FindGameObjectsWithTag("Enemy");
}
```
