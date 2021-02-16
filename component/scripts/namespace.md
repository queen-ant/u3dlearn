命名空间
======
随着项目变得越来越大并且脚本数量增加，**脚本类名称之间发生冲突的可能性也越来越大**。当几个程序员分别处理游戏的不同方面并最终将他们的工作结合在一个项目中时，尤其如此。

例如，一个程序员可能正在编写代码来控制主要玩家角色，而另一个程序员则为敌人编写等效的代码。两个程序员都可能选择调用他们的主脚本类 _Controller_，但这样会在他们的项目组合在一起时引起冲突。

在某种程度上，可通过采用命名约定或通过在发现冲突时对类重命名来避免此问题（例如，上面的类可命名为类似 PlayerController 和 EnemyController 的名称）。但是，当存在多个具有冲突名称的类或当使用这些名称来声明变量时，这很麻烦：必须替换每次出现的旧类名才能编译代码。

C# 语言提供了称为命名空间的功能，以一种可靠的方式解决了该问题。命名空间就是类的集合；引用集合中的类时需要在类名中使用所选的前缀。在下面的示例中，类 Controller1 和 Controller2 是名为 Enemy 的命名空间的成员：
````
namespace Enemy {
    public class Controller1 : MonoBehaviour {
        ...
    }
    
    public class Controller2 : MonoBehaviour {
        ...
    }
}
```
在代码中，这些类分别以 Enemy.Controller1 和 Enemy.Controller2 的形式引用。就目前而言，这种解决方案比重命名类更好，因为命名空间声明可在现有类声明外层括起来（即，没有必要单独更改所有类的名称）。此外，无论类在任何位置，均可使用多个带括号的命名空间部分将类包裹起来，即使这些类位于不同的源文件中也是如此。

通过在文件顶部添加 using 指令可避免重复输入命名空间前缀。

 using Enemy;
此行表示，在出现类名 Controller1 和 Controller2 的位置，它们应分别表示 Enemy.Controller1 和 Enemy.Controller2。

如果脚本还需要引用不同命名空间中同名的类（比如说名为 Player 的命名空间），那么仍然可以使用前缀。**如果同时使用 using 指令导入两个包含冲突类名的命名空间，则编译器将报错**。
