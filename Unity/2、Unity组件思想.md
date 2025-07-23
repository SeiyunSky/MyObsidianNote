### 预制体

### Unity生命周期函数
组件从激活到摧毁的过程

**Awake()**  唤醒事件，**一开始就执行，只执行一次**
**Start()** 开始事件，执行一次

**Update()** 更新事件，每帧执行一次
FixedUpdate() 固定更新事件，每0.02秒执行一次，所有物理相关的都在此事件进行
LateUpdate() 稍后更新事件，在Update执行后执行一次

OnEnable() 启用事件，每次启用都执行一次，当脚本组件被启用时执行一次
OnDisable() 禁用事件，每次禁用执行一次，在OnDestrory()事件也会执行
**OnDestory()** 销毁事件，组件销毁时执行

Awake() → OnEnable() → Start()

### Invoke

`Invoke(String methodName，float time) `
输入一个方法名称，过几秒执行

`InvokeRepeating(String methodName, float time, float repeatRate)`
重复调用

`CancelInvoke(String methodName)`
取消调用，无参即取消全部

### 协程
协同程序，而非多线程
```Csharp
协程不会因为前面的return，而不执行后面的代码
public IEnumerator 名字(){
   做一些事情
   yield return new WaitForSeconds(1f);
   做一些事情
}
```
使用协程函数
```Csharp
Coroutine StartCoroutine(方法(方法参数))
```

```Csharp
利用协程做动画
public IEnumerator Demo(){
  while(true){
      yield return new WaitForSeconds();
      transform.Rotate(5,0,0);
  }
}
```

主动停止协程：
```Csharp
StopAllCoroutines() 结束全部协程
StopCoroutine   结束某个协程
    (IEnumerator routine)  调用协程一样用
    (String methodname)  写入协程方法名
    (Coroutine routine)  参数写协程变量
```

## 常用工具类
**数学类**
- Random.Range(范围,范围) 返回随机值
如果用int重载，右边范围不包含，如果是float重载，那么会包含全部范围

**时间类**
- Time.time  游戏运行到现在的时间，暂停不记录
- Time.deltatime  从上一帧到当前帧的时间
- Time.realtimeSinceStartup  现实时间

- Time.timeScale 时间缩放