#  [Unity 3D游戏开发（第2版）](http://www.ituring.com.cn/book/2154)
[TOC]
##                         第3章　拓展编辑器　

上一章中，我们学习了Unity的5大视图（Project视图、Hierarchy视图、Inspector视图、Scene视图和Game视图），它们都有一套自己的编辑布局，以及相互协作的工作方式。其实Unity还内置了很多工具视图，后面会陆续介绍。Unity内置的编辑器做得再好，其实也满足不了多变的开发需求，不过Unity提供了灵活多变的编辑器拓展API接口，通过代码反射，可以修改一些系统自带的编辑器窗口。此外，丰富的EditorGUI接口也可以拓展出各式各样的编辑器窗口。学好本章，可以为日后开发专属的游戏编辑器打下良好的基础。

## 3.1　拓展Project视图

在Project视图下，存放着大量游戏资源，资源之间的依赖关系非常复杂，有效地管理这些资源尤其重要。默认的布局比较简单，按照文件夹为单位依次显示，文件夹下可以嵌套子文件夹，我们可以根据资源类型归类存放。右键弹出的菜单项的功能比较基础，但是可以满足绝大多数的需求。本章中，我们将学习如何拓展它，让菜单项更加丰富。

### 3.1.1　拓展右键菜单

在Project视图中，点击鼠标右键会弹出视图菜单，如图3-1所示。菜单的基础功能包括资源的创建、打开、删除、导入、导出、查找引用、刷新和重新导入等。其中，删除、打开和重新导入等操作需要在Project视图中选中一个或多个资源。选中某个资源时，该资源名称上将会出现蓝色的矩形框，此时右键弹出菜单即可针对选中的资源来处理。下面首先来学习如何拓展这个菜单。

编辑器使用的代码应该仅限于编辑模式下，也就是说正式的游戏包不应该包含这些代码。Unity提供了一个规则：如果属于编辑模式下的代码，需要放在Editor文件夹下；如果属于运行时执行的代码，放在任意非Editor文件夹下即可。如图3-2所示，将编辑代码放在Editor文件夹下。这里需要说明的是， Editor文件夹的位置比较灵活，它还可以作为多个目录的子文件夹存在，这样开发者就可以按功能来划分，将不同功能的编辑代码放在不同的Editor目录下。例如可以有多个Editor目录，它们各自处理各自的逻辑。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=180978b0666f72296d82)

接着，看一下拓展鼠标右键的窗口菜单，如图3-3所示。先用鼠标左键选中Scene游戏资源，此时点击鼠标右键，此时将弹出系统菜单。这里出现了我们拓展的My Tools菜单集合，右侧出现了Tools 2和Tools 1菜单条，点击其中任意一个菜单条，程序将会自动在Console窗口中打印它的名称。本示例的代码如代码清单3-1所示。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=1809272a54eef545d246)

代码清单3-1　Script_03_01.cs文件

```
using UnityEngine;
using UnityEditor;

public class Script_03_01
{
    [MenuItem("Assets/My Tools/Tools 1",false,2)]
    static void MyTools1()
    {
        Debug.Log(Selection.activeObject.name);
    }
    [MenuItem("Assets/My Tools/Tools 2",false,1)]
    static void MyTools2()
    {
        Debug.Log(Selection.activeObject.name);
    }
}
```

自定义菜单的参数需要在MenuItem方法中写入显示的菜单路径。如果菜单条比较多，可以在第三个参数处输入表示排序的整数，数值越小，它的排序就越靠前。最后使用Debug.Log打印选择的游戏对象Selection.activeObject.name即可。

### 3.1.2　创建菜单

在Project视图中，点击左上方的Create按钮，可以弹出资源创建菜单，如图3-4所示。在最上面拓展了My Create菜单，点击Sphere和Cube菜单条，程序将创建球体和立方体游戏对象。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=1809af8b097e2aa7a8ff)

菜单条的排序方法和上一节介绍的完全一样，设置MenuItem方法的第三个参数即可。如代码清单3-2所示，重写MenuItem方法中的路径即可拓展菜单项。

代码清单3-2　Script_03_02.cs文件

```
using UnityEngine;
using UnityEditor;

public class Script_03_02
{
    [MenuItem("Assets/Create/My Create/Cube",false,2)]
    static void CreateCube()
    {
        GameObject.CreatePrimitive(PrimitiveType.Cube); //创建立方体
    }
    [MenuItem("Assets/Create/My Create/Sphere",false,1)]
    static void CreateSphere()
    {
        GameObject.CreatePrimitive(PrimitiveType.Sphere);//创建球体
    }
}
```

通过观察可以发现，拓展菜单的关键就是找到正确的菜单路径，通过“/”符号将它们拼合而成即可。代码中的GameObject.CreatePrimitive()方法用于创建Unity基础模型体。

### 3.1.3　拓展布局

如图3-5所示，当用鼠标选中一个资源后，右边将出现拓展后的click按钮，点击这个按钮，程序会自动在Console窗口中打印选中的资源名。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=180935511818eeb496da)

如代码清单3-3所示，在Project视图代码的右侧拓展自定义按钮，在代码中既可设置拓展按钮的区域，也可监听按钮的点击事件。

代码清单3-3　Script_03_03.cs文件

```
using UnityEngine;
using UnityEditor;

public class Script_03_03
{
    [InitializeOnLoadMethod]
    static void InitializeOnLoadMethod()
    {
        EditorApplication.projectWindowItemOnGUI = delegate(string guid,
            Rect selectionRect) {
            //在Project视图中选择一个资源
            if(Selection.activeObject &&
                guid == AssetDatabase.AssetPathToGUID(AssetDatabase.GetAssetPath
                    (Selection.activeObject))){
                //设置拓展按钮区域
                float width=50f;
                selectionRect.x +=(selectionRect.width - width);
                selectionRect.y +=2f;
                selectionRect.width = width;
                GUI.color = Color.red;
                //点击事件
                if(GUI.Button(selectionRect,"click")){
                    Debug.LogFormat("click : {0}",Selection.activeObject.name);
                }
                GUI.color = Color.white;
            }
        };
    }
}
```

需要说明的是，在方法前面添加[InitializeOnLoadMethod]表示此方法会在C# 代码每次编译完成后首先调用。监听EditorApplication.projectWindowItemOnGUI委托，即可使用GUI方法来绘制自定义的UI元素。这里我们添加了一个按钮。此外，GUI还提供了丰富的元素接口，可以用来添加文本、图片、滚动条和下拉框等复杂元素。

### 3.1.4　监听事件

Project视图中的资源比较多，如果不好好规划，资源就会很凌乱。有时候，我们可能需要借助程序来约束资源，这可以通过监听资源的创建、删除、移动和保存等事件来实现。例如，将某个文件移动到错误的目录下，此时就可以监听资源移动事件，程序判断资源的原始位置以及将要移动的位置是否合法，从而决定是否能阻止本次移动。Unity提供了监听的基类。

如代码清单3-4所示，首先需要继承UnityEditor.AssetModificationProcessor，接着重写监听资源创建、删除、移动和保存的方法，处理自己的特殊逻辑。

代码清单3-4　Script_03_04.cs文件

