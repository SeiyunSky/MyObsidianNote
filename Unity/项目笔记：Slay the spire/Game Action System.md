**GameAction类**
基类，负责管理动作列表
以前中后为管理逻辑
```c#
public abstract class GameAction
{
    public List<GameAction> PreReactions{get ; private set;} = new();
    public List<GameAction> PerformReactions{get ; private set;} = new();
    public List<GameAction> PostReactions{get ; private set;} = new();
    
}
```

**ActionSystem类**
```c#
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ActionSystem : Singleton<ActionSystem>
{
    //等待处理的动作
    private List<GameAction> reactions = null;
    
    //表示是否有动作正在执行
    public bool IsPerforming { get; private set; } = false;
    
    //所有订阅了game-preaction的对象，在动作执行前触发的回调（如检查条件、播放动画）。
    private static Dictionary<Type, List<Action<GameAction>>> preSubs = new();
    //所有订阅了game-postaction的对象，在动作执行后触发的回调（如更新UI、触发音效）。
    private static Dictionary<Type, List<Action<GameAction>>> postSubs = new();
    //这里存放数据，存储每种 `GameAction`对应的协程执行逻辑。
    private static Dictionary<Type, Func<GameAction, IEnumerator>> performers = new();

    public void Perform(GameAction action, System.Action OnPerformFinished = null)
    {
        if (IsPerforming) return; 
        IsPerforming = true; 
        StartCoroutine(Flow(action, () => 
        { 
            IsPerforming = false; 
            OnPerformFinished?.Invoke(); 
        }));
    }
    
    //外部类向当前类实时添加需要处理的action
    //可以让我们在处理过程中，外部委托调用增加机制
    public void AddReaction(GameAction gameAction){
        reactions?.Add(gameAction);
    }
    
    //内部处理功能
    private IEnumerator Flow(GameAction action, Action OnFlowFinished = null)
    {
        // 1. 预处理阶段（Pre-reactions）
        reactions = action.PreReactions;
        PerformSubscribers(action, preSubs);
        yield return PerformReactions();
    
        // 2. 主要执行阶段（Perform）
        reactions = action.PerformReactions;
        yield return PerformPerformer(action);
        yield return PerformReactions();
    
        // 3. 后处理阶段（Post-reactions）
        reactions = action.PostReactions;
        PerformSubscribers(action, postSubs);
        yield return PerformReactions();
    
        // 4. 完成回调
        OnFlowFinished?.Invoke();
    }
    
    //执行特定类型动作的所有订阅者
    private void PerformSubscribers(GameAction action, Dictionary<Type, List<Action<GameAction>>> subs)
    {
        Type type = action.GetType();
        if (subs.ContainsKey(type))
        {
            foreach (var sub in subs[type])
            {
            //Action是个委托，这里的subs是一个存满了委托的列表
            //委托类型是GameAction
            //我们将需要处理的action传过来以后，调用委托调用
            //而委托在这里就涉及到方法的具体处理
                sub(action);
            }
        }
    }
    
    //动作的具体执行
    private IEnumerator PerformReactions()
    {
        foreach (var reaction in reactions)
        {
            yield return Flow(reaction);
        }
    }
    
    //
    private IEnumerator PerformPerformer(GameAction action)
    {
        Type type = action.GetType();
        if (performers.ContainsKey(type))
        {
            yield return performers[type](action);
        }
    }
    
    
    //实际的去向字典内部添加方法委托
    public static void AttachPerformer<T>(Func<T, IEnumerator> performer) where T : GameAction
    {
        //获取具体类型
        Type type = typeof(T);
        
        //创建包装方法转换GameAction为具体类型
        IEnumerator wrappedPerformer(GameAction action) => performer((T)action);
        
        //存储进字典
        if (performers.ContainsKey(type)) 
            performers[type] = wrappedPerformer;
        else 
            performers.Add(type, wrappedPerformer);
    }

    //移除方法委托
    public static void DetachPerformer<T>() where T : GameAction
    {
        Type type = typeof(T);
        if (performers.ContainsKey(type)) 
            performers.Remove(type);
    }
    
    //外部订阅特定类型游戏动作的前置或后置反应
    public static void SubscribeReaction<T>(Action<T> reaction, ReactionTiming timing) where T : GameAction 
    {
        Dictionary<Type, List<Action<GameAction>>> subs = timing == ReactionTiming.PRE ? preSubs : postSubs;
    
        void wrappedReaction(GameAction action) => reaction((T)action);
    
        if (subs.ContainsKey(typeof(T)))
        {
            subs[typeof(T)].Add(wrappedReaction);
        }
        else
        {
        //等价于 subs.get(typeof(T)).add(wrappedReaction);
        //java里的map.put(key, value) 在c#可以直接map[key] = value
            subs.Add(typeof(T), new());
            subs[typeof(T)].Add(wrappedReaction);
        }
    }
    
    //移除特定类型游戏动作的前置或后置反应
    public static void UnsubscribeReaction<T>(Action<T> reaction, ReactionTiming timing) where T : GameAction 
    {
        Dictionary<Type, List<Action<GameAction>>> subs = timing == ReactionTiming.PRE ? preSubs : postSubs;
        if (subs.ContainsKey(typeof(T)))
        {
            void wrappedReaction(GameAction action) => reaction((T)action);
            subs[typeof(T)].Remove(wrappedReaction);
        }
    }
}
```

**ReactionTiming**枚举类
```C#
public enum ReactionTiming{
    PRE,
    POST
}
```


### 具体例子——CardSystem
**DrawCardGA**

**DealDamageGA**

**IncreaseStatsGA**


```c#
using UnityEngine;

public class Card : MonoBehaviour 
{
    void OnMouseDown() 
    { 
        if (ActionSystem.Instance.IsPerforming) return; 
        DrawCardGA drawCardGA = new(); 
        ActionSystem.Instance.Perform(drawCardGA); 
        Destroy(gameObject); 
    }
}
```

```c#
using UnityEngine;
using DG.Tweening; 
using System.Collections;

public class CardSystem : MonoBehaviour
{
    [SerializeField] private Card cardPrefab;
    [SerializeField] private Transform spawn;
    [SerializeField] private Transform hand;
    
    private void OnEnable()
    {
        ActionSystem.AttachPerformer<DrawCardGA>(DrawCardPerformer);
    }
    
    private void OnDisable()
    {
        ActionSystem.DetachPerformer<DrawCardGA>();
    }
    
    private IEnumerator DrawCardPerformer(DrawCardGA drawCardGA)
    {
        Card card = Instantiate(cardPrefab, spawn.position, Quaternion.identity);
        Tween tween = card.transform.DOMove(hand.position, 0.5f);
        yield return tween.WaitForCompletion();
    }
}
```