# GameFramework入门记录
>官网地址：http://gameframework.cn/

## 流程 (Procedure) 
**流程 (Procedure)** – 是贯穿游戏运行时整个生命周期的有限状态机。通过流程，将不同的游戏状态进行解耦将是一个非常好的习惯。对于网络游戏，你可能需要如检查资源流程、更新资源流程、检查服务器列表流程、选择服务器流程、登录服务器流程、创建角色流程等流程，而对于单机游戏，你可能需要在游戏选择菜单流程和游戏实际玩法流程之间做切换。如果想增加流程，只要派生自 ProcedureBase 类并实现自己的流程类即可使用。
>1.设定默认加载流程，并把所有场景在到built列表。
---

## UI组件（UIComponent）
```CSHARP
// 加载框架UI组件
UIComponent UI = UnityGameFramework.Runtime.GameEntry.GetComponent<UIComponent>();
// 加载UI
UI.OpenUIForm("Assets/Demo4/UI_Menu.prefab", "DefaultGroup", this);
```
>1.需要设置好UIGroup

---
## 内置事件（EventComponent）
为了订阅事件，我们需要加载框架的EventComponent组件。
```CSHARP
EventComponent Event = UnityGameFramework.Runtime.GameEntry.GetComponent<EventComponent>();
// 订阅UI加载成功事件
Event.Subscribe(OpenUIFormSuccessEventArgs.EventId, OnOpenUIFormSuccess);
```  
订阅事件的方式很简单：Event.Subscribe(事件唯一ID, 回调函数)。
OpenUIFormSuccessEventArgs是框架自带的事件类，这个事件类处理的是”打开界面成功事件”，而它的EventId只是一个确保唯一的int类型，用于标识这是什么事件。
而回调函数，当然就是事件发生时我们希望调用的函数了。
>1.订阅事件加载UI需要挂载UIFormLogic逻辑脚本。

---

## 加载配置文件
>官方文档：http://gameframework.cn/archives/category/module/buildin/datatable

---

## 创建实体
```csharp
// 获取框架实体组件
EntityComponent entityComponent = UnityGameFramework.Runtime.GameEntry.GetComponent<EntityComponent>();
// 创建实体
entityComponent.ShowEntity<Demo6_HeroLogic>(1, "Assets/Demo6/CubeEntity.prefab", "EntityGroup");
```
实体添加实体逻辑脚本：
```csharp
public class Demo6_HeroLogic : EntityLogic
{
    protected override void OnShow (object userData) {
        base.OnShow (userData);
        Log.Debug("显示实体.");
    }
}
```

---

## Web请求
1.API列表  
![](http://www.benmutou.com/wp-content/uploads/2018/03/032618_2354_Demo7Web1.png)

WebRequest组件用于发起短链接的请求，包括Get、Post，以上几个不同的AddWebRequest函数是用于在发生数据的时候附上post数据、Form表单数据，这个看参数名就知道了。至于userData，当然是自定义的数据，在请求成功或失败时可以获取这个自定义数据。

---

## AssetBundle打包