```
using UnityEngine;
using UnityEditor;
using System.Collections.Generic;

public class Script_03_04 : UnityEditor.AssetModificationProcessor
{
    [InitializeOnLoadMethod]
    static void InitializeOnLoadMethod()
    {
        //全局监听Project视图下的资源是否发生变化（添加、删除和移动）
        EditorApplication.projectWindowChanged = delegate() {
            Debug.Log("change");
        };
    }
    //监听“双击鼠标左键，打开资源”事件
    public static bool IsOpenForEdit(string assetPath, out string message)
    {
        message = null;
        Debug.LogFormat("assetPath : {0} ", assetPath);
        //true表示该资源可以打开，false表示不允许在Unity中打开该资源
        return true;
    }
    //监听“资源即将被创建”事件
    public static void OnWillCreateAsset(string path)
    {
        Debug.LogFormat("path : {0}", path);
    }
    //监听“资源即将被保存”事件
    public static string[] OnWillSaveAssets(string[] paths)
    {
        if(paths != null) {
            Debug.LogFormat("path : {0}",    string.Join(",",paths));
        }
        return paths;
    }
    //监听“资源即将被移动”事件
    public static AssetMoveResult OnWillMoveAsset(string oldPath, string newPath)
    {
        Debug.LogFormat("from : {0} to : {1}", oldPath,newPath);
        //AssetMoveResult.DidMove表示该资源可以移动
        return AssetMoveResult.DidMove;
    }
    //监听“资源即将被删除”事件
    public static AssetDeleteResult OnWillDeleteAsset(string assetPath,
        RemoveAssetOptions option)
    {
        Debug.LogFormat("delete : {0}", assetPath);
        //AssetDeleteResult.DidNotDelete表示该资源可以被删除
        return AssetDeleteResult.DidNotDelete;
    }
}
```

## 3.2　拓展Hierarchy视图

Hierarchy视图中出现的都是游戏对象，这些对象之间同样具有一定的关联关系。我们可以用树状结构来表示游戏对象之间复杂的父子关系。Hierarchy视图中的游戏对象会通过摄像机最终投影在发布的游戏中。Hierarchy视图也比较简单，本章就来学习如何拓展它使其更加丰富。

### 3.2.1　拓展菜单

在Hierarchy视图中，也可以对Create菜单项进行拓展。如图3-6所示，在Hierarchy视图中点击Create按钮，弹出的菜单My Create→Cube就是自定义拓展的菜单，相关代码如代码清单3-5所示。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=1809b513ae1ea581da57)

代码清单3-5　Script_03_05.cs文件

```
using UnityEngine;
using UnityEditor;

public class Script_03_05
{
    [MenuItem("GameObject/My Create/Cube",false,0)]
    static void CreateCube()
    {
        GameObject.CreatePrimitive(PrimitiveType.Cube); //创建立方体
    }
}
```

菜单中已经包含了系统默认的一些菜单项，我们拓展的原理就是重写MenuItem的自定义路径。Create按钮下的菜单项都在GameObject路径下面，所以只要开头是GameObject/xx/xx，均可自由拓展。

### 3.2.2　拓展布局

在Hierarchy视图中，同样可以对布局进行拓展。如图3-7所示，选择不同的游戏对象后，在右侧可根据EditorGUI拓展出一组按钮，点击Unity图标按钮后，在Console窗口中输入这个游戏对象。它的工作原理就是监听EditorApplication.hierarchyWindowItemOnGUI渲染回调。相关代码如代码清单3-6所示。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=1809576b024f83101d05)

代码清单3-6　Script_03_06.cs文件

```
using UnityEngine;
using UnityEditor;

public class Script_03_06
{
    [InitializeOnLoadMethod]
    static void InitializeOnLoadMethod()
    {
        EditorApplication.hierarchyWindowItemOnGUI = delegate(int instanceID,
            Rect selectionRect) {
            //在Hierarchy视图中选择一个资源
            if(Selection.activeObject &&
                instanceID ==Selection.activeObject.GetInstanceID()){
                //设置拓展按钮区域
                float width=50f;
                float height=20f;
                selectionRect.x +=(selectionRect.width - width);
                selectionRect.width = width;
                selectionRect.height= height;
                //点击事件
                if(GUI.Button(selectionRect,AssetDatabase.LoadAssetAtPath<Texture>
                    ("Assets/unity.png"))){
                    Debug.LogFormat("click : {0}",Selection.activeObject.name);
                }
            }    
        };
    }
}
```

在代码中实现EditorApplication.hierarchyWindowItemOnGUI委托，就可以重写Hierarchy视图了。这里我们使用GUI.Button来绘制自定义按钮，点击按钮可监听事件。GUI的种类比较丰富。此外，我们还可以拓展其他显示元素。

### 3.2.3　重写菜单

通过上面的学习，我们知道Hierarchy视图中的菜单可以在原有基础上拓展，那么如果想彻底抛弃它的菜单项，完全使用自己的菜单项是否可行呢？答案是可行的。如图3-8所示，先用鼠标选择一个游戏对象，点击右键即可弹出我们的重写菜单，这个菜单项已经和Unity自带的完全不一样了。它的工作原理就是监听点击的事件，打开一个新的菜单窗口。相关代码如代码清单3-7所示。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=180955f3980c69596d08)

代码清单3-7　Script_03_07.cs文件

```
using UnityEngine;
using UnityEditor;

public class Script_03_07
{
    [MenuItem("Window/Test/yusong")]
    static void Test()
    {
    }

    [MenuItem("Window/Test/momo")]
    static void Test1()
    {
    }
    [MenuItem("Window/Test/雨松/MOMO")]
    static void Test2()
    {
    }

    [InitializeOnLoadMethod]
    static void StartInitializeOnLoadMethod()
    {
        EditorApplication.hierarchyWindowItemOnGUI += OnHierarchyGUI;
    }

    static void OnHierarchyGUI(int instanceID, Rect selectionRect)
    {
        if(Event.current != null && selectionRect.Contains(Event.current.mousePosition)
            && Event.current.button == 1 && Event.current.type <= EventType.MouseUp)
        {
            GameObject selectedGameObject = EditorUtility.InstanceIDToObject(instanceID)
                as GameObject;
            //这里可以判断selectedGameObject的条件
            if(selectedGameObject)
            {
                Vector2 mousePosition = Event.current.mousePosition;

                EditorUtility.DisplayPopupMenu(new Rect(mousePosition.x,
                    mousePosition.y, 0, 0), "Window/Test",null);
                Event.current.Use();
            }
        }
    }
}
```

在上述代码中，我们使用Event.current来获取当前的事件。当监听到鼠标抬起的事件后，并且满足游戏对象的选中状态，开始执行自定义事件。其中，EditorUtility.DisplayPopupMenu用于弹出自定义菜单，Event.current.Use()的含义是不再执行原有的操作，所以就实现了重写菜单。

此外，Hierarchy视图还可以重写系统自带的菜单行为。例如，我觉得Unity创建的Image组件不好，可以复写它的行为，如图3-9所示。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=1809835283d85c344bec)

创建Image组件时，会自动勾选RaycastTarget。如果图片不需要处理点击事件，这样会带来一些额外的开销。代码清单3-8就是复写了创建Image组件的逻辑，让RaycastTarget默认不勾选。

代码清单3-8　Script_03_08.cs文件

```
using UnityEngine;
using UnityEditor;
using UnityEngine.UI;

public class Script_03_08
{
    [MenuItem("GameObject/UI/Image")]
    static void CreatImage()
    {
        if(Selection.activeTransform)
        {
            if(Selection.activeTransform.GetComponentInParent<Canvas>())
            {
                Image image = new GameObject("image").AddComponent<Image>();
                image.raycastTarget = false;
                image.transform.SetParent(Selection.activeTransform,false);
                //设置选中状态
                Selection.activeTransform = image.transform;
            }
        }
    }
}
```

由于重写了菜单，所以需要通过脚本自行创建Image对象和组件。接着，获取到image组件对象，直接设置它的raycastTarget属性即可。

## 3.3　拓展Inspector视图

Inspector视图可用来展示组件以及资源的详细信息面板，每个组件的面板信息是各不相同的。系统提供的大量组件通常可以满足开发需求，但是偶尔我们还是希望能在原有组件上去拓展，比如添加一些按钮或者添加一些逻辑等。

### 3.3.1　拓展源生组件

摄像机就是典型的源生组件。如图3-10所示，可以在摄像机组件的最上面添加一个按钮。它的局限性就是拓展组件只能加在源生组件的最上面或者最下面，不能插在中间，不过这样也就够了。相关代码如代码清单3-9所示。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=180902e2764917e0ee47)

