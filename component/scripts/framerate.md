时间和帧率管理
=======

游戏的帧率不是恒定的，因此 Update 函数调用之间的时间长度也不是恒定的。

举例来说，假设在一项任务中需要逐步向前移动某个对象，一次一帧。起初看起来好像可以在每帧将对象移动一个固定距离：
```
//C# 脚本示例
using UnityEngine;
using System.Collections;

public class ExampleScript : MonoBehaviour {
    public float distancePerFrame;
    
    void Update() {
        transform.Translate(0, 0, distancePerFrame);
    }
}
```
但是，如果帧时间不是恒定的，那么对象看起来会以不规则的速度移动。如果帧时间为 10 毫秒，那么对象将以 distancePerFrame 的距离每秒前进一百次。但如果帧时间增加到 25 毫秒（比如由于 CPU 负载的原因），那么对象每秒只会前进四十次，因此移动的总距离更短。

**解决方案是通过可从 Time.deltaTime 属性读取的帧时间来缩放移动距离大小**：
```
//C# 脚本示例
using UnityEngine;
using System.Collections;

public class ExampleScript : MonoBehaviour {
    public float distancePerSecond;
    
    void Update() {
        //在 z 方向以 distancePerSecond 单位/秒的速度移动 GameObject。将 distancePerSecond 乘以 Time.deltaTime，然后与 z 分量相加。
        transform.Translate(0, 0, distancePerSecond * Time.deltaTime);  
    }
}
```
请注意，**此移动距离现在为 distancePerSecond 而不是 distancePerFrame**。随着帧率的变化，移动步长大小也会相应改变，因此对象的速度将保持不变。

**MonoBehaviour.FixedUpdate 使用 fixedDeltaTime，而非 deltaTime**。不依赖于 MonoBehaviour.OnGUI 中的 Time.deltaTime。Unity 可以每帧多次调用 OnGUI。应用程序每次调用使用相同的 deltaTime 值。

固定时间步长
-------
与主帧更新不同，**Unity 的物理系统会工作到固定的时间步长**，这对于模拟的准确性和一致性很重要。

在物理更新开始时，Unity 通过将固定的时间步长值添加到上次物理更新结束的时间来设置“警报”时间。然后，物理系统将执行计算，直到警报响起。

可从 Time 窗口中更改固定时间步长的大小，并可使用 Time.fixedDeltaTime 属性从脚本中读取该值。

**请注意，较低的时间步长值将产生更频繁的物理更新和更精确的模拟，但代价是更大的 CPU 负载**。除非要对物理引擎提出很高的要求，否则可能不需要更改默认的固定时间步长。

最大允许步长
-------
Maximum Allowed Timestep 设置（位于 Time 窗口中）可限制 Unity 在指定的帧更新期间处理物理系统和 FixedUpdate 调用所花费的时间。

**如果帧更新花费的时间超过 Maximum Allowed Timestep 设置，则物理引擎将“让时间停止”并让帧处理赶上**。

一旦帧更新完成，物理引擎将恢复，就好像让时间停止后没有流逝一样。这种情况下的结果是刚体不会像正常情况下那样实时完美移动，而会稍微减慢。然而，物理“时钟”仍将跟踪它们，就像它们正常移动一样。物理时间的减慢通常是不明显的，并且是针对游戏性能的合理折衷。

时间标度
-------
对于特殊效果，例如“子弹时间”，有时减慢游戏时间的流逝会很有用，能够使动画和脚本响应以较低的速率发生。此外，有时可能希望完全冻结游戏时间，就像游戏暂停时一样。

Unity 有一个 Time Scale 属性可以控制游戏时间相对于实时时间的进展速度。**如果该标度设置为 1.0，则游戏时间与实时时间匹配。值为 2.0 会使 Unity 中的时间流逝速度加倍（即，动作将加速），而值为 0.5 则会将游戏速度减半。值为零将使时间完全“停止”**。

请注意，**时间标度实际上并不会降低执行速度，而只是更改了通过 Time.deltaTime 和 Time.fixedDeltaTime 报告给 Update 和 FixedUpdate 函数的时间步长**。当游戏时间减慢时，调用 Update 函数的频率可能高于平常，但每帧报告的 deltaTime 步长将会缩短。

**其他脚本函数不受时间标度的影响，因此您可以在游戏暂停时显示具有正常交互的 GUI**。

Time 窗口有一个属性可用于全局设置时间标度，但使用 Time.timeScale 属性从脚本设置该值通常更有用：
```
//C# 脚本示例
using UnityEngine;
using System.Collections;

public class ExampleScript : MonoBehaviour {
    void Pause() {
        Time.timeScale = 0;
    }
    
    void Resume() {
        Time.timeScale = 1;
    }
}
```

Capture Framerate
-------
一个非常特殊的时间管理案例是您希望将游戏过程录制为视频。由于保存屏幕图像的任务需要相当长的时间，因此如果在正常游戏过程中尝试执行此操作，则游戏的常规帧率将大幅降低。这将导致视频无法反映游戏的真实性能。

Unity 提供了一个 Capture Framerate 属性可用于解决该问题。

**该属性的值设置为零以外的任何值时，游戏时间将减慢，而帧更新将以精确的定期时间间隔发出。帧之间的时间间隔等于 1.0 / Time.captureFramerate，而与实时和渲染帧所需的时间无关**。因此如果该值设置为 5.0，则每五分之一秒更新一次。随着对帧率的要求有效降低，在 Update 函数中便有了时间保存截屏或采取其他操作：

```
//C# 脚本示例
using UnityEngine;
using System.Collections;

public class ExampleScript : MonoBehaviour {
    // 捕获帧作为截屏序列。图像
    // 作为 PNG 文件存储在文件夹中 - 这些文件可通过
    //图像实用程序软件（例如，QuickTime Pro）组合成电影。
    //该文件夹将包含我们的截屏。
    //如果该文件夹存在，我们将附加数字来创建一个空文件夹。
    string folder ="ScreenshotFolder";
    int frameRate = 25;
        
    void Start () {
        // 设置回放帧率（实时时间将与此后的游戏时间不相关）。
        Time.captureFramerate = frameRate;
        
        //创建文件夹
        System.IO.Directory.CreateDirectory(folder);
    }
    
    void Update () {
        // 将文件名附加到文件夹名（格式为"0005 shot.png""）
        string name = string.Format("{0}/{1:D04} shot.png", folder, Time.frameCount );
        
        // 将截屏捕获到指定的文件。
        Application.CaptureScreenshot(name);
    }
}
```

尽管使用这种技术录制的视频通常看起来非常好，但是当速度减慢时，游戏很难游玩。可能需要测试 Time.captureFramerate 的值来确保合适的帧率。
