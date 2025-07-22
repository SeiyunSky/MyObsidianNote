# C\#设计模式专题
## 委托
将方法作为函数来进行传递，此类型可以赋值一个方法的引用
```Csharp
//声明委托
delegate void MyDelegate()

//使用委托
MyDelegate test = new Mydelegate(目标函数)
MyDelegate test1 = 目标函数
```

## ACTION委托
Action\<T\>是自带的泛型委托，可以用其以参数形式传递方法，而非显示声明自定义委托。封装的方法必须与此委托定义的方法签名相对应，即必须有一个通过值传递给它的参数。且只能用Void类型
```Csharp
Action action1 = Show1;
action1();

Action<int,int> action2 = Show2;
action2(1,1);
```

## Func委托
内置带有返回了类型的泛型委托，其中最后一个类型，为返回值。
```Csharp
Func<String> func1 = show1;

string Show(){
    return "Show1"
}
```

## 匿名方法、Event事件、多播委托
```Csharp
//匿名方法写法
Action action = delegate(){
    方法内容;
}

//多播委托
Action action =Show1;
action+=Show2;
action-=Show1;
//也就是委托可以身兼数值
//委托的引用类型，默认为null，使用前需要判断是否为空

//Event事件
//只允许作为【类的成员变量】，在类的内部使用才可以，外部不可直接调用
event Action action;

action = Show1;
action();
```

## 单例模式
单例模式即所谓的一个类仅仅一个实例，类只在内部实例一次，然后提供该实例
1、全局唯一。2、只能自己创建自己的唯一实例

```

```