代码清单3-9　Script_03_09.cs文件

```
using UnityEngine;
using UnityEditor;

[CustomEditor(typeof(Camera))]
public class Script_03_09 : Editor
{
    public override void OnInspectorGUI(){
        if(GUILayout.Button("拓展按钮")) {
        }
        base.OnInspectorGUI();
    }
}
```

在代码清单3-9中，CustomEditor()表示自定义哪个组件，OnInspectorGUI()可以对它进行重新绘制，base.OnInspectorGUI()表示是否绘制父类原有元素。

### 3.3.2　拓展继承组件

有些系统组件可能在Unity内部已经重写了绘制方法，但是外部是访问不了内部代码的，所以修改起来比较麻烦。不过，我们还是有办法的。如图3-11所示的Transform组件，按照前面介绍的方式直接拓展的话，它的面板就变得非常丑陋，最好拓展面板后，还能保留组件原有的绘制方式。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=1809f4c451b68bbaf9a6)

Unity将大量的Editor绘制方法封装在内部的DLL文件里，开发者无法调用它的方法。如果想解决这个问题，可以使用C# 反射的方式调用内部未公开的方法。如图3-12所示，通过拓展的Transfom组件，现在就可以保留原有的绘制方式了。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=1809d5608f8e8da24bc6)

如代码清单3-10所示，通过反射先得到UnityEditor.TransformInspector对象，然后就可以调用它内部的OnInspectorGUI()方法了。

代码清单3-10　Script_03_10.cs文件

```
using UnityEngine;
using UnityEditor;
using System.Reflection;

[CustomEditor(typeof(Transform))]
public class Script_03_10 : Editor
{
    private Editor m_Editor;
    void OnEnable()
    {
        m_Editor = Editor.CreateEditor(target,
            Assembly.GetAssembly(typeof(Editor)).GetType("UnityEditor.
                TransformInspector",true));
    }

    public override void OnInspectorGUI(){
        if(GUILayout.Button("拓展按钮")) {
        }
        //调用系统绘制方法
        m_Editor.OnInspectorGUI();
        //base.OnInspectorGUI();
    }
}
```

上述代码中，我们重写了OnInspectorGUI()方法。使用GUILayout.Button绘制了自定义的按钮元素，接着调用m_Editor.OnInspectorGUI()绘制Transform原有面板信息，这样我们拓展的按钮就会显示在Transform面板的上方。

### 3.3.3　组件不可编辑

在Unity中，我们可以给组件设置状态，这样它就无法编辑了。如图3-13所示，将Transform组件的原始功能禁掉（灰色表示不可编辑），而不影响我们上下拓展的两个按钮。相关代码如代码清单3-11所示。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=18094846d57d72e3f0bb)

代码清单3-11　Script_03_11.cs文件

```
using UnityEngine;
using UnityEditor;
using System.Reflection;

[CustomEditor(typeof(Transform))]
public class Script_03_11 : Editor
{
    private Editor m_Editor;
    void OnEnable()
    {
        m_Editor = Editor.CreateEditor(target,
            Assembly.GetAssembly(typeof(Editor)).GetType("UnityEditor.
                TransformInspector",true));
    }

    public override void OnInspectorGUI(){
        if(GUILayout.Button("拓展按钮上")) {
        }
        //开始禁止
        GUI.enabled = false;
        m_Editor.OnInspectorGUI();
        //结束禁止
        GUI.enabled = true;
        if(GUILayout.Button("拓展按钮下")) {
        }
    }
}
```

如果想整体禁止组件，可以按照图3-14所示选择任意游戏对象，然后从右键菜单中选择3D Object→Lock→Lock（锁定）或者UnLock（解锁）。它的原理就是设置游戏对象的hideFlags。需要说明的是，我们不一定非设置游戏对象的hideFlags，也可以单独给某个组件设置hideFlags，这样只会影响到某一个组件并非全部。相关代码如代码清单3-12所示。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=1809b425118335d58d94)

代码清单3-12　Script_03_12.cs文件

```
using UnityEngine;
using UnityEditor;

public class Script_03_12
{
    [MenuItem("GameObject/3D Object/Lock/Lock",false,0)]
    static void Lock()
    {
        if(Selection.gameObjects != null) {
            foreach(var gameObject in Selection.gameObjects) {
                gameObject.hideFlags = HideFlags.NotEditable;
            }
        }
    }
    [MenuItem("GameObject/3D Object/Lock/UnLock",false,1)]
    static void UnLock()
    {
        if(Selection.gameObjects != null) {
            foreach(var gameObject in Selection.gameObjects) {
                gameObject.hideFlags = HideFlags.None;
            }
        }
    }
}
```

HideFlags可以使用按位或（|）同时保持多个属性，其含义都很好理解，大家可以自行输入代码调试一下。

    HideFlags.None：清除状态。

    HideFlags.DontSave：设置对象不会被保存（仅编辑模式下使用，运行时剔除掉）。

    HideFlags.DontSaveInBuild：设置对象构建后不会被保存。

    HideFlags.DontSaveInEditor：设置对象编辑模式下不会被保存。

    HideFlags.DontUnloadUnusedAsset：设置对象不会被Resources.UnloadUnused-
Assets()卸载无用资源时卸掉。

    HideFlags.HideAndDontSave：设置对象隐藏，并且不会被保存。

    HideFlags.HideInHierarchy：设置对象在层次视图中隐藏。

    HideFlags.HideInInspector：设置对象在控制面板视图中隐藏。

    HideFlags.NotEditable：设置对象不可被编辑。

### 3.3.4　Context菜单

点击组件中设置（鼠标右键），可以弹出Context菜单，如图3-15所示，我们可在原有菜单中拓展出新的菜单栏，相关代码如代码清单3-13所示。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=180927cf08d48693d61c)

代码清单3-13　Script_03_13.cs文件

```
using UnityEngine;
using UnityEditor;

public class Script_03_13
{
    [MenuItem("CONTEXT/Transform/New Context 1")]
    public static void NewContext1(MenuCommand command)
    {    
        //获取对象名
        Debug.Log(command.context.name);
    }
    [MenuItem("CONTEXT/Transform/New Context 2")]
    public static void NewContext2(MenuCommand command)
    {
        Debug.Log(command.context.name);
    }
}
```

其中，[MenuItem("CONTEXT/Transform/New Context 1")] 表示将新菜单扩展在Transform组件上。如果想拓展在别的组件上，例如摄像机组件，直接修改字符串中的Transform为Camera即可。如果想给所有组件都添加菜单栏，这里改成Compoment即可。

以上设置也可以应用在自己写的脚本中。如图3-16所示，Script_03_14.cs是自己创建的脚本，在代码中可以通过MenuCommand来获取脚本对象，从而访问脚本中的变量，相关代码如代码清单3-14所示。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=1809651ce50fb2f77a66)

代码清单3-14　Script_03_14.cs文件

```
using UnityEngine;
#if UNITY_EDITOR
using UnityEditor;
#endif
public class Script_03_14 : MonoBehaviour
{
    public string contextName;
    #if UNITY_EDITOR
    [MenuItem("CONTEXT/Script_03_14/New Context 1")]
    public static void NewContext2(MenuCommand command)
    {
        Script_03_14 script = (command.context as Script_03_14);
        script.contextName = "hello world!!";
    }
    #endif
}
```

在上述代码中，我们使用到了宏定义标签，其中UNITY_EDITOR表示这段代码只会在Editor模式下执行，发布后将被剔除掉。

当然，我们也可以在自己的脚本中这样写。如果和系统的菜单项名称一样，还可以覆盖它。比如，这里重写了删除组件的按钮，就可以执行一些自己的操作了：

```
[ContextMenu("Remove Component")]
void RemoveComponent()
{
    Debug.Log("RemoveComponent");
    //等一帧再删除自己
    UnityEditor.EditorApplication.delayCall = delegate() {
        DestroyImmediate(this);
    };
}
```

