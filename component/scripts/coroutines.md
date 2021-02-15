协程
======

协程就像一个函数，能够暂停执行并将控制权返还给 Unity，然后在下一帧继续执行。在 C# 中，声明协程的方式如下：
```
IEnumerator Fade() 
{
    for (float f = 1f; f >= 0; f -= 0.1f) 
    {
        Color c = renderer.material.color;
        c.a = f;
        renderer.material.color = c;
        yield return null;
    }
}
```
此协程本质上是一个用返回类型 IEnumerator 声明的函数，并在主体中的某个位置包含 yield return 语句。**yield return 行是暂停执行并随后在下一帧恢复的点**。

要将协程设置为运行状态，必须使用 StartCoroutine 函数：
```
void Update()
{
    if (Input.GetKeyDown("f")) 
    {
        StartCoroutine("Fade");
    }
}
```

您会注意到 Fade 函数中的循环计数器能够在协程的生命周期内保持正确值。实际上，在 yield 语句之间可以正确保留任何变量或参数。

**默认情况下，协程将在执行 yield 后的帧上恢复，但也可以使用 WaitForSeconds 来引入时间延迟**：
```
IEnumerator Fade() 
{
    for (float f = 1f; f >= 0; f -= 0.1f) 
    {
        Color c = renderer.material.color;
        c.a = f;
        renderer.material.color = c;
        yield return new WaitForSeconds(.1f);
    }
}
```
可通过这样的协程在一段时间内传播效果，但也可将其作为一种有用的优化。

游戏中的许多任务需要定期执行，最容易想到的方法是将任务包含在 Update 函数中。但是，通常情况下，每秒将多次调用该函数。

**不需要以这样的频繁程度重复任务时，可以将其放在协程中来进行定期更新，而不是每一帧都更新**。这方面的一个示例可能是在附近有敌人时向玩家发出的警报。此代码可能如下所示：
```
function ProximityCheck() 
{
    for (int i = 0; i < enemies.Length; i++)
    {
        if (Vector3.Distance(transform.position, enemies[i].transform.position) < dangerDistance) {
                return true;
        }
    }
    
    return false;
}
```

如果有很多敌人，那么每帧都调用此函数可能会带来很大开销。但是，可以使用协程，每十分之一秒调用一次：
```
IEnumerator DoCheck() 
{
    for(;;) 
    {
        ProximityCheck;
        yield return new WaitForSeconds(.1f);
    }
}
```
这将大大减少所进行的检查次数，而不会对游戏运行过程产生任何明显影响。

**注意：禁用 MonoBehaviour 时，不会停止协程，仅在明确销毁 MonoBehaviour 时才会停止协程。
可以使用 MonoBehaviour.StopCoroutine 和 MonoBehaviour.StopAllCoroutines 来停止协程。
销毁 MonoBehaviour 时，也会停止协程。**
