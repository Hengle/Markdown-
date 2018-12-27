>大量相同物体进行渲染时采用ECS可以节省大量DrawCall以达到节约性能的目的
## 首先我们需要Unity 2018.X以上版本
在Window -> Package Manager 中选择Advanced -> Show Preview Packages，然后选择Entities并安装以使用ECS。

## 创建一个脚本并命名Bootstrap
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.Entities;
 
public class Bootstrap
{
    [RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.BeforeSceneLoad)]
    public static void Awake()
    {
 
    }
 
    [RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.AfterSceneLoad)]
    public static void Start()
    {
 
    }
}
```
ECS不同于以往的Monobehavior，有一套自己的生命周期。自己编写的Awake和Start函数打上一个特性就会被Unity在场景加载的前后时机进行调用。 **（方法必须为静态方法）**
   

## 实体物体添加组件：加上Mesh Instance Renderer Component（渲染组件）。
在Awake中创建出EntityManager与EntityArchetype,通过World创建出EntityManager，EntityManager的对象提供了创建实体，给实体添加组件，获取组件，移除组件，实例化与销毁实体等功能。所以EntityManager就是一个实体的管理器。
>按照Unity的说法，默认情况下会在进入播放模式时创建好World，因此我们直接在Awake使用World创建EntityManager就好了。

```csharp
private static EntityManager entityManager;     //所有实体的管理器, 提供操作Entity的API
 
private static EntityArchetype playerArchetype; //Entity原型, 可以看成由组件组成的数组

[RuntimeInitializeOnLoadMethod(loadType: RuntimeInitializeLoadType.BeforeSceneLoad)]
public static void Awake()
{
    entityManager = World.Active.GetOrCreateManager<EntityManager>();

    //下面的的Position类型需要引入Unity.Transforms命名空间
    playerArchetype = entityManager.CreateArchetype(typeof(Position));
}

[RuntimeInitializeOnLoadMethod(loadType: RuntimeInitializeLoadType.AfterSceneLoad)]
public static void Start()
{
    //把GameObect.Find放在这里因为场景加载完成前无法获取游戏物体。
    GameObject playerGo = GameObject.Find("Player");

    //下面的类型是一个Struct, 需要引入Unity.Rendering命名空间(可以看成是组件数据，众多组件数据组成的数组形成原型数据展现原型，数据组成是共享的，因此共用drawcall达到节约性能)
    MeshInstanceRenderer playerRenderer = playerGo.GetComponent<MeshInstanceRendererComponent>().Value;

    //获取到渲染数据后可以销毁空物体
    Object.Destroy(playerGo); 

    Entity player = entityManager.CreateEntity(playerArchetype);

    //修改实体的Position组件
    entityManager.SetComponentData(player, new Position { Value = new Unity.Mathematics.float3(0, 2, 0) });

    // 向实体添加共享数据组件
    entityManager.AddSharedComponentData(player, playerRenderer);
}
```
>1.float3 是Unity新推出的数学库(Unity.Mathematics)中的类型，用法跟Vector3基本一致。Unity建议在ECS中使用该数学库。
>2.关于ISharedComponentData接口：他的作用是当实体拥有属性时，比如：球形的实体共享同样的Mesh网格数据时。这些数据会储存在一个Chuck中并非每个实体上，因此在每个实体上可以实现0内存开销。   

值得注意的是：如果Entity不包含Position组件，这个实体是不会被Unity的渲染系统关注的。因此想在屏幕上看见这个实体，必须确保MeshInstanceRenderer跟Position都添加到了实体上。
现在在Hierarchy中并没有这个实体的信息,因为Unity编辑器还没有与ECS整合，因此我们需要打开Window -> Analysis -> Entity Debugger面板查看我们的系统与实体。

## 组件+系统:实现逻辑控制
创建PlayerComponent，并在player创建出来后添加组件：
```csharp
using Unity.Entities;
 
//组件必须是struct并且得继承IComponentData接口
public struct PlayerComponent : IComponentData
{
}
```
`entityManager.AddComponentData(player, new PlayerComponent()); //添加PlayerComponent组件`
现在我们创建MovementSystem，继承自ComponentSystem：类似继承Monobehavior，我们的系统只要继承了这个基类就会被Unity识别，并且每一帧都调用OnUpdate。
```csharp
using UnityEngine;
using Unity.Entities;
using Unity.Transforms;
using Unity.Mathematics;
 
public class MovementSystem : ComponentSystem
{
    //这里声明一个结构, 其中包含我们定义的过滤条件, 也就是必须拥有CameraComponent组件才会被注入。
    public struct Group
    {
        public readonly int Length;
 
        public ComponentDataArray<Position> Positions;
    }
 
    //然后声明结构类型的字段, 并且加上[Inject]
    [Inject] Group data;

    protected override void OnUpdate()
    {
        float deltaTime = Time.deltaTime;
 
        for (int i = 0; i < data.Length; i++)
        {
            float3 up = new float3(0, 1, 0);
 
            float3 pos = data.Positions.Value; //Read
 
            pos += up * deltaTime;
 
            data.Positions = new Position { Value = pos }; //Write
        }
    }
}
```

## 结合之前项目
如果要跟以前的项目结合，我们还需要能够访问场景中的游戏物体，比如一个经典的Cube。在Bootstrap.Start中获取我们的Cube，并且加上GameObjectEntity组件。GameObjectEntity 确实叫这个名, 是Unity提供的组件。添加上这个组件后Cube就可以被entityManager关注，并且可以获取Cube上的任意组件：
```csharp
//获取Cube        
GameObjectEntity cubeEntity = GameObject.Find("Cube").AddComponent<GameObjectEntity>();

//添加Velocity组件
entityManager.AddComponentData(cubeEntity.Entity, new VelocityComponent { moveDir = new Unity.Mathematics.float3(0, 1, 0) });
```