编辑模式下的代码同步时有可能会有问题，比如上述DestroyImmediate(this)删除自己的代码时，会触发引擎底层的一个错误。不过我们可以使用UnityEditor.EditorApplication.
delayCall来延迟一帧调用。后续如果大家在开发编辑器代码时发现类似问题，也可以尝试等一帧再执行自己的代码。

## 3.4　拓展Scene视图

Scene视图承担着游戏“第三人称”观察的工作。Unity提供了强大的Gizmos工具API，我们可以在Scene视图中绘制立方体、网格、贴图、射线和UI等，开发者可以自由地拓展显示组件。

### 3.4.1　辅助元素

场景在编辑的过程中，通常需要一些辅助元素，这样使用者可以更高效地完成编辑工作。如图3-17所示，选中Main Camera对象时，程序会给摄像机组件添加一条红色的辅助线，并且在线段终点处添加一个立方体辅助对象。请注意这里拓展的辅助元素只能用来编辑，并不会影响到最终发布的游戏。相关代码如代码清单3-15所示。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=1809e9f9948af9d33dfc)

Gizmo的绘制原理就是在脚本中添加OnDrawGizmosSelected()，此方法仅在编辑模式下生效。使用Gizmos.cs工具类，我们可以绘制出任意辅助元素。

代码清单3-15　Script_03_15.cs文件

```
using UnityEngine;

public class Script_03_15 : MonoBehaviour
{
    void OnDrawGizmosSelected()
    {
        Gizmos.color = Color.red;
        //画线
        Gizmos.DrawLine(transform.position, Vector3.one);
        //立方体
        Gizmos.DrawCube(Vector3.one, Vector3.one);
    }
}
```

如果希望辅助元素并不依赖选择对象出现，而是始终都出现在Scene视图中，可使用方法OnDrawGizmos()绘制元素：

```
void OnDrawGizmos()
{
    Gizmos.DrawSphere(transform.position, 1);
}
```

此外，Gizmos工具类中还有很多常用绘制元素，读者也可以自行摸索。

### 3.4.2　辅助UI

在Scene视图中，我们可以添加EditorGUI，这样可以方便地在视图中处理一些操作事件。如图3-18所示，我们可以在Scene视图中绘制辅助UI，EditorGUI的代码需要在Handles.BeginGUI()和Handles.EndGUI()中间绘制完成。这里我们只设置摄像机辅助UI，其实也可以修改成别的对象，比如游戏对象。相关代码如代码清单3-16所示。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=180952668bda8034c1f5)

代码清单3-16　Script_03_16.cs文件

```
using UnityEngine;
using UnityEditor;
[CustomEditor(typeof(Camera))]
public class Script_03_16 : Editor
{
    void OnSceneGUI()
    {
        Camera camera = target as Camera;
        if(camera!=null){
            Handles.color = Color.red;
            Handles.Label(camera.transform.position, camera.transform.position.
                ToString());

            Handles.BeginGUI();
            GUI.backgroundColor = Color.red;
            if(GUILayout.Button("click",GUILayout.Width(200f))) {
                Debug.LogFormat("click = {0}", camera.name);
            }
            GUILayout.Label("Label");
            Handles.EndGUI();
        }
    }
}
```

在上述代码中，我们继承了Editor类，这样重写OnSceneGUI()方法，就可以在Scene视图中拓展自定义元素了。

### 3.4.3　常驻辅助UI

上一节中，我们介绍的辅助UI需要选中一个游戏对象。当然，我们也可以设置常驻辅助UI。例如，无须选择游戏对象，EditorGUI将常驻显示在Scene视图中，如图3-19所示。其原理就是要重写SceneView.onSceneGUIDelegate，依然需要在Handles.BeginGUI()和Handles.EndGUI()中间绘制完成。相关代码如代码清单3-17所示。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=18096b5ae24feb883e72)

代码清单3-17　Script_03_17.cs文件

```
using UnityEngine;
using UnityEditor;

public class Script_03_17
{
    [InitializeOnLoadMethod]
    static void InitializeOnLoadMethod(){
        SceneView.onSceneGUIDelegate = delegate(SceneView sceneView) {
            Handles.BeginGUI();

            GUI.Label(new Rect(0f,0f,50f,15f),"标题");
            GUI.Button(new Rect(0f,20f,50f,50f),
                AssetDatabase.LoadAssetAtPath<Texture>("Assets/unity.png"));

            Handles.EndGUI();
        };
    }
}
```

上述代码中，全局监听了SceneView.onSceneGUIDelegate委托，这样就可以使用GUI全局绘制元素了。

### 3.4.4　禁用选中对象

在Scene视图和Hierarchy视图中，都可以选择游戏对象。Scene视图中因为东西很多，而且很可能大量重叠，很容易选错对象。在开发编辑器的时候，当操作某个对象时，如果不希望Scene视图中误操作别的对象，我们可以禁用选中对象的功能。相关代码如代码清单3-18所示。

代码清单3-18　Script_03_18.cs文件

```
using UnityEngine;
using UnityEditor;

public class Script_03_18
{
    [InitializeOnLoadMethod]
    static void InitializeOnLoadMethod(){
        SceneView.onSceneGUIDelegate = delegate(SceneView sceneView) {
            Event e = Event.current;
            if(e != null){
                int controlID = GUIUtility.GetControlID(FocusType.Passive);
                if(e.type == EventType.Layout)
                {
                    HandleUtility.AddDefaultControl(controlID);
                }
            }
        };
    }
}
```

在上述代码中，FocusType.Passive表示禁止接收控制焦点，获取它的controllID后，即可禁止将点击事件穿透下去。

此外，还有个办法可以禁止选中功能，即以层为单位设置某个层无法选中。如图3-20所示，右边有个“小锁头”的就无法选中了。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=1809e5c5f04ef4219508)

直接在Scene视图中很容易选择到子节点，此时可以给它绑定一个[SelectionBase]标记，这样该脚本下的所有节点都会定位到绑定这个标记的对象身上：

```
[SelectionBase]
public class RootScript : MonoBehaviour {
}
```

## 3.5　拓展Game视图

Game视图输出的是最终的游戏画面，理论上是不需要拓展的，不过Unity也可以对其进行拓展。它的拓展主要分为两种：运行模式下以及非运行模式下。

脚本挂在游戏对象后，需要运行游戏才可以执行脚本的生命周期，不过非运行模式下其实也可以执行脚本。如图3-21所示，Game视图在非运行模式下也可以绘制GUI。其原理就是在脚本类名上方声明[ExecuteInEditMode]，表示此脚本可以在编辑模式中生效。此类脚本通常只是用来做编辑器，正式发布后是不需要的，此时可以使用UNITY_EDITOR条件编译发布后剥离掉。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=1809eb5520e740456a6a)

如代码清单3-19所示，在类的上面标记[ExecuteInEditMode]，表示非运行模式下也会执行代码的生命周期，接着在OnGUI()方法中就可以绘制元素了。

代码清单3-19　Script_03_19.cs文件

```
using UnityEngine;
#if UNITY_EDITOR
[ExecuteInEditMode]
public class Script_03_19 : MonoBehaviour
{
    void OnGUI()
    {
        if(GUILayout.Button("Click")) {
            Debug.Log("click!!!");
        }
        GUILayout.Label("Hello World!!!");
    }
}
#endif
```

## 3.6　MenuItem菜单

Unity编辑器使用的拓展菜单是MenuItem。当然，开发者也可以自由拓展，直接使用“/”符号区分开它的路径即可。系统上方自带的一排菜单也在大量使用这个功能。

### 3.6.1　覆盖系统菜单

Unity编辑器中自带了很多菜单，大多数情况下可以按照它的原有菜单路径拼合集合。例如，如果需要重写创建Text按钮的功能，可以按照下面的代码执行：

```
[MenuItem("GameObject/UI/Text")]
static void CreateNewText()
{
    Debug.Log("CreateNewText!");
}
```

### 3.6.2　自定义菜单

