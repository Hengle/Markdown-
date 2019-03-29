# 简介

本文目标是在 Windows 环境下为 Unity 进行单元测试以提高代码质量、稳定已完成特性和固化已经完成的 bug 修复方案。

Unity 中的测试工具是 [Unity Test Runner](https://docs.unity3d.com/Manual/testing-editortestsrunner.html)， 它基于 NUnit，并增加了 `UnityTestAttribute` 以提供跳过当前帧的功能，这对于涉及到 `Update()` 等生命周期函数的测试非常有用，例如 `GameObject` 的运动测试。

# 环境

* Unity 2018.2.5f1 Personal (64bit)
* Microsoft Windows 10 企业版 10.0.14393 版本 14393

# 环境配置

1.  安装 Unity 2018.1.5f1 Personal (64bit) 即可。官方已经内置所有的工具，不需要其他依赖。

# 使用方法

以下介绍 Edit Mode 和 Play Mode 的测试创建使用方法。

1.  新建项目，在 `Assets` 目录下创建 `Scripts` 文件夹。
2.  在 `Assets/Scripts` 下新建一个 C# 脚本作为待测试模块，命名为 `SampleClass.cs`（此处展示普通类和 Monobehavior 类的测试方法，简便起见放在了一个文件中，实际开发时应该分开，一个类一个独立文件），删除自动生成的内容，复制以下代码到文件中:

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace Sample
{

    public class SampleClass
    {

        public int SampleIntReturn(int input)
        {
            return input * 2;
        }
    }

    public class SampleClassMonoBehavior : MonoBehaviour
    {

        private bool m_IsUpdateNeeded = false;
        // Use this for initialization
        void Start()
        {
            transform.position.Set(0, 0, 0);
        }

        // Update is called once per frame
        void Update()
        {
            if (m_IsUpdateNeeded)
            {
                m_IsUpdateNeeded = false;
                transform.position += new Vector3(1f, 1f, 1f);
            }

        }

        public void TriggerUpdate()
        {
            m_IsUpdateNeeded = true;
        }
    }
}
```

1.  在 `Assets` 目录下创建 `Editor` 文件夹，在其中创建 `Tests` 文件夹。注意测试文件按照 Unity 的约定（参见[手册](https://docs.unity3d.com/Manual/testing-editortestsrunner.html)的 `Testing in Edit mode` 部分），一定要在 `Editor` 名称的目录下（可以是子目录）。
2.  在 `Assets/Editor/Tests` 路径下，右键，`Create/Testing/C# Test Script` ，命名为 `SampleTests.cs`。删除自动生成的内容，复制以下代码到文件中:

```csharp
using UnityEngine;
using UnityEngine.TestTools;
using NUnit.Framework;
using System.Collections;

namespace SampleClass.Tests
{
    public class SampleClassTests
    {

        [Test]
        public void SampleTestsSimplePasses()
        {
            Assert.AreEqual(1, 2);
        }

        [UnityTest]
        public IEnumerator SampleTestsWithEnumeratorPasses()
        {
            yield return null;
        }
    }

    public class SampleClassMonoBehaviorTests
    {
        [Test]
        public void SampleTestsSimplePasses()
        {
            Assert.AreEqual(1, 2);
        }

    }
}
```

1.  此时在 Unity 中点击 `Window/General/Test Runner` 打开 Test Runner 窗口，可以看到我们写的测试已经被 Unity 识别，如图：  
    ![](https://img-blog.csdn.net/20181008170322208?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTM2MTQxMjY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
2.  点击 `Run All`，可以看到 1 个测试通过，2 个测试失败，与我们测试文件所写一致。从界面右侧的数字也可以直接看出通过和失败的数量。  
    ![](https://img-blog.csdn.net/20181008170340274?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTM2MTQxMjY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
3.  接下来模拟一个相对完整的测试，将 `SampleTests.cs` 中的内容替换如下：

```csharp
using UnityEngine;
using UnityEngine.TestTools;
using NUnit.Framework;
using System.Collections;

namespace Sample.Tests
{
    public class SampleClassTests
    {
        SampleClass m_Target;

        [SetUp]
        public void SetUp()
        {
            m_Target = new SampleClass();
        }

        [Test]
        public void SampleTestsSimplePasses()
        {
            Assert.AreEqual(2, m_Target.SampleIntReturn(1));
        }

    }

    public class SampleClassMonoBehaviorTests
    {
        SampleClassMonoBehavior m_Target;
        GameObject m_GO;

        [SetUp]
        public void SetUp()
        {
            m_GO = new GameObject("TestGameObject");
            m_Target = m_GO.AddComponent<SampleClassMonoBehavior>();
        }

        [TearDown]
        public void TearDown()
        {
            Object.DestroyImmediate(m_GO);
        }

        [Test]
        public void ShouldReturnCallCount()
        {
            Assert.AreEqual(1, m_Target.NumOfCall());
        }
        [Test]
        public void ShouldRecordCallCount()
        {
            m_Target.NumOfCall();
            Assert.AreEqual(2, m_Target.NumOfCall());
        }

        [UnityTest]
        public IEnumerator ShouldNotUpdatePosition()
        {
            Assert.AreEqual(new Vector3(0, 0, 0), m_Target.transform.position);
            m_Target.TriggerUpdate();
            Assert.AreEqual(new Vector3(0, 0, 0), m_Target.transform.position);
            yield return null;
            // editor 中等待的是 EditorApplication.update callback loop
            // 所以在这里不会更新位置
            // https://docs.unity3d.com/ScriptReference/EditorApplication-update.html
            Assert.AreEqual(new Vector3(0, 0, 0), m_Target.transform.position);

        }
        [UnityTest]
        public IEnumerator ShouldNotUpdatePositionWithSet()
        {
            Assert.AreEqual(new Vector3(0, 0, 0), m_Target.transform.position);
            m_Target.transform.position.Set(1, 1, 1);
            yield return null;
            Assert.AreEqual(new Vector3(0, 0, 0), m_Target.transform.position);
        }

        [UnityTest]
        public IEnumerator ShouldUpdatePosition()
        {
            Assert.AreEqual(new Vector3(0, 0, 0), m_Target.transform.position);
            m_Target.transform.position = new Vector3(1f, 1f, 1f);
            yield return null;
            Assert.AreEqual(new Vector3(1f, 1f, 1f), m_Target.transform.position);

        }

        [Test]
        public void ShouldUpdatePositionInEditMode()
        {
            Assert.AreEqual(new Vector3(0, 0, 0), m_Target.transform.position);
            m_Target.transform.position = new Vector3(1f, 1f, 1f);
            Assert.AreEqual(new Vector3(1f, 1f, 1f), m_Target.transform.position);

        }
        [Test]
        public void ShouldNotUpdatePositionWithSetInEditMode()
        {
            Assert.AreEqual(new Vector3(0, 0, 0), m_Target.transform.position);
            m_Target.transform.position.Set(1, 1, 1);
            Assert.AreEqual(new Vector3(0, 0, 0), m_Target.transform.position);
        }
    }
}
```

1.  运行结果如下：  
    ![](https://img-blog.csdn.net/20181008170357714?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTM2MTQxMjY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

2.  在 `Assets/Scripts` 目录下创建 `PlayModeTests` 文件夹，在文件夹内右键，`Create/Testing/C# Test Script` ，命名为 `SampleTestPlayMode.cs`。删除自动生成的内容，复制以下代码到文件中:

```csharp
using UnityEngine;
using UnityEngine.TestTools;
using NUnit.Framework;
using System.Collections;

namespace Sample.Tests
{
    public class SampleClassMonoBehaviorPlayModeTests
    {
        SampleClassMonoBehavior m_Target;
        GameObject m_GO;

        [SetUp]
        public void SetUp()
        {
            m_GO = new GameObject("TestGameObject");
            m_Target = m_GO.AddComponent<SampleClassMonoBehavior>();
        }

        [TearDown]
        public void TearDown()
        {
            Object.DestroyImmediate(m_GO);
        }

        [UnityTest]
        public IEnumerator ShouldUpdatePosition()
        {
            Assert.AreEqual(new Vector3(0, 0, 0), m_Target.transform.position);
            m_Target.TriggerUpdate();
            Assert.AreEqual(new Vector3(0, 0, 0), m_Target.transform.position);
            //yield return new WaitForFixedUpdate();
            yield return null;
            Assert.AreEqual(new Vector3(1f, 1f, 1f), m_Target.transform.position);
        }
    }
}
```
1.  在 Test Runner 窗口右上角点击下拉菜单，选中 `Enable play mode tests for all assemblies`，然后按提示重启编辑器。然后在 Test Runner 中点击 `PlayMode` 标签页，运行结果如下：  
    ![](https://img-blog.csdn.net/20181008170411401?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTM2MTQxMjY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

# 示例项目下载

以上文件作为一个项目上传到了 gitee

* https://gitee.com/hustlion-dev/UnityTestRunnerSample

# 关键知识点

* Edit Mode 测试应该放在 `Editor` 目录或其子目录下。
* Play Mode 测试放在非 `Editor` 目录下，建议放 `Assets/PlayModeTests`，并需要在 Test Runners 窗口右侧的下拉菜单中选中 `Enable play mode tests for all assemblies`。注意在发布游戏前要把这个选项关掉，否则测试相关的内容也会被编译到最终文件中。
* 测试函数要标上 `[Test]` 或者 `[UnityTest]` 属性才会被 Test Runner 识别，前者是普通测试，后者具有跳过帧的能力（可使用 `yield return null`，根据测试模式决定是跳 EditorApplication.update 还是 update）。
* `[UnityTest]` 在 Edit Mode 中是跳过 `EditorApplication.update`, 在 PlayMode 中是跳过 Monobehavior 的 Update。要测试涉及到游戏内 Update 相关内容时，必须将测试写为 PlayMode 测试; Edit Mode 的测试只用于与 Update 无关或者与 Editor 的 Update 相关的内容（主要是编辑器插件关注）。

    * 总结：显式或涉及 Update 的测试内容全部需要放在 Play Mode 下，其他的在 Edit Mode 下。

* Play Mode 测试运行时会新建临时场景，运行后自动删除，而 Edit Mode 则不会。
* 使用 `Assert` 类语法对关注的值进行测试，具体可用内容参见 NUnit 文档。
* Unity Test Runner 使用 NUnit 3.5 版本。

> The Unity Test Runner uses a Unity integration of the NUnit library, which is an open-source unit testing library for .NET languages. In Unity 5.6, this integrated library is updated from version 2.6 to ‘‘version 3.5’’. This update has introduced some breaking changes that might affect your existing tests. See the NUnit update guide for more information.  
> https://docs.unity3d.com/Manual/UpgradeGuide56.html

# 参考

* [Unity Test Runner 官方手册](https://docs.unity3d.com/Manual/testing-editortestsrunner.html)

* https://docs.unity3d.com/Manual/testing-editortestsrunner.html

* https://docs.unity3d.com/Manual/PlaymodeTestFramework.html

* [使用NUnit为Unity3D编写高质量单元测试的思考](https://www.gameres.com/668571.html)

* [TDD在Unity3D游戏项目开发中的实践](https://www.gameres.com/497930.html)

* [Unity 单元测试(NUnit,UnityTestTools)](https://www.cnblogs.com/plateFace/p/5176961.html)

* [在Unity中写单元测试](https://blog.csdn.net/kingsea168/article/details/49583083)
