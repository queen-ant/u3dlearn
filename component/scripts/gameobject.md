创建和销毁游戏对象
======
在 Unity 中，可以使用 Instantiate 函数来创建游戏对象。该函数可以生成现有对象的新副本：
```
public GameObject enemy;

void Start() {
    for (int i = 0; i < 5; i++) {
        Instantiate(enemy);
    }
} 
```
请注意，进行复制的源对象不必存在于场景中。更常见的做法是在 Editor 中使用从 Project 面板拖动到公共变量的预制件。**此外，实例化游戏对象将复制原始对象中存在的所有组件**。

此外，还有一个 Destroy 函数。该函数将在帧更新完成后或选择在短时间延迟后销毁对象：
```
void OnCollisionEnter(Collision otherObj) {
    if (otherObj.gameObject.tag == "Missile") {
        Destroy(gameObject,.5f);
    }
}
```

请注意，**Destroy 函数可以在不影响游戏对象本身的情况下销毁个别组件**。一个常见的错误就是编写如下代码：
```
 Destroy(this);
```
**这种情况下实际只会销毁调用该函数的脚本组件，而不是销毁附加脚本的游戏对象**。