自定义菜单可以设置路径、排序、勾选框和禁止选中状态。如图3-22所示，菜单中下方有一个下划线。在代码中设置上一个菜单的priority（优先级），一共预留了10个元素位置，只需要priority+11即可，就会自动增加这个下划线效果。相关代码如代码清单3-20所示。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=180997595b5720bf8293)

代码清单3-20　Script_03_20.cs文件

```
using UnityEngine;
using UnityEditor;

public class Script_03_20
{
    [MenuItem("Root/Test1",false,1)]
    static void Test1()
    {
    }
    //菜单排序
    [MenuItem("Root/Test0",false,0)]
    static void Test0()
    {
    }
    [MenuItem("Root/Test/2")]
    static void Test2()
    {
    }
    [MenuItem("Root/Test/2", true,20)]
    static bool Test2Validation()
    {
        //false表示Root/Test/2菜单将置灰，即不可点击
        return false;
    }
    [MenuItem("Root/Test3",false,3)]
    static void Test3()
    {
        //勾选框中的菜单
        var menuPath = "Root/Test3";
        bool mchecked = Menu.GetChecked(menuPath);
        Menu.SetChecked(menuPath, !mchecked);
    }
}
```

### 3.6.3　源生自定义菜单

MenuItem是依托于Unity编辑器的菜单栏，换句话说就是无法设置它的位置。如果希望菜单的位置以及出现时机更加灵活的话，可以调用源生自定义菜单的方法。如图3-23所示，拓展后，可以在Scene视图中点击鼠标右键，此时会弹出一组源生自定义菜单。相关代码如代码清单3-21所示。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=180915d27f2d24a12c9c)

代码清单3-21　Script_03_21.cs文件

```
using UnityEngine;
using UnityEditor;

public class Script_03_21
{
    [InitializeOnLoadMethod]
    static void InitializeOnLoadMethod()
    {
        SceneView.onSceneGUIDelegate = delegate(SceneView sceneView) {
            Event e = Event.current;
            //鼠标右键抬起时
            if(e != null && e.button ==1 && e.type == EventType.MouseUp){
                Vector2 mousePosition = e.mousePosition;
                //设置菜单项
                var options = new GUIContent[]{
                    new GUIContent("Test1"),
                    new GUIContent("Test2"),
                    new GUIContent(""),
                    new GUIContent("Test/Test3"),
                    new GUIContent("Test/Test4"),    
                };
                //设置菜单显示区域
                var selected= -1;
                var userData=Selection.activeGameObject;
                var width =100;
                var height =100;
                var position =new Rect(mousePosition.x,mousePosition.y - height,
                    width,height);
                //显示菜单
                EditorUtility.DisplayCustomMenu(position,options,selected,
                    delegate(object data, string[] opt, int select) {
                    Debug.Log(opt[select]);
                },userData);
                e.Use();
            }
        };
    }
}
```

在上述代码中，首先监听鼠标右键以获取鼠标位置，接着使用EditorUtility.Display-
CustomMenu()方法来弹出自定义菜单，以及监听菜单选择后的事件。

### 3.6.4　拓展全局自定义快捷键

Unity没有提供全局自定义快捷键的拓展，不过可以利用MenuItem提供的快捷键来实现这个目的。如图3-24所示，我们自定义了快捷键Command +Shift +D，使用者将需要执行的逻辑（即快捷键后的逻辑）写在方法体内即可。相关代码如代码清单3-22所示。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=1809a555ba977b54e0c5)

代码清单3-22　Script_03_22.cs文件

```
using UnityEngine;
using UnityEditor;

public class Script_03_22
{
    [MenuItem("Assets/HotKey %#d",false,-1)]
    private static void HotKey()
    {
        Debug.Log("Command Shift + D");
    }
}
```

热键可以相互组合，其中 %#d就表示 Command+Shift+D。按照这个格式，我们也可以自由拓展热键组合。

其他热键如下。

    %：表示Windows 下的 Ctrl键和macOS下的Command键。

    #：表示Shift键。

    &：表示Alt键。

    LEFT/RIGHT/UP/DOWN：表示左、右、上、下4个方向键。

    F1…F12：表示F1至F12菜单键。

    HOME、END、PGUP和PGDN键。

## 3.7　面板拓展

脚本挂在游戏对象上时，右侧会出现它的详细信息面板，这些信息是根据脚本中声明的public可序列化变量而来的。此外，也可以通过EditorGUI来对它进行绘制，让面板更具可操作性。

### 3.7.1　Inspector面板

EditorGUI和GUI的用法几乎完全一致，目前来说前者多用于编辑器开发，后者多用于发布后调试编辑器。总之，它们都是起辅助作用的。EditorGUI提供的组件非常丰富，常用的绘制元素包括文本、按钮、图片和滚动框等。做一个好的编辑器，是离不开EditorGUI的。如图3-25所示，我们将EditorGUI拓展在Inspector面板上了，相关代码如代码清单3-23所示。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=1809442410359155c4cf)

代码清单3-23　Script_03_23.cs文件

```
using UnityEngine;
#if UNITY_EDITOR
using UnityEditor;
#endif

public class Script_03_23 : MonoBehaviour
{
    public Vector3 scrollPos;
    public int myId;
    public string myName;
    public GameObject prefab;
    public MyEnum myEnum = MyEnum.One;
    public bool toogle1;
    public bool toogle2;

    public enum MyEnum
    {
        One=1,
        Two,
    }
}

#if UNITY_EDITOR
[CustomEditor(typeof(Script_03_23))]
public class ScriptEditor_03_23 : Editor
{
    private bool m_EnableToogle;

    public override void OnInspectorGUI()
    {
        //获取脚本对象
        Script_03_23 script = target as Script_03_23;
        //绘制滚动条
        script.scrollPos =
            EditorGUILayout.BeginScrollView(script.scrollPos,false,true);        

        script.myName = EditorGUILayout.TextField("text",script.myName);
        script.myId = EditorGUILayout.IntField("int", script.myId);
        script.prefab = EditorGUILayout.ObjectField("GameObject", script.prefab,
            typeof(GameObject),true)as GameObject;

        //绘制按钮
        EditorGUILayout.BeginHorizontal();
        GUILayout.Button("1");
        GUILayout.Button("2");
        script.myEnum =(Script_03_23.MyEnum)EditorGUILayout.EnumPopup("MyEnum:",
            script.myEnum);
        EditorGUILayout.EndHorizontal();
        //Toogle组件
        m_EnableToogle = EditorGUILayout.BeginToggleGroup("EnableToogle",
            m_EnableToogle);
        script.toogle1 = EditorGUILayout.Toggle("toogle1", script.toogle1);
        script.toogle2 = EditorGUILayout.Toggle("toogle2", script.toogle2);
        EditorGUILayout.EndToggleGroup();

        EditorGUILayout.EndScrollView();
    }
}
#endif
```

在上述代码中，我们将脚本部分和Editor部分的代码合在一个文件中。如果需要拓展的面板比较复杂，建议分成两个文件存放，一个是脚本，另一个是Editor脚本。

### 3.7.2　EditorWindows窗口

Unity提供编辑器窗口，开发者可以自由拓展自己的窗口。Unity编辑器系统自带的视图窗口其实也是用EditorWindows实现的。如图3-26所示，我们来制作一个简单的编辑窗口，它绘制元素时同样使用EditorGUI代码。由此可见，一个完美的编辑器GUI的知识是多么重要啊！

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=180953f8013abddc4dc3)

如代码清单3-24所示，使用EditorWindow.GetWindow()方法即可打开自定义窗口，在OnGUI()方法中可以绘制窗口元素。请大家注意代码中EditorWindows窗口的生命周期。

代码清单3-24　Script_03_24.cs文件

