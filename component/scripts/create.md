创建C#脚本
======
可以从 Project 面板左上方的 Create 菜单新建脚本，也可以通过从主菜单选择 Assets > Create > C# Script 来新建脚本。

该文件的初始内容将如下所示：
```
using UnityEngine;
using System.Collections;

public class MainPlayer : MonoBehaviour {

    // 使用此函数进行初始化
    void Start () {
    
    }
    
    // 每帧调用一次 Update
    void Update () {
    
    }
}
```

Update 函数是放置代码的地方，用于处理游戏对象的帧更新。

在游戏开始之前（即第一次调用 Update 函数之前），Unity 将调用 Start 函数；此函数是进行所有初始化的理想位置。

为了附加脚本，可将脚本资源拖动到层级视图面板中的游戏对象，或拖动到当前选定游戏对象的检视面板。

Debug.Log 是一个简单的命令，只是将消息输出到 Unity 的控制台。

初始变量
-----
正如其他组件通常具有可在 Inspector 中编辑的属性一样，您也可以让自己脚本中的值可在 Inspector 中编辑。

```
using UnityEngine;
using System.Collections;

public class MainPlayer : MonoBehaviour 
{
    public string myName;
    
    // 此函数用于初始化
    void Start () 
    {
        Debug.Log("I am alive and my name is " + myName);
    }
}
```

此代码在 Inspector 中创建一个标记为“My Name”的可编辑字段。

Unity 通过在变量名称中**出现大写字母的位置引入空格**来创建 Inspector 标签。**但是，这纯粹是出于显示目的，在代码中应始终使用变量名称**。

**在 C# 中，必须将变量声明为 public 才能在 Inspector 中查看该变量**。
