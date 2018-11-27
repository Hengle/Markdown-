# PlayMaker自定义action的相关事宜

@import "logo.png"
[TOC]
状态机这种东西,据说在游戏开发中属于神奇级别的设计模式

playmaker也因此一直占据unity的销售榜前三的位置

同事以前说,大概是因为开发者中非程序员占比较大,所以卖的火

最近感觉不太像是这么回事,因为确实超级方便啊.........

理清逻辑,画好关系图,连连线就完成游戏逻辑,多美妙啊

当然,还有就是需要为自己的脚本写action

因为感觉目前的pm调用自己的脚本方法可能是用的sendMessage或是反射,都是比较耗费性能的东西

## 自定义action的格式规范
----
### nameSpace命名空间,门类,描述信息
```
using UnityEngine;
namespace HutongGames.PlayMaker.Actions
{
    [ActionCategory(ActionCategory.GameObject)]
    [Tooltip("Gets a Game Object's Tag and stores it in a String Variable.")]![logo](undefined)
    public class MyPlaymakerAction:BaseLogAction
    {
    }
}
```
首先是所有action都在一个统一的命名空间下,HUtonggame.playmaker.Actions
然后是需要添加一个attributer ,第一个属性是将action分类,一个枚举值,包含大部分类别
如果希望新添加一个门类的话,[ActionCategory("MyCustomCatergory")],添加一个字符串即可
第二个attributer是tooltip,就是你的action的描述

### 变量的定义

接下来是变量的定义

playmaker的自定义action编写与monoUnity的脚本编写基本相同
只不过是将gameobject,int,string等类型换成了fsmGameObject,fsmInt,fsmString等类型即可

变量也需要定义attributer
[requiredField ]表示这是一个不能为空的字段
[UIHint()]表示用户可以用一个下拉框来选择变量(在变量面板自定义的变量),而不可以自己输入变量

action添加到一个状态上时,可以在reset函数中执行变量的初始赋值
```
public override void Reset()
{
    gameObject = null;
    storeResult = null;
    everyFrame = false;
}
```
大部分时候null即可

PS:访问添加action的gameobeject的话,变量为owner,例:owner.transform.rotate();

### action的生命周期
OnEnter
OnUpdate
OnExit
Reset

OnEnter是进入时执行的
Reset是给状态机添加action时候,可以初始赋值用,也负责将action的状态回归用

### action事件

如果需要触发onTrigger,onCollision 事件的话

Fsm.HandleOnTriggerEnter = true

即可

### Action Attributes
You can add Attributes to actions to customize their appearance in the Playmaker editor.

|Attribute|Description|
|:--:|:--|
ActionCategory|Defines the action's category in the Action Browser.<br>Use either the built-in ActionCategory enum or a string value to define a custom category.<br>C# Example:<br>```[ActionCategory(ActionCategory.Animation)]  public class MyAnimationAction : FsmStateAction ``` ```[ActionCategory("Custom")]public class MyCustomAction : FsmStateAction``` 
Tooltip|Defines rollover hints. Use on a class or any public field.<br>C# Example:<br>[Tooltip("This is the best action ever!")]public class MyCustomAction : FsmStateAction|
RequiredField|Use on any public field that the user must define. If left undefined the editor will show an error.<br>C# Example:```[RequiredField]public FsmBool importantOption;|```
CheckForComponent|Use on a public GameObject, FsmGameObject or FsmOwnerDefault field to check that the specified game object has the required components. Otherwise an error will show under the field.<br>NOTE, you can specify up to 3 component types to check for.<br>C# Example:<br>```[CheckForComponent(typeof(Animation))]public FsmGameObject gameObjectToAnimate;```|
HasFloatSlider|Use on a public float field to show a slider. Note: You specify min/max values for the slider, but you can still enter values outside that range in the edit field. This is by design...<br>C# Example:<br>```[HasFloatSlider(0, 100)]public FsmFloat health;```
UIHint|Use on a public field to help Playmaker show the most appropriate editor for the type of data.<br>See the table below for details.

### UIHint
Use a UIHint attribute on a public field to help Playmaker show the most appropriate editor for the type of data.

Hint|Description
:---:|:---|
Animation|Use on a string field for an animation browser button.
Behaviour |Use on a string field for a Behavior popup button. The popup will attempt to find behaviors on the previously specified game object.
Coroutine |Use on a string field for a Coroutine popup button. The popup will attempt to find Coroutine methods on the previously specified game object.
Description|Use on a string field to format the text in a large readonly info box.
Layer|Use on an integer field for a Layer popup button.
Script|Use on a string field for a script popup button. The popup will attempt to find scripts on the previously specified game object.
Tag|Use on a string field for a Tag popup button.
TextArea|Use on a string field for a larger text edit field.
Variable|Use on an Fsm Variable field (FsmFloat, FsmInt, FsmBool...) to show a variable popup button used to select a variable of that type.
 <meta http-equiv="refresh" content="1">