```
using UnityEngine;
using UnityEditor;

public class Script_03_24 : EditorWindow
{

    [MenuItem("Window/Open My Window")]
    static void Init()
    {
        Script_03_24 window =(Script_03_24)EditorWindow.GetWindow(typeof(Script_03_24));
        window.Show();
    }

    private Texture m_MyTexture = null;
    private float m_MyFloat = 0.5f;
    void Awake()
    {
        Debug.LogFormat("窗口初始化时调用");
        m_MyTexture = AssetDatabase.LoadAssetAtPath<Texture>("Assets/unity.png");
    }
    void OnGUI()
    {
        GUILayout.Label("Hello World!!", EditorStyles.boldLabel);
        m_MyFloat = EditorGUILayout.Slider("Slider", m_MyFloat, -5, 5);
        GUI.DrawTexture(new Rect(0,30,100,100),m_MyTexture);
    }
    void OnDestroy()
    {
        Debug.LogFormat("窗口销毁时调用");
    }
    void OnFocus(){
        Debug.LogFormat("窗口拥有焦点时调用");
    }
    void OnHierarchyChange()
    {
        Debug.LogFormat("Hierarchy视图发生改变时调用");
    }
    void OnInspectorUpdate()
    {
        //Debug.LogFormat("Inspector每帧更新");
    }    
    void OnLostFocus()
    {
        Debug.LogFormat("失去焦点");
    }
    void OnProjectChange()
    {
        Debug.LogFormat("Project视图发生改变时调用");
    }
    void OnSelectionChange()
    {
        Debug.LogFormat("在Hierarchy或者Project视图中选择一个对象时调用");
    }    
    void Update()
    {
        //Debug.LogFormat("每帧更新");
    }
}
```

### 3.7.3　EditorWindows下拉菜单

如图3-27所示，在EditorWindows编辑窗口的右上角，有个下拉菜单，我们也可以对该菜单中的选项进行拓展，不过这里需要实现IHasCustomMenu接口。相关代码如代码清单3-25所示。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=18094c27f6d66c9dc40f)

代码清单3-25　Script_03_25.cs文件

```
using UnityEngine;
using UnityEditor;

public class Script_03_25 : EditorWindow, IHasCustomMenu
{
    void IHasCustomMenu.AddItemsToMenu(GenericMenu menu)
    {
        menu.AddDisabledItem(new GUIContent("Disable"));
        menu.AddItem(new GUIContent("Test1"), true,()=> {
            Debug.Log("Test1");
        });
        menu.AddItem(new GUIContent("Test2"), true,()=> {
            Debug.Log("Test2");
        });
        menu.AddSeparator("Test/");
        menu.AddItem(new GUIContent("Test/Tes3"),true,()=> {
            Debug.Log("Tes3");
        });

    }

    [MenuItem("Window/Open My Window")]
    static void Init()
    {
        Script_03_25 window =(Script_03_25)EditorWindow.GetWindow(typeof
            (Script_03_25));
        window.Show();
    }
}
```

上述代码中，我们通过AddItem()方法来添加列表元素，并且监听选择后的事件。

### 3.7.4　预览窗口

选择游戏对象或者游戏资源后，Inspector面板下方将会出现它的预览窗口，但是有些资源是没有预览信息的，不过我们可以监听它的窗口方法来重新绘制它，如图3-28所示，相关代码如代码清单3-26所示。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=180986510d904ea7b6d0)

代码清单3-26　Script_03_26.cs文件

```
using UnityEngine;
using UnityEditor;

[CustomPreview(typeof(GameObject))]
public class Script_03_26 : ObjectPreview
{
    public override bool HasPreviewGUI()
    {
        return true;
    }
    public override void OnPreviewGUI(Rect r, GUIStyle background)
    {
        GUI.DrawTexture(r,AssetDatabase.LoadAssetAtPath<Texture>("Assets/unity.png"));
        GUILayout.Label("Hello World!!!");
    }
}
```

这段代码的原理就是继承ObjectPreview并且重写OnPreviewGUI()方法，接着就可以通过代码进行绘制了。[CustomPreview(typeof(GameObject))]中的GameObject代表需要重新绘制的预览对象，也可以换成别的系统对象或自定义的脚本对象。

### 3.7.5　获取预览信息

有些资源是有预览信息的，比如模型资源。在预览窗口中，我们可以看到它的样式。如果需要在自定义窗口中显示它，就需要获取它的预览信息。如图3-29所示，选择一个游戏对象后，会在自定义窗口中显示它，相关代码如代码清单3-27所示。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=18090f24d3bc9d5c361f)

代码清单3-27　Script_03_27.cs文件

```
using UnityEngine;
using UnityEditor;

public class Script_03_27 : EditorWindow
{
    private GameObject m_MyGo;
    private Editor m_MyEditor;

    [MenuItem("Window/Open My Window")]
    static void Init()
    {
        Script_03_27 window =(Script_03_27)EditorWindow.GetWindow(typeof(Script_03_27));
        window.Show();
    }
    void OnGUI() {
        //设置一个游戏对象
        m_MyGo =(GameObject) EditorGUILayout.ObjectField(m_MyGo,
            typeof(GameObject), true);

        if(m_MyGo != null) {
            if(m_MyEditor == null) {
                //创建Editor实例
                m_MyEditor = Editor.CreateEditor(m_MyGo);
            }
            //预览它
            m_MyEditor.OnPreviewGUI(GUILayoutUtility.GetRect(500, 500),
                EditorStyles.whiteLabel);
        }
    }
}
```

在上述代码中，预览对象首先需要通过Editor.CreateEditor()拿到它的Editor实例对象，接着调用OnPreviewGUI()方法传入窗口的显示区域。

## 3.8　Unity编辑器的源码

Unity编辑器几乎都是用C# 编写而成的，视图中也大量使用EditorGUI来编辑布局。例如，对于常见的5大布局视图，所有的代码都放在UnityEditor.dll中。如图3-30所示，打开Unity安装目录，在Managed子目录中存放着引擎所需要用到的所有DLL文件。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=1809cd170a19a5d816be)

### 3.8.1　查看DLL

拿到UnityEditor.dll以后，就可以通过第三方工具来分析和查看了，常用的工具包括 .NET Reflector以及ILSpy。此外，也可以使用Unity自带的IDE（Visual Studio）来查看。

用Visual Studio打开任意Unity工程，接着在Assembly-CSharp-Editor中找到UnityEditor，双击打开它，然后在左上方的Visibility中选择All members，在右上角的Language中选择C#，此时源码都出来了，如图3-31所示。可以发现，面板最上方还有个搜索框，可以用来模糊搜索代码。

有兴趣的朋友还可以看看UnityEngine.dll。其实Unity的C# 版API接口都在这里，只是源码的核心功能都是在C/C++ 中完成的，DLL只负责中间调用接口而已。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=18092b696b2da02edf22)

阅读UnityEditor.dll源码是非常必要的，因为只有熟悉了Unity自己内部的编辑器开发，我们才能做出更完美的编辑器。但是Unity内部的有些方法设置了sealed属性，这样我们写的代码就无法访问它了，所以有时我们不得不使用反射来做点东西。反射可以访问私有成员变量、私有成员方法、私有静态变量和静态方法。

### 3.8.2　清空控制台日志

系统日志以及Debug.Log()产出的日志都输出在Console窗口中。在Console窗口的左上角，有个Clean按钮，它用于清空控制台日志。如果希望脚本可以灵活自动清空日志，就必须使用反射了。如图3-32所示，首先找到控制台的窗口类（ConsoleWindows.cs），接着在OnGUI()方法中可以看到：点击Clean按钮后，Unity会执行LogEntries.Clear()方法。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=18099f932e7569be2788)

接着，在反射中找到LogEntries.cs类。如图3-33所示，这个类标记了sealed属性，所以外部是无法直接访问到的，只能反射调用Clear()这个方法了。相关代码如代码清单3-28所示。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=1809eea108f8a23ce229)

代码清单3-28　Script_03_28.cs文件

```
using UnityEngine;
using UnityEditor;
using System.Reflection;

public class Script_03_28
{
    [MenuItem("Tools/CreateConsole")]
    static void CreateConsole()
    {
        Debug.Log("CreateConsole");
    }

    [MenuItem("Tools/CleanConsole")]
    static void CleanConsole()
    {
        //获取assembly
        Assembly assembly = Assembly.GetAssembly(typeof(Editor));
        //反射获取LogEntries对象
        MethodInfo methodInfo = assembly.GetType("UnityEditor.LogEntries").
            GetMethod("Clear");
        //反射调用它的Clear方法
        methodInfo.Invoke(new object(), null);
    }
}
```

上述代码中，我们学习了如何通过反射调用Unity内部方法。此外，Unity还有很多内部方法。

### 3.8.3　获取EditorStyles样式

EditorStyles是编辑器用到的样式，但是Unity文档中并没有集中说明每个样式对应的效果。开发一个漂亮的编辑器，肯定会用到很多样式，我们可以借助反射的原理找出它们，这样以后使用起来就方便多了。如图3-34和图3-35所示，反射出EditorSytles的属性，找到GUIStyle并最终在OnGUI中预览出来。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=180975ea0eae5b40871f)
如代码清单3-29所示，首先反射获取EditorStyles下的所有样式，然后在窗口中依次绘制出来。其中，BindingFlags表示反射属性的类型。

代码清单3-29　Script_03_29.cs文件

```
using UnityEngine;
using UnityEditor;
using System.Reflection;
using System.Collections.Generic;

public class Script_03_29 : EditorWindow
{
    static List<GUIStyle> styles = null;
    [MenuItem("Window/Open My Window")]
    public static void Test() {
        EditorWindow.GetWindow<Script_03_29>("styles");

        styles = new List<GUIStyle>();
        foreach(PropertyInfo fi in typeof(EditorStyles).GetProperties(BindingFlags.        Static|BindingFlags.Public|BindingFlags.NonPublic))
        {
            object o = fi.GetValue(null, null);
            if(o.GetType() == typeof(GUIStyle)) {
                styles.Add(o as GUIStyle);
            }
        }
    }

    public Vector2 scrollPosition = Vector2.zero;
    void OnGUI()
    {
        scrollPosition = GUILayout.BeginScrollView(scrollPosition);
        for(int i = 0; i < styles.Count; i++) {
            GUILayout.Label("EditorStyles." +styles[i].name, styles[i]);
        }
        GUILayout.EndScrollView();
    }
}
```

通过上述代码，Unity引擎内部的所有样式都反射出来了。如果以后做编辑器时需要用到这些样式，直接设置style名字即可。

### 3.8.4　获取内置图标样式

Unity编辑器还内置了很多图标样式，这些样式也没有在文档中详细说明。如图3-36所示，系统一共内置了2000多个样式，有了这些图标，就可以随心所欲地拓展编辑器了。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=1809862740648420924b)

如代码清单3-30所示，首先通过Resources.FindObjectsOfTypeAll()方法查出所有贴图，接着使用EditorGUIUtility.IconContent()方法加载引擎所用到的图标。

代码清单3-30　Script_03_30.cs文件

```
using UnityEngine;
using UnityEditor;
using System.Reflection;
using System.Collections.Generic;
using System;

public class Script_03_30 : EditorWindow
{
    [MenuItem("Window/Open My Window")]
    public static void OpenMyWindow() {
        EditorWindow.GetWindow<Script_03_30>("icons");
    }
    private Vector2 m_Scroll;
    private List<string> m_Icons = null;
    void Awake()
    {
        m_Icons = new List<string>();;
        Texture2D[] t = Resources.FindObjectsOfTypeAll<Texture2D>();
        foreach(Texture2D x in t) {
            Debug.unityLogger.logEnabled = false;
            GUIContent gc = EditorGUIUtility.IconContent(x.name);
            Debug.unityLogger.logEnabled = true;
            if(gc != null && gc.image != null) {
                m_Icons.Add(x.name);
            }
        }
        Debug.Log(m_Icons.Count);
    }
    void OnGUI()
    {
        m_Scroll = GUILayout.BeginScrollView(m_Scroll);
        float width = 50f;
        int count =(int)(position.width / width);
        for(int i =0; i< m_Icons.Count; i += count)
        {
            GUILayout.BeginHorizontal();
            for(int j =0; j < count; j++)
            {
                int index = i + j;
                if(index < m_Icons.Count)
                    GUILayout.Button(EditorGUIUtility.IconContent(m_Icons[index]),
                        GUILayout.Width(width), GUILayout.Height(30));
            }
            GUILayout.EndHorizontal();
        }

        EditorGUILayout.EndScrollView();
    }
}
```

通过上述代码，我们将引擎内所有的图标展示在窗口中。通过这些图标，可以让自定义编辑器的内容更加丰富。

### 3.8.5　拓展默认面板

有些资源系统没有提供面板，比如文件夹面板。如图3-37所示，选择任意文件夹，声明[CustomEditor(typeof(UnityEditor.DefaultAsset))]后，即可开始拓展它。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=1809d4ef761469525490)

如代码清单3-31所示，在OnInspectorGUI()方法中就可以拓展自定义文件夹面板了。

代码清单3-31　Script_03_31.cs文件

```
using UnityEngine;
using UnityEditor;

[CustomEditor(typeof(UnityEditor.DefaultAsset))]
public class Script_03_31 : Editor
{
    public override void OnInspectorGUI()
    {
        string path = AssetDatabase.GetAssetPath(target);
        GUI.enabled = true;
        if(path.EndsWith(string.Empty)){
            GUILayout.Label("拓展文件夹");
            GUILayout.Button("我是文件夹");
        }
    }
}
```

在上述代码中，UnityEditor.DefaultAsset表示默认资源，也就是Unity引擎无法识别的资源类型。如果还有别的资源需要拓展面板，大家可以判断路径的后缀。

### 3.8.6　例子：查找ManagedStaticReferences()静态引用

在Profiler里，会经常看到某个资源在内存中，但无法被GC卸载掉，其原因可能是它被static静态引用了，也可能是资源的循环引用造成的。这样，我们就无法分辨出在哪里被引用了。在下面的代码中，我们使用static强引用一个游戏资源：

```
public class NewBehaviourScript : MonoBehaviour
{
    public static GameObject prefab;
    void Start() {
        prefab = Resources.Load<GameObject>("prefab");
    }
}
```

运行游戏后，通过反射来递归查询这个资源被代码哪里static强引用了。如图3-38所示，在导航栏菜单中选择“脚本Static引用”进行查询。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=1809b29429d84160b9f1)

如代码清单3-32所示，首先反射出DLL里的静态属性，找到内存中的对象，再利用EditorUtility.
CollectDependencies()方法最终递归查找出资源被哪里static强引用了。

代码清单3-32　Script_03_32.cs文件

```
using UnityEngine;
using UnityEditor;
using System.Reflection;
using System;
using System.Collections.Generic;
using System.Text;
using System.Collections;

public class Script_03_32 : Editor
{
    [MenuItem("Tools/Report/脚本Static引用")]
    static void StaticRef()
    {
        //静态引用
        LoadAssembly("Assembly-CSharp-firstpass");
        LoadAssembly("Assembly-CSharp");

    }

    static void LoadAssembly(string name)
    {
        Assembly assembly = null;
        try {
            assembly = Assembly.Load(name);
        }
        catch(Exception ex) {
            Debug.LogWarning(ex.Message);
        }
        finally{
            if(assembly != null) {
                foreach(Type type in assembly.GetTypes()) {
                    try {
                        HashSet<string> assetPaths = new HashSet<string>();
                        FieldInfo[] listFieldInfo = type.GetFields(BindingFlags.
                            Static | BindingFlags.NonPublic | BindingFlags.Public);  
                        foreach(FieldInfo fieldInfo in listFieldInfo) {
                            if(!fieldInfo.FieldType.IsValueType) {
                                SearchProperties(fieldInfo.GetValue(null),
                                    assetPaths);
                            }
                        }
                        if(assetPaths.Count > 0) {
                            StringBuilder sb = new StringBuilder();
                            sb.AppendFormat("{0}.cs\n", type.ToString());
                            foreach(string path in assetPaths) {
                                sb.AppendFormat("\t{0}\n", path);
                            }
                            Debug.Log(sb.ToString());
                        }

                    } catch(Exception ex){
                        Debug.LogWarning(ex.Message);
                    }
                }
            }
        }
    }

    static HashSet<string> SearchProperties(object obj,HashSet<string> assetPaths)
    {    
        if(obj != null) {
            if(obj is UnityEngine.Object) {
                UnityEngine.Object[]depen = EditorUtility.CollectDependencies
                    (new UnityEngine.Object[]{ obj as UnityEngine.Object });
                foreach(var item in depen) {
                    string assetPath = AssetDatabase.GetAssetPath(item);
                    if(!string.IsNullOrEmpty(assetPath)) {
                        if(!assetPaths.Contains(assetPath)) {
                            assetPaths.Add(assetPath);
                        }
                    }
                }
            } else if(obj is IEnumerable) {
                foreach(object child in (obj as IEnumerable)) {
                    SearchProperties(child,assetPaths);
                }
            }else if(obj is System.Object) {
                if(!obj.GetType().IsValueType) {
                    FieldInfo[] fieldInfos = obj.GetType().GetFields();
                    foreach(FieldInfo fieldInfo in fieldInfos) {
                        object o = fieldInfo.GetValue(obj);
                        if(o != obj) {
                            SearchProperties(fieldInfo.GetValue(obj),assetPaths);
                        }
                    }
                }
            }
        }
        return assetPaths;
    }
}
```

上述代码中，反射Assembly-CSharp-firstpass和Assembly-CSharp两个DLL文件，找到static静态对象后，递归查询被哪里static强引用到，最终输入被引用的对象位置。

需要注意的是，这个脚本只能找出代码中的静态引用。但是ManagedStaticReferences也可能是由资源的环形引用造成的。环形引用的意思就是A引用B，B引用C，C引用A，这是一件非常恐怖的事，平时开发中一定要避免这种情况。

### 3.8.7　UIElements

以前的Editor开发编辑器只能使用GUI来开发，但是GUI有很多缺陷，比如显示区域必须写死在代码中，无法配置。而Unity 2017引入了新的概念，那就是UIElements，它可以像CSS一样来布局界面。如图3-39所示，首先创建样式文件，将style.uss文件放入Resources目录下并且用记事本直接打开它，然后就可以配置TextField和Button的大小。UIElements目前还处于实验阶段，未来可能还会进行调整。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=18095cedbbaa2265e034)

在代码中，通过AddStyleSheetPath(style)即可添加这个样式，后面添加的按钮以及文本输入框都会读取style.uss的样式配置，并且还可以设置循环自动换行。目前可以控制的元素包括Button（按钮）、Toggle（勾选框）、Label（文本）、ScrollView（滚动区域）、TextField（输入框）和EditorTextField（编辑文本输入框），如图3-40所示。相关代码如代码清单3-33所示。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=1809a83dae057d04c6d2)

代码清单3-33　Script_03_33.cs文件

```
using UnityEngine;
using UnityEditor;
using UnityEditor.Experimental.UIElements;
using UnityEngine.Experimental.UIElements;
using UnityEngine.Experimental.UIElements.StyleEnums;

public class Script_03_33 : EditorWindow
{    

    [MenuItem("UIElementsTest/Style")]
    public static void ShowExample()
    {
        Script_03_33 window = GetWindow<Script_03_33>();
        window.titleContent = new GUIContent("Script_03_33");
    }

    public void OnEnable()
    {
        var root = this.GetRootVisualContainer();
        //添加style.uss样式
        root.AddStyleSheetPath("style");

        var boxes = new VisualContainer();
        //设置自动换行
        boxes.style.flexDirection = FlexDirection.Row;
        boxes.style.flexWrap = Wrap.Wrap;
        for(int i = 0; i < 20; i++) {
            TextField m_TextField = new TextField();
            boxes.Add(m_TextField);
            Button button = new Button(delegate() {
                Debug.LogFormat("Click");        
            });
            button.text = "我是按钮我要自适应";

            boxes.Add(button);
        }
        root.Add(boxes);
    }
}
```

上述代码中，我们添加USS样式，再创建的TextField和Button元素就会按样式中配置参数的大小显示。

### 3.8.8　查询系统窗口

Unity系统的一些窗口做得很棒，有时需要仿照它来做自定义编辑器。例如，要查看Profiler窗口是如何绘制的，首先需要查询到它的窗口代码写在哪里，然后在编辑器中打开Profiler窗口，在代码中可以输出当前打开的所有窗口：

```
[MenuItem("Tool/GetEditorWindow")]
static void GetEditorWindow()
{
    foreach(var item in Resources.FindObjectsOfTypeAll<EditorWindow>())
    {
        Debug.Log(item.GetType().ToString());
    }
}
```

如图3-41所示，我们查出Profiler窗口的位置在UnityEditor.ProfilerWindow。如图3-42所示，找到代码所在位置，就可以查看它是如何画出来的了，其中核心绘制代码都在OnGUI()方法中。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=18093720552227d306b6)

### 3.8.9　自定义资源导入类型

将文本、FBX模型、MP3音乐等常用类型的资源拖入Unity引擎后即可直接识别，不过其他后缀名的文件Unity是无法识别的。Unity 2018添加了自定义资源导入类型来识别自定义格式的资源。我们先创建一个自定义格式的文件test.yusongmomo，默认情况下Unity是无法识别它的。

如代码清单3-34所示，首先声明[ScriptedImporter(1, "yusongmomo")]（这表示该脚本用于监听后缀名是yusongmomo的自定义文件），接着在OnImportAsset()方法中就可以调用Unity自己的方法来对资源组合赋值了。

代码清单3-34　Script_03_34.cs文件

```
using UnityEngine;
using UnityEditor.Experimental.AssetImporters;
using System.IO;

//监听的后缀名
[ScriptedImporter(1, "yusongmomo")]
public class Script_03_33 : ScriptedImporter
{
    //监听自定义资源导入
    public override void OnImportAsset(AssetImportContext ctx)
    {
        //创建立方体对象
        var cube = GameObject.CreatePrimitive(PrimitiveType.Cube);
        //将参数提取出来
        var position = JsonUtility.FromJson<Vector3>(File.ReadAllText(ctx.assetPath));

        cube.transform.position = position;
        cube.transform.localScale = Vector3.one;
        //将立方体绑定到对象身上
        ctx.AddObjectToAsset("obj", cube);
        ctx.SetMainObject(cube);

        //添加材质
        var material = new Material(Shader.Find("Standard"));
        material.color = Color.red;
        ctx.AddObjectToAsset("material", material);        

        var tempMesh = new Mesh();
        DestroyImmediate(tempMesh);
    }
}
```

在上述代码中，我们创建了立方体对象，并且将从自定义文件test.yusongmomo中获取的坐标信息赋值给它。如图3-43所示，该文件放入引擎中已经变成可识别资源了。

![图像说明文字](http://epub.ituring.com.cn/api/storage/getbykey/screenshow?key=18099f2c74c2dd533dc5)

## 3.9　小结

本章中，我们了解了如何拓展编辑器。编辑器的API非常丰富，可以灵活地自由扩展。学习了EditorGUI，可以用来编辑拓展界面，做出各式各样的编辑器窗口。我们还介绍了Unity的五大常用视图（Project视图、Hierarchy视图、Inspector视图、Scene视图和Game视图）的拓展，以及常用菜单栏的拓展。最后，我向大家介绍了如何查看Unity编辑器的源码。通过阅读源码，可以借鉴Unity内置编辑器的开发思路，这为我们日后开发优秀的编辑器打下良好的基础。
