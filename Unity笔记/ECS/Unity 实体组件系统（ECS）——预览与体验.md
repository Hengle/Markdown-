<font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)" class="__reader_view_article_wrap_8602036873257461__"><div align="left">这次给大家带来一期"新技术"的介绍。没错，主角就是Unity官方正在推行的ECS框架（Entity-Component-System）。</div><div align="left">相信大家多少听说过ECS（实体组件系统），或者在网络上查找过相关资料，甚至动手实现过一个自己的简易ECS框架。如果没有听说过也没有关系，可以通过实践可以更好地理解它。</div><br>
<br>
<div align="left">简单介绍一下ECS的核心概念：</div><div align="left">Entity（实体）：由一个唯一ID所标识的一系列组件的集合。</div><div align="left">Component（组件）：一系列数据的集合，本身没有任何方法。只能用于存储状态。</div><div align="left">System（系统）：只有方法，没有状态的工具，类似静态类。</div><div align="left">这种设计看上去很新颖且奇特，具体到游戏开发的环节中是什么样的呢？</div><font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
</font></font></font><br>
<font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
</font></font></font><br>
<br>
<div align="left">简单来说，Entity相当于一个只有唯一ID的GameObject，Component就是一个只有字段的Struct，System只有方法没有任何字段。Entity通过不同Component的组合可以被不同的System关注。</div><br>
<div align="left">如下图所示：</div><div align="center">
<ignore_js_op>

<img style="cursor: pointer;" id="aimg_169547" aid="169547" src="http://img.manew.com/data/attachment/forum/201811/12/120528on02cslmbn8hax5a.jpg.thumb.jpg" onclick="zoom(this, this.getAttribute('zoomfile'), 0, 0, '0')" zoomfile="http://img.manew.com/data/attachment/forum/201811/12/120528on02cslmbn8hax5a.jpg" file="http://img.manew.com/data/attachment/forum/201811/12/120528on02cslmbn8hax5a.jpg.thumb.jpg" inpost="1" onmouseover="showMenu({'ctrlid':this.id,'pos':'12'})" lazyloaded="true" _load="1" initialized="true">

<div class="tip tip_4 aimg_tip" id="aimg_169547_menu" style="position: absolute; z-index: 301; left: 638.5px; top: 1780px; display: none;" disautofocus="true" initialized="true">
<div class="xs0">
<p><strong>4.jpg</strong> <em class="xg1">(86.5 KB, 下载次数: 8)</em></p>
<p>
<a href="http://www.manew.com/forum.php?mod=attachment&amp;aid=MTY5NTQ3fGU2YjllMTEwfDE1NDQ0MDQxNjR8Mjk4NzA5fDE0MTg5MA%3D%3D&amp;nothumb=yes" target="_blank">下载附件</a>

&nbsp;<a href="javascript:;" onclick="showWindow(this.id, this.getAttribute('url'), 'get', 0);" id="savephoto_169547" url="home.php?mod=spacecp&amp;ac=album&amp;op=saveforumphoto&amp;aid=169547&amp;handlekey=savephoto_169547">保存到相册</a>

</p>

<p class="xg1 y">2018-11-12 12:05 上传</p>

</div>
<div class="tip_horn"></div>
</div>

</ignore_js_op>
</div><br>
<br>
<br>
<div align="left">Player（Entity）拥有Position，MoveSpeed，Velocity，Player这些Component。那么他就会被PlayerInputSystem，MoveSystem所关注，这些系统在Update时会对该实体的组件进行读写操作。</div><br>
<br>
<br>
<div align="left">系统的调用顺序也可以打乱。PlayerInputSystem跟AIInputSystem由于写的是不同的实体的Velocity所以可以并行，可以把他们归到一个Group里面。MoveSystem由于需要读取Velocity，所以得等待Group中所有写入操作都完成后才能Update。</div><font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
</font></font></font><br>
<br>
<br>
<div align="left">至于为什么要使用ECS，相信很多熟悉OOP（面向对象编程）的同学开发稍微复杂点的游戏时都遇到过：一大堆类不知道继承哪一个，为了解耦写一大堆管理器等。</div><font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
</font></font></font><br>
<br>
<div align="left">ECS这种反直觉的设计理念在游戏开发中比起OOP有这些显而易见的优点：</div><br>
<ul type="1" class="litype_1"><li>没有大量的管理器或者中间件，简单地说就是避免了OOP中常见的过度抽象。</li><li>比起继承，组合的方式更容易塑造新的实体类型。</li><li></li><li>数据驱动，因为Component没有方法且被统一管理，方便利用Excel配置数据。</li><li>可以利用Utils（工具类）抽出System的共有方法，加上SingletonComponent（单例组件）提供全局访问进行解耦。</li><li><br>
</li></ul><div align="left">ECS在1998年就已经被应用在一款叫做：Thief : The Dark Project 的游戏中。直到2017年在 GDC 2017上的演讲：<a href="http://link.zhihu.com/?target=http%3A//gad.qq.com/article/detail/28682" target="_blank">Overwatch Gameplay Architecture and Netcode</a></div><font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
</font></font></font><br>
<font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
</font></font></font><br>
<br>
<div align="left">守望先锋团队向大家分享了在守望先锋中使用的ECS以及一系列实现上的细节。这下才被广大开发者熟知。</div><font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
</font></font></font><br>
<br>
<div align="left">由于ECS架构的一些特点，他可以很容易利用多个CPU实现逻辑并行，紧凑且连续的内存布局，比起OOP可以更方便地获得更大的性能提升。</div><font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
</font></font></font><br>
<br>
<div align="left">在Unity2018中，伴随Unity ECS推出的还有Burst编译器与C# Job System。下面列出了一部分Unity ECS的愿景：</div><font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
</font></font></font><br>
<br>
<ul type="1" class="litype_1"><li>我们相信我们可以快速编写高性能代码，就像MonoBehaviour.Update一样简单。</li><li>我们相信，在基础层面，这将使Unity比现在更加灵活。</li><li>我们会立即为您提供有关任何竞态条件的错误信息。</li><li>对于小内容，我们希望Unity在不到1秒的时间内加载。</li><li>在大型项目中更改单个.cs文件时。组合编译和热重载时间应小于500毫秒。<br>
</li></ul><br>
<div align="left">Unity ECS现阶段并不推荐直接用于生产，但是了解他的使用方法还是很有用处的，因为ECS不仅可以提高性能，还可以帮助你编写更清晰，更易于维护的代码。</div><div align="left">看到这里有没有很想体验一下Unity的ECS？</div><br>
<br>
<div align="center">
<ignore_js_op>

<img id="aimg_169548" aid="169548" src="http://img.manew.com/data/attachment/forum/201811/12/120720qjmtw1bf7ww5wr7j.gif" zoomfile="http://img.manew.com/data/attachment/forum/201811/12/120720qjmtw1bf7ww5wr7j.gif" file="http://img.manew.com/data/attachment/forum/201811/12/120720qjmtw1bf7ww5wr7j.gif" class="zoom" onclick="zoom(this, this.src, 0, 0, 0)" width="200" inpost="1" onmouseover="showMenu({'ctrlid':this.id,'pos':'12'})" lazyloaded="true" _load="1" style="">

<div class="tip tip_4 aimg_tip" id="aimg_169548_menu" style="position: absolute; display: none" disautofocus="true">
<div class="xs0">
<p><strong>1.1.gif</strong> <em class="xg1">(537.53 KB, 下载次数: 8)</em></p>
<p>
<a href="http://www.manew.com/forum.php?mod=attachment&amp;aid=MTY5NTQ4fDczMDAzOTMwfDE1NDQ0MDQxNjR8Mjk4NzA5fDE0MTg5MA%3D%3D&amp;nothumb=yes" target="_blank">下载附件</a>

&nbsp;<a href="javascript:;" onclick="showWindow(this.id, this.getAttribute('url'), 'get', 0);" id="savephoto_169548" url="home.php?mod=spacecp&amp;ac=album&amp;op=saveforumphoto&amp;aid=169548&amp;handlekey=savephoto_169548">保存到相册</a>

</p>

<p class="xg1 y">2018-11-12 12:07 上传</p>

</div>
<div class="tip_horn"></div>
</div>

</ignore_js_op>
<br>
</div><font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
</font></font></font><br>
<br>
<br>
<div align="left">下面我们就来写一些Unity ECS-Style风格的代码。</div><br>
<div align="left">首先我们下载一个Unity 2018.X，新建一个工程在Window -&gt; Package Manager 中选择Advanced -&gt; Show Preview Packages，然后选择Entities并点击Install。</div><br>
<br>
<div align="left">（在2018.1中点击All可以看到Entities）</div><div align="center">
<ignore_js_op>

<img style="cursor: pointer;" id="aimg_169549" aid="169549" src="http://img.manew.com/data/attachment/forum/201811/12/120743c4yfz6yzyyyyzc46.jpg.thumb.jpg" onclick="zoom(this, this.getAttribute('zoomfile'), 0, 0, '0')" zoomfile="http://img.manew.com/data/attachment/forum/201811/12/120743c4yfz6yzyyyyzc46.jpg" file="http://img.manew.com/data/attachment/forum/201811/12/120743c4yfz6yzyyyyzc46.jpg.thumb.jpg" inpost="1" onmouseover="showMenu({'ctrlid':this.id,'pos':'12'})" lazyloaded="true" _load="1">

<div class="tip tip_4 aimg_tip" id="aimg_169549_menu" style="position: absolute; display: none" disautofocus="true">
<div class="xs0">
<p><strong>5.jpg</strong> <em class="xg1">(2.27 KB, 下载次数: 8)</em></p>
<p>
<a href="http://www.manew.com/forum.php?mod=attachment&amp;aid=MTY5NTQ5fDg1OGJhNmQzfDE1NDQ0MDQxNjR8Mjk4NzA5fDE0MTg5MA%3D%3D&amp;nothumb=yes" target="_blank">下载附件</a>

&nbsp;<a href="javascript:;" onclick="showWindow(this.id, this.getAttribute('url'), 'get', 0);" id="savephoto_169549" url="home.php?mod=spacecp&amp;ac=album&amp;op=saveforumphoto&amp;aid=169549&amp;handlekey=savephoto_169549">保存到相册</a>

</p>

<p class="xg1 y">2018-11-12 12:07 上传</p>

</div>
<div class="tip_horn"></div>
</div>

</ignore_js_op>
</div><br>
<div align="center"><font color="#1a1a1a">看到Jobs出现就说明Entities已经安装完毕</font></div><br>
<div align="left">准备就绪后，我们直接创建一个脚本并命名Bootstrap。</div><br>
<br>
<div style="padding:15px 0;"><div style="font-size:12px;">[AppleScript] <em class="viewsource" style="cursor:pointer;font-size:12px;color:#369 !important;" num="0">纯文本查看</em> <em class="copycode" style="cursor:pointer;font-size:12px;color:#369 !important;" num="0">复制代码</em></div><div><div id="highlighter_348573" class="syntaxhighlighter notranslate applescript"><div class="toolbar"><span><a href="#" class="toolbar_item command_help help">?</a></span></div><table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td class="gutter"><div class="line number0 index0 alt1 break">&nbsp;</div><div class="line number1 index1 alt2">01</div><div class="line number2 index2 alt1">02</div><div class="line number3 index3 alt2">03</div><div class="line number4 index4 alt1">04</div><div class="line number5 index5 alt2">05</div><div class="line number6 index6 alt1">06</div><div class="line number7 index7 alt2">07</div><div class="line number8 index8 alt1">08</div><div class="line number9 index9 alt2">09</div><div class="line number10 index10 alt1">10</div><div class="line number11 index11 alt2">11</div><div class="line number12 index12 alt1">12</div><div class="line number13 index13 alt2">13</div><div class="line number14 index14 alt1">14</div><div class="line number15 index15 alt2">15</div><div class="line number16 index16 alt1">16</div><div class="line number17 index17 alt2">17</div><div class="line number18 index18 alt1">18</div></td><td class="code"><div class="container"><div class="line number0 index0 alt1 break"><code class="applescript color3">using</code> <code class="applescript plain">System.Collections;</code></div><div class="line number1 index1 alt2"><code class="applescript color3">using</code> <code class="applescript plain">System.Collections.Generic;</code></div><div class="line number2 index2 alt1"><code class="applescript color3">using</code> <code class="applescript plain">UnityEngine;</code></div><div class="line number3 index3 alt2"><code class="applescript color3">using</code> <code class="applescript plain">Unity.Entities;</code></div><div class="line number4 index4 alt1">&nbsp;</div><div class="line number5 index5 alt2"><code class="applescript plain">public </code><code class="applescript color3">class</code> <code class="applescript plain">Bootstrap</code></div><div class="line number6 index6 alt1"><code class="applescript color1">{</code></div><div class="line number7 index7 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">[RuntimeInitializeOnLoadMethod</code><code class="applescript color1">(</code><code class="applescript plain">RuntimeInitializeLoadType.BeforeSceneLoad</code><code class="applescript color1">)</code><code class="applescript plain">]</code></div><div class="line number8 index8 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">public static void Awake</code><code class="applescript color1">(</code><code class="applescript color1">)</code></div><div class="line number9 index9 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">{</code></div><div class="line number10 index10 alt1">&nbsp;</div><div class="line number11 index11 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">}</code></div><div class="line number12 index12 alt1">&nbsp;</div><div class="line number13 index13 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">[RuntimeInitializeOnLoadMethod</code><code class="applescript color1">(</code><code class="applescript plain">RuntimeInitializeLoadType.AfterSceneLoad</code><code class="applescript color1">)</code><code class="applescript plain">]</code></div><div class="line number14 index14 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">public static void Start</code><code class="applescript color1">(</code><code class="applescript color1">)</code></div><div class="line number15 index15 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">{</code></div><div class="line number16 index16 alt1">&nbsp;</div><div class="line number17 index17 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">}</code></div><div class="line number18 index18 alt1"><code class="applescript color1">}</code></div></div></td></tr></tbody></table></div></div></div></font></font></font>

<font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
</font></font></font>  

<font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
<div align="left">ECS不同于以往的Monobehavior，他有一套自己的生命周期。</div><br>
<br>
<blockquote>代码中的Awake跟Start方法都是我自己编写的，只要给他们打上一个特性就会被Unity在场景加载的前后时机进行调用。（方法必须为静态方法）</blockquote><div align="left">我们在场景中创建一个空物体并命名Player，加上Mesh Instance Renderer Component（渲染组件）。</div><br>
<br>
<div align="center">
<ignore_js_op>

<img style="cursor: pointer;" id="aimg_169550" aid="169550" src="http://img.manew.com/data/attachment/forum/201811/12/120816jo1q0rz91al0ca0z.jpg.thumb.jpg" onclick="zoom(this, this.getAttribute('zoomfile'), 0, 0, '0')" zoomfile="http://img.manew.com/data/attachment/forum/201811/12/120816jo1q0rz91al0ca0z.jpg" file="http://img.manew.com/data/attachment/forum/201811/12/120816jo1q0rz91al0ca0z.jpg.thumb.jpg" inpost="1" onmouseover="showMenu({'ctrlid':this.id,'pos':'12'})" lazyloaded="true" _load="1" initialized="true">

<div class="tip tip_4 aimg_tip" id="aimg_169550_menu" style="position: absolute; z-index: 301; left: 644px; top: 5011px; display: none;" disautofocus="true" initialized="true">
<div class="xs0">
<p><strong>6.jpg</strong> <em class="xg1">(24.93 KB, 下载次数: 7)</em></p>
<p>
<a href="http://www.manew.com/forum.php?mod=attachment&amp;aid=MTY5NTUwfDBhOGNlYzU1fDE1NDQ0MDQxNjR8Mjk4NzA5fDE0MTg5MA%3D%3D&amp;nothumb=yes" target="_blank">下载附件</a>

&nbsp;<a href="javascript:;" onclick="showWindow(this.id, this.getAttribute('url'), 'get', 0);" id="savephoto_169550" url="home.php?mod=spacecp&amp;ac=album&amp;op=saveforumphoto&amp;aid=169550&amp;handlekey=savephoto_169550">保存到相册</a>

</p>

<p class="xg1 y">2018-11-12 12:08 上传</p>

</div>
<div class="tip_horn"></div>
</div>

</ignore_js_op>
</div><br>
<br>
<br>
<div align="left">Mesh选择球形，新建一个材质球Red并且放在Material中，再把Cast Shadows（投影）设置为开启。</div><font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
</font></font></font><br>
<font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
</font></font></font><br>
<br>
<div align="left">打开Bootstrap脚本，在Awake中创建出EntityManager与EntityArchetype：</div><br>
<br>
&nbsp; &nbsp;<div style="padding:15px 0;"><div style="font-size:12px;">[AppleScript] <em class="viewsource" style="cursor:pointer;font-size:12px;color:#369 !important;" num="1">纯文本查看</em> <em class="copycode" style="cursor:pointer;font-size:12px;color:#369 !important;" num="1">复制代码</em></div><div><div id="highlighter_588417" class="syntaxhighlighter notranslate applescript"><div class="toolbar"><span><a href="#" class="toolbar_item command_help help">?</a></span></div><table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td class="gutter"><div class="line number0 index0 alt1 break">&nbsp;</div><div class="line number1 index1 alt2">01</div><div class="line number2 index2 alt1">02</div><div class="line number3 index3 alt2">03</div><div class="line number4 index4 alt1">04</div><div class="line number5 index5 alt2">05</div><div class="line number6 index6 alt1">06</div><div class="line number7 index7 alt2">07</div><div class="line number8 index8 alt1">08</div><div class="line number9 index9 alt2">09</div><div class="line number10 index10 alt1">10</div><div class="line number11 index11 alt2">11</div></td><td class="code"><div class="container"><div class="line number0 index0 alt1 break"><code class="applescript plain">private static EntityManager entityManager;&nbsp;&nbsp;&nbsp;&nbsp; </code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">所有实体的管理器</code><code class="applescript color1">,</code> <code class="applescript plain">提供操作Entity的API</code></div><div class="line number1 index1 alt2">&nbsp;</div><div class="line number2 index2 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">private static EntityArchetype playerArchetype; </code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">Entity原型</code><code class="applescript color1">,</code> <code class="applescript plain">可以看成由组件组成的数组</code></div><div class="line number3 index3 alt2">&nbsp;</div><div class="line number4 index4 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">[RuntimeInitializeOnLoadMethod</code><code class="applescript color1">(</code><code class="applescript plain">loadType</code><code class="applescript color1">:</code> <code class="applescript plain">RuntimeInitializeLoadType.BeforeSceneLoad</code><code class="applescript color1">)</code><code class="applescript plain">]</code></div><div class="line number5 index5 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">public static void Awake</code><code class="applescript color1">(</code><code class="applescript color1">)</code></div><div class="line number6 index6 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">{</code></div><div class="line number7 index7 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">entityManager </code><code class="applescript color2">=</code> <code class="applescript plain">World.Active.GetOrCreateManager</code><code class="applescript color2">&lt;</code><code class="applescript plain">EntityManager</code><code class="applescript color2">&gt;</code><code class="applescript color1">(</code><code class="applescript color1">)</code><code class="applescript plain">;</code></div><div class="line number8 index8 alt1">&nbsp;</div><div class="line number9 index9 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">下面的的Position类型需要引入Unity.Transforms命名空间</code></div><div class="line number10 index10 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">playerArchetype </code><code class="applescript color2">=</code> <code class="applescript plain">entityManager.CreateArchetype</code><code class="applescript color1">(</code><code class="applescript plain">typeof</code><code class="applescript color1">(</code><code class="applescript plain">Position</code><code class="applescript color1">)</code><code class="applescript color1">)</code><code class="applescript plain">;</code></div><div class="line number11 index11 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">}</code></div></div></td></tr></tbody></table></div></div></div><br>
<div align="left">通过World创建出EntityManager，EntityManager的对象提供了创建实体，给实体添加组件，获取组件，移除组件，实例化与销毁实体等功能。</div><br>
<blockquote>按照Unity的说法，默认情况下会在进入播放模式时创建好World，因此我们直接在Awake使用World创建EntityManager就好了。</blockquote><div align="left">所以EntityManager就是一个实体的管理器。</div><br>
<br>
<br>
<blockquote>上个版本的Unity ECS还是静态类，现在已经该为由World创建的实例了。</blockquote><div align="left">等待场景加载完成之后会调用Start方法：</div><br>
<br>
<div style="padding:15px 0;"><div style="font-size:12px;">[AppleScript] <em class="viewsource" style="cursor:pointer;font-size:12px;color:#369 !important;" num="2">纯文本查看</em> <em class="copycode" style="cursor:pointer;font-size:12px;color:#369 !important;" num="2">复制代码</em></div><div><div id="highlighter_712721" class="syntaxhighlighter notranslate applescript"><div class="toolbar"><span><a href="#" class="toolbar_item command_help help">?</a></span></div><table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td class="gutter"><div class="line number0 index0 alt1 break">&nbsp;</div><div class="line number1 index1 alt2">01</div><div class="line number2 index2 alt1">02</div><div class="line number3 index3 alt2">03</div><div class="line number4 index4 alt1">04</div><div class="line number5 index5 alt2">05</div><div class="line number6 index6 alt1">06</div><div class="line number7 index7 alt2">07</div><div class="line number8 index8 alt1">08</div><div class="line number9 index9 alt2">09</div><div class="line number10 index10 alt1">10</div><div class="line number11 index11 alt2">11</div><div class="line number12 index12 alt1">12</div><div class="line number13 index13 alt2">13</div><div class="line number14 index14 alt1">14</div><div class="line number15 index15 alt2">15</div><div class="line number16 index16 alt1">16</div><div class="line number17 index17 alt2">17</div><div class="line number18 index18 alt1">18</div><div class="line number19 index19 alt2">19</div><div class="line number20 index20 alt1">20</div><div class="line number21 index21 alt2">21</div></td><td class="code"><div class="container"><div class="line number0 index0 alt1 break"><code class="applescript plain">[RuntimeInitializeOnLoadMethod</code><code class="applescript color1">(</code><code class="applescript plain">loadType</code><code class="applescript color1">:</code> <code class="applescript plain">RuntimeInitializeLoadType.AfterSceneLoad</code><code class="applescript color1">)</code><code class="applescript plain">]</code></div><div class="line number1 index1 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">public static void Start</code><code class="applescript color1">(</code><code class="applescript color1">)</code></div><div class="line number2 index2 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">{</code></div><div class="line number3 index3 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">把GameObect.Find放在这里因为场景加载完成前无法获取游戏物体。</code></div><div class="line number4 index4 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">GameObject playerGo </code><code class="applescript color2">=</code> <code class="applescript plain">GameObject.Find</code><code class="applescript color1">(</code><code class="applescript string">"Player"</code><code class="applescript color1">)</code><code class="applescript plain">;</code></div><div class="line number5 index5 alt2">&nbsp;</div><div class="line number6 index6 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">下面的类型是一个Struct</code><code class="applescript color1">,</code> <code class="applescript plain">需要引入Unity.Rendering命名空间</code></div><div class="line number7 index7 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">MeshInstanceRenderer playerRenderer </code><code class="applescript color2">=</code></div><div class="line number8 index8 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">playerGo.GetComponent</code><code class="applescript color2">&lt;</code><code class="applescript plain">MeshInstanceRendererComponent</code><code class="applescript color2">&gt;</code><code class="applescript color1">(</code><code class="applescript color1">)</code><code class="applescript plain">.Value;</code></div><div class="line number9 index9 alt2">&nbsp;</div><div class="line number10 index10 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">获取到渲染数据后可以销毁空物体</code></div><div class="line number11 index11 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">Object.Destroy</code><code class="applescript color1">(</code><code class="applescript plain">playerGo</code><code class="applescript color1">)</code><code class="applescript plain">; </code></div><div class="line number12 index12 alt1">&nbsp;</div><div class="line number13 index13 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">Entity player </code><code class="applescript color2">=</code> <code class="applescript plain">entityManager.CreateEntity</code><code class="applescript color1">(</code><code class="applescript plain">playerArchetype</code><code class="applescript color1">)</code><code class="applescript plain">;</code></div><div class="line number14 index14 alt1">&nbsp;</div><div class="line number15 index15 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">修改实体的Position组件</code></div><div class="line number16 index16 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">entityManager.SetComponentData</code><code class="applescript color1">(</code><code class="applescript plain">player</code><code class="applescript color1">,</code> <code class="applescript color3">new</code> <code class="applescript plain">Position </code></div><div class="line number17 index17 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">{</code> <code class="applescript plain">Value </code><code class="applescript color2">=</code> <code class="applescript color3">new</code> <code class="applescript plain">Unity.Mathematics.float</code><code class="applescript color1">3</code><code class="applescript color1">(</code><code class="applescript color1">0</code><code class="applescript color1">,</code> <code class="applescript color1">2</code><code class="applescript color1">,</code> <code class="applescript color1">0</code><code class="applescript color1">)</code> <code class="applescript color1">}</code><code class="applescript color1">)</code><code class="applescript plain">;</code></div><div class="line number18 index18 alt1">&nbsp;</div><div class="line number19 index19 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color2">/</code><code class="applescript color2">/</code> <code class="applescript plain">向实体添加共享数据组件</code></div><div class="line number20 index20 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">entityManager.AddSharedComponentData</code><code class="applescript color1">(</code><code class="applescript plain">player</code><code class="applescript color1">,</code> <code class="applescript plain">playerRenderer</code><code class="applescript color1">)</code><code class="applescript plain">;</code></div><div class="line number21 index21 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">}</code></div></div></td></tr></tbody></table></div></div></div><br>
<blockquote>float3 是Unity新推出的数学库(Unity.Mathematics)中的类型，用法跟Vector3基本一致。<br>
Unity建议在ECS中使用该数学库。</blockquote><div align="left">通过entityManager对象创建的实体都会被管理起来，在创建Entity时我们可以在CreateEntity方法的参数中填上之前创建好的playerArchetype（玩家原型），即按照原型中包含的组件依次添加到实体上。</div><br>
<div align="left">在Update方法中获取了之前在场景中设置的渲染组件，并且作为AddSharedComponentData的参数。</div><br>
<div align="left">这时我们的player实体已经拥有了两个组件：Position跟MeshInstanceRenderer。</div><br>
<blockquote>关于ISharedComponentData接口：他的作用是当实体拥有属性时，比如：球形的实体共享同样的Mesh网格数据时。这些数据会储存在一个Chuck中并非每个实体上，因此在每个实体上可以实现0内存开销。</blockquote><div align="left">值得注意的是：如果Entity不包含Position组件，这个实体是不会被Unity的渲染系统关注的。因此想在屏幕上看见这个实体，必须确保MeshInstanceRenderer跟Position都添加到了实体上。</div><br>
<br>
<div align="left">我们运行游戏就会看到我们创建的Player被显示出来了：</div><div align="center">
<ignore_js_op>

<img style="cursor: pointer;" id="aimg_169551" aid="169551" src="http://img.manew.com/data/attachment/forum/201811/12/120941abz88w5k4har4o8r.png.thumb.jpg" onclick="zoom(this, this.getAttribute('zoomfile'), 0, 0, '0')" zoomfile="http://img.manew.com/data/attachment/forum/201811/12/120941abz88w5k4har4o8r.png" file="http://img.manew.com/data/attachment/forum/201811/12/120941abz88w5k4har4o8r.png.thumb.jpg" inpost="1" onmouseover="showMenu({'ctrlid':this.id,'pos':'12'})" lazyloaded="true" _load="1" initialized="true">

<div class="tip tip_4 aimg_tip" id="aimg_169551_menu" style="position: absolute; z-index: 301; left: 604px; top: 7483px; display: none;" disautofocus="true" initialized="true">
<div class="xs0">
<p><strong>7.png</strong> <em class="xg1">(88.21 KB, 下载次数: 7)</em></p>
<p>
<a href="http://www.manew.com/forum.php?mod=attachment&amp;aid=MTY5NTUxfGRmOGQ1ZTdmfDE1NDQ0MDQxNjR8Mjk4NzA5fDE0MTg5MA%3D%3D&amp;nothumb=yes" target="_blank">下载附件</a>

&nbsp;<a href="javascript:;" onclick="showWindow(this.id, this.getAttribute('url'), 'get', 0);" id="savephoto_169551" url="home.php?mod=spacecp&amp;ac=album&amp;op=saveforumphoto&amp;aid=169551&amp;handlekey=savephoto_169551">保存到相册</a>

</p>

<p class="xg1 y">2018-11-12 12:09 上传</p>

</div>
<div class="tip_horn"></div>
</div>

</ignore_js_op>
</div><br>
<br>
<div align="left">细心的你也发现了，在Hierarchy中并没有这个实体的信息。</div><br>
<div align="center">
<ignore_js_op>

<img style="cursor: pointer;" id="aimg_169552" aid="169552" src="http://img.manew.com/data/attachment/forum/201811/12/121004t769ii9nvcieonoi.jpg.thumb.jpg" onclick="zoom(this, this.getAttribute('zoomfile'), 0, 0, '0')" zoomfile="http://img.manew.com/data/attachment/forum/201811/12/121004t769ii9nvcieonoi.jpg" file="http://img.manew.com/data/attachment/forum/201811/12/121004t769ii9nvcieonoi.jpg.thumb.jpg" inpost="1" onmouseover="showMenu({'ctrlid':this.id,'pos':'12'})" lazyloaded="true" _load="1">

<div class="tip tip_4 aimg_tip" id="aimg_169552_menu" style="position: absolute; display: none" disautofocus="true">
<div class="xs0">
<p><strong>8.jpg</strong> <em class="xg1">(6.92 KB, 下载次数: 6)</em></p>
<p>
<a href="http://www.manew.com/forum.php?mod=attachment&amp;aid=MTY5NTUyfGFlOWU2MzhkfDE1NDQ0MDQxNjR8Mjk4NzA5fDE0MTg5MA%3D%3D&amp;nothumb=yes" target="_blank">下载附件</a>

&nbsp;<a href="javascript:;" onclick="showWindow(this.id, this.getAttribute('url'), 'get', 0);" id="savephoto_169552" url="home.php?mod=spacecp&amp;ac=album&amp;op=saveforumphoto&amp;aid=169552&amp;handlekey=savephoto_169552">保存到相册</a>

</p>

<p class="xg1 y">2018-11-12 12:10 上传</p>

</div>
<div class="tip_horn"></div>
</div>

</ignore_js_op>
</div><br>
<div align="left">原因是现在Unity编辑器还没有与ECS整合，因此我们需要打开Window -&gt; Analysis -&gt; Entity Debugger面板查看我们的系统与实体。</div><br>
<div align="center">
<ignore_js_op>

<img style="cursor: pointer;" id="aimg_169553" aid="169553" src="http://img.manew.com/data/attachment/forum/201811/12/121023lx4x0dnxp2ix0vfc.jpg.thumb.jpg" onclick="zoom(this, this.getAttribute('zoomfile'), 0, 0, '0')" zoomfile="http://img.manew.com/data/attachment/forum/201811/12/121023lx4x0dnxp2ix0vfc.jpg" file="http://img.manew.com/data/attachment/forum/201811/12/121023lx4x0dnxp2ix0vfc.jpg.thumb.jpg" inpost="1" onmouseover="showMenu({'ctrlid':this.id,'pos':'12'})" lazyloaded="true" _load="1">

<div class="tip tip_4 aimg_tip" id="aimg_169553_menu" style="position: absolute; display: none" disautofocus="true">
<div class="xs0">
<p><strong>9.jpg</strong> <em class="xg1">(12.73 KB, 下载次数: 7)</em></p>
<p>
<a href="http://www.manew.com/forum.php?mod=attachment&amp;aid=MTY5NTUzfDQxMGRiNGY3fDE1NDQ0MDQxNjR8Mjk4NzA5fDE0MTg5MA%3D%3D&amp;nothumb=yes" target="_blank">下载附件</a>

&nbsp;<a href="javascript:;" onclick="showWindow(this.id, this.getAttribute('url'), 'get', 0);" id="savephoto_169553" url="home.php?mod=spacecp&amp;ac=album&amp;op=saveforumphoto&amp;aid=169553&amp;handlekey=savephoto_169553">保存到相册</a>

</p>

<p class="xg1 y">2018-11-12 12:10 上传</p>

</div>
<div class="tip_horn"></div>
</div>

</ignore_js_op>
</div><br>
<br>
<div align="left">在EntityManager中可以看到Entity 0，那就是我们创建的player实体。此时他的Inspector菜单也会有数据填充：</div><br>
<div align="center">
<ignore_js_op>

<img style="cursor: pointer;" id="aimg_169554" aid="169554" src="http://img.manew.com/data/attachment/forum/201811/12/121046mu4g06p2kkekppzg.jpg.thumb.jpg" onclick="zoom(this, this.getAttribute('zoomfile'), 0, 0, '0')" zoomfile="http://img.manew.com/data/attachment/forum/201811/12/121046mu4g06p2kkekppzg.jpg" file="http://img.manew.com/data/attachment/forum/201811/12/121046mu4g06p2kkekppzg.jpg.thumb.jpg" inpost="1" onmouseover="showMenu({'ctrlid':this.id,'pos':'12'})" lazyloaded="true" _load="1">

<div class="tip tip_4 aimg_tip" id="aimg_169554_menu" style="position: absolute; display: none" disautofocus="true">
<div class="xs0">
<p><strong>11.jpg</strong> <em class="xg1">(20.01 KB, 下载次数: 6)</em></p>
<p>
<a href="http://www.manew.com/forum.php?mod=attachment&amp;aid=MTY5NTU0fDI5YTJmYTA1fDE1NDQ0MDQxNjR8Mjk4NzA5fDE0MTg5MA%3D%3D&amp;nothumb=yes" target="_blank">下载附件</a>

&nbsp;<a href="javascript:;" onclick="showWindow(this.id, this.getAttribute('url'), 'get', 0);" id="savephoto_169554" url="home.php?mod=spacecp&amp;ac=album&amp;op=saveforumphoto&amp;aid=169554&amp;handlekey=savephoto_169554">保存到相册</a>

</p>

<p class="xg1 y">2018-11-12 12:10 上传</p>

</div>
<div class="tip_horn"></div>
</div>

</ignore_js_op>
</div><br>
<div align="left">每个Value对应一个Component及其具体的值。可以看到他的坐标，Mesh与Material都被更改了。</div><br>
<br>
<div align="left">现在我们搭建一个简易场景，首先创建一个Plane并命名为Ground。然后创建一个灰色的材质球挂上去：</div><br>
<div align="center">
<ignore_js_op>

<img style="cursor: pointer;" id="aimg_169555" aid="169555" src="http://img.manew.com/data/attachment/forum/201811/12/121117rex9bgtd2dfkduxe.png.thumb.jpg" onclick="zoom(this, this.getAttribute('zoomfile'), 0, 0, '0')" zoomfile="http://img.manew.com/data/attachment/forum/201811/12/121117rex9bgtd2dfkduxe.png" file="http://img.manew.com/data/attachment/forum/201811/12/121117rex9bgtd2dfkduxe.png.thumb.jpg" inpost="1" onmouseover="showMenu({'ctrlid':this.id,'pos':'12'})" lazyloaded="true" _load="1">

<div class="tip tip_4 aimg_tip" id="aimg_169555_menu" style="position: absolute; display: none" disautofocus="true">
<div class="xs0">
<p><strong>12.png</strong> <em class="xg1">(127.65 KB, 下载次数: 7)</em></p>
<p>
<a href="http://www.manew.com/forum.php?mod=attachment&amp;aid=MTY5NTU1fGE3NDk2Y2NkfDE1NDQ0MDQxNjR8Mjk4NzA5fDE0MTg5MA%3D%3D&amp;nothumb=yes" target="_blank">下载附件</a>

&nbsp;<a href="javascript:;" onclick="showWindow(this.id, this.getAttribute('url'), 'get', 0);" id="savephoto_169555" url="home.php?mod=spacecp&amp;ac=album&amp;op=saveforumphoto&amp;aid=169555&amp;handlekey=savephoto_169555">保存到相册</a>

</p>

<p class="xg1 y">2018-11-12 12:11 上传</p>

</div>
<div class="tip_horn"></div>
</div>

</ignore_js_op>
</div><br>
<div align="left">同时保持他的默认组件就好了：</div><br>
<div align="center">
<ignore_js_op>

<img style="cursor: pointer;" id="aimg_169556" aid="169556" src="http://img.manew.com/data/attachment/forum/201811/12/121145aez72yf26izpeeua.jpg.thumb.jpg" onclick="zoom(this, this.getAttribute('zoomfile'), 0, 0, '0')" zoomfile="http://img.manew.com/data/attachment/forum/201811/12/121145aez72yf26izpeeua.jpg" file="http://img.manew.com/data/attachment/forum/201811/12/121145aez72yf26izpeeua.jpg.thumb.jpg" inpost="1" onmouseover="showMenu({'ctrlid':this.id,'pos':'12'})" lazyloaded="true" _load="1">

<div class="tip tip_4 aimg_tip" id="aimg_169556_menu" style="position: absolute; display: none" disautofocus="true">
<div class="xs0">
<p><strong>13.jpg</strong> <em class="xg1">(44.78 KB, 下载次数: 6)</em></p>
<p>
<a href="http://www.manew.com/forum.php?mod=attachment&amp;aid=MTY5NTU2fGZlZWQzYzcyfDE1NDQ0MDQxNjR8Mjk4NzA5fDE0MTg5MA%3D%3D&amp;nothumb=yes" target="_blank">下载附件</a>

&nbsp;<a href="javascript:;" onclick="showWindow(this.id, this.getAttribute('url'), 'get', 0);" id="savephoto_169556" url="home.php?mod=spacecp&amp;ac=album&amp;op=saveforumphoto&amp;aid=169556&amp;handlekey=savephoto_169556">保存到相册</a>

</p>

<p class="xg1 y">2018-11-12 12:11 上传</p>

</div>
<div class="tip_horn"></div>
</div>

</ignore_js_op>
</div><br>
<br>
<br>
<div align="left">运行游戏看一下效果：</div><br>
<br>
<div align="center">
<ignore_js_op>

<img style="cursor: pointer;" id="aimg_169557" aid="169557" src="http://img.manew.com/data/attachment/forum/201811/12/121218iy099a95pnatz25t.png.thumb.jpg" onclick="zoom(this, this.getAttribute('zoomfile'), 0, 0, '0')" zoomfile="http://img.manew.com/data/attachment/forum/201811/12/121218iy099a95pnatz25t.png" file="http://img.manew.com/data/attachment/forum/201811/12/121218iy099a95pnatz25t.png.thumb.jpg" inpost="1" onmouseover="showMenu({'ctrlid':this.id,'pos':'12'})" lazyloaded="true" _load="1">

<div class="tip tip_4 aimg_tip" id="aimg_169557_menu" style="position: absolute; display: none" disautofocus="true">
<div class="xs0">
<p><strong>14.png</strong> <em class="xg1">(105.69 KB, 下载次数: 6)</em></p>
<p>
<a href="http://www.manew.com/forum.php?mod=attachment&amp;aid=MTY5NTU3fDg5OTNlZjQxfDE1NDQ0MDQxNjR8Mjk4NzA5fDE0MTg5MA%3D%3D&amp;nothumb=yes" target="_blank">下载附件</a>

&nbsp;<a href="javascript:;" onclick="showWindow(this.id, this.getAttribute('url'), 'get', 0);" id="savephoto_169557" url="home.php?mod=spacecp&amp;ac=album&amp;op=saveforumphoto&amp;aid=169557&amp;handlekey=savephoto_169557">保存到相册</a>

</p>

<p class="xg1 y">2018-11-12 12:12 上传</p>

</div>
<div class="tip_horn"></div>
</div>

</ignore_js_op>
</div><br>
<br>
<br>
<div align="left">我们只需要修改摄像机的Transform就能让游戏画面呈现出俯视角的效果：</div><br>
<div align="center">
<ignore_js_op>

<img style="cursor: pointer;" id="aimg_169558" aid="169558" src="http://img.manew.com/data/attachment/forum/201811/12/121232woqmqnmmlf7dzkfb.jpg.thumb.jpg" onclick="zoom(this, this.getAttribute('zoomfile'), 0, 0, '0')" zoomfile="http://img.manew.com/data/attachment/forum/201811/12/121232woqmqnmmlf7dzkfb.jpg" file="http://img.manew.com/data/attachment/forum/201811/12/121232woqmqnmmlf7dzkfb.jpg.thumb.jpg" inpost="1" onmouseover="showMenu({'ctrlid':this.id,'pos':'12'})" lazyloaded="true" _load="1">

<div class="tip tip_4 aimg_tip" id="aimg_169558_menu" style="position: absolute; display: none" disautofocus="true">
<div class="xs0">
<p><strong>15.jpg</strong> <em class="xg1">(11.8 KB, 下载次数: 6)</em></p>
<p>
<a href="http://www.manew.com/forum.php?mod=attachment&amp;aid=MTY5NTU4fGJkOTk2YzM4fDE1NDQ0MDQxNjR8Mjk4NzA5fDE0MTg5MA%3D%3D&amp;nothumb=yes" target="_blank">下载附件</a>

&nbsp;<a href="javascript:;" onclick="showWindow(this.id, this.getAttribute('url'), 'get', 0);" id="savephoto_169558" url="home.php?mod=spacecp&amp;ac=album&amp;op=saveforumphoto&amp;aid=169558&amp;handlekey=savephoto_169558">保存到相册</a>

</p>

<p class="xg1 y">2018-11-12 12:12 上传</p>

</div>
<div class="tip_horn"></div>
</div>

</ignore_js_op>
</div><br>
<div align="left">视角调的还不错：</div><br>
<br>
<div align="center">
<ignore_js_op>

<img style="cursor: pointer;" id="aimg_169559" aid="169559" src="http://img.manew.com/data/attachment/forum/201811/12/121246qe07le3eqqaogov1.jpg.thumb.jpg" onclick="zoom(this, this.getAttribute('zoomfile'), 0, 0, '0')" zoomfile="http://img.manew.com/data/attachment/forum/201811/12/121246qe07le3eqqaogov1.jpg" file="http://img.manew.com/data/attachment/forum/201811/12/121246qe07le3eqqaogov1.jpg.thumb.jpg" inpost="1" onmouseover="showMenu({'ctrlid':this.id,'pos':'12'})" lazyloaded="true" _load="1">

<div class="tip tip_4 aimg_tip" id="aimg_169559_menu" style="position: absolute; display: none" disautofocus="true">
<div class="xs0">
<p><strong>16.jpg</strong> <em class="xg1">(3.67 KB, 下载次数: 6)</em></p>
<p>
<a href="http://www.manew.com/forum.php?mod=attachment&amp;aid=MTY5NTU5fDNiMmJlNTAyfDE1NDQ0MDQxNjR8Mjk4NzA5fDE0MTg5MA%3D%3D&amp;nothumb=yes" target="_blank">下载附件</a>

&nbsp;<a href="javascript:;" onclick="showWindow(this.id, this.getAttribute('url'), 'get', 0);" id="savephoto_169559" url="home.php?mod=spacecp&amp;ac=album&amp;op=saveforumphoto&amp;aid=169559&amp;handlekey=savephoto_169559">保存到相册</a>

</p>

<p class="xg1 y">2018-11-12 12:12 上传</p>

</div>
<div class="tip_horn"></div>
</div>

</ignore_js_op>
</div><br>
<br>
<div align="left">这时如果我们想在ECS框架中控制这个小球（Player）的移动该怎么实现呢？</div><br>
<br>
<div align="left">我们之前在Bootstrap脚本中已经实现了Awake跟Start了，其实每一个System都会实现Update方法。并且会在Start调用后开始调用。</div><div align="left">player现在只包含两个组件：Position与MeshInstanceRenderer，显然缺乏一个标识组件，我们创建一个脚本并命名为PlayerComponent：</div><div style="padding:15px 0;"><div style="font-size:12px;">[AppleScript] <em class="viewsource" style="cursor:pointer;font-size:12px;color:#369 !important;" num="3">纯文本查看</em> <em class="copycode" style="cursor:pointer;font-size:12px;color:#369 !important;" num="3">复制代码</em></div><div><div id="highlighter_615800" class="syntaxhighlighter notranslate applescript"><div class="toolbar"><span><a href="#" class="toolbar_item command_help help">?</a></span></div><table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td class="gutter"><div class="line number0 index0 alt1 break">&nbsp;</div><div class="line number1 index1 alt2">1</div><div class="line number2 index2 alt1">2</div><div class="line number3 index3 alt2">3</div><div class="line number4 index4 alt1">4</div><div class="line number5 index5 alt2">5</div></td><td class="code"><div class="container"><div class="line number0 index0 alt1 break"><code class="applescript color3">using</code> <code class="applescript plain">Unity.Entities;</code></div><div class="line number1 index1 alt2">&nbsp;</div><div class="line number2 index2 alt1"><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">组件必须是struct并且得继承IComponentData接口</code></div><div class="line number3 index3 alt2"><code class="applescript plain">public struct PlayerComponent </code><code class="applescript color1">:</code> <code class="applescript plain">IComponentData</code></div><div class="line number4 index4 alt1"><code class="applescript color1">{</code></div><div class="line number5 index5 alt2"><code class="applescript color1">}</code></div></div></td></tr></tbody></table></div></div></div><br>
<div align="left">在Bootstrap.Start方法中，在player创建出来后加上一句：</div><div style="padding:15px 0;"><div style="font-size:12px;">[AppleScript] <em class="viewsource" style="cursor:pointer;font-size:12px;color:#369 !important;" num="4">纯文本查看</em> <em class="copycode" style="cursor:pointer;font-size:12px;color:#369 !important;" num="4">复制代码</em></div><div><div id="highlighter_872801" class="syntaxhighlighter notranslate applescript"><div class="toolbar"><span><a href="#" class="toolbar_item command_help help">?</a></span></div><table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td class="gutter"><div class="line number0 index0 alt1 break">&nbsp;</div></td><td class="code"><div class="container"><div class="line number0 index0 alt1 break"><code class="applescript plain">entityManager.AddComponentData</code><code class="applescript color1">(</code><code class="applescript plain">player</code><code class="applescript color1">,</code> <code class="applescript color3">new</code> <code class="applescript plain">PlayerComponent</code><code class="applescript color1">(</code><code class="applescript color1">)</code><code class="applescript color1">)</code><code class="applescript plain">; </code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">添加PlayerComponent组件</code></div></div></td></tr></tbody></table></div></div></div><br>
<br>
<br>
<br>
<div align="left">现在我们创建一个脚本命名为MovementSystem，继承自ComponentSystem：</div><blockquote>类似继承Monobehavior，我们的系统只要继承了这个基类就会被Unity识别，并且每一帧都调用OnUpdate。</blockquote><div style="padding:15px 0;"><div style="font-size:12px;">[AppleScript] <em class="viewsource" style="cursor:pointer;font-size:12px;color:#369 !important;" num="5">纯文本查看</em> <em class="copycode" style="cursor:pointer;font-size:12px;color:#369 !important;" num="5">复制代码</em></div><div><div id="highlighter_619702" class="syntaxhighlighter notranslate applescript"><div class="toolbar"><span><a href="#" class="toolbar_item command_help help">?</a></span></div><table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td class="gutter"><div class="line number0 index0 alt1 break">&nbsp;</div><div class="line number1 index1 alt2">01</div><div class="line number2 index2 alt1">02</div><div class="line number3 index3 alt2">03</div><div class="line number4 index4 alt1">04</div><div class="line number5 index5 alt2">05</div><div class="line number6 index6 alt1">06</div><div class="line number7 index7 alt2">07</div><div class="line number8 index8 alt1">08</div><div class="line number9 index9 alt2">09</div><div class="line number10 index10 alt1">10</div></td><td class="code"><div class="container"><div class="line number0 index0 alt1 break"><code class="applescript color3">using</code> <code class="applescript plain">UnityEngine;</code></div><div class="line number1 index1 alt2"><code class="applescript color3">using</code> <code class="applescript plain">Unity.Entities;</code></div><div class="line number2 index2 alt1"><code class="applescript color3">using</code> <code class="applescript plain">Unity.Transforms;</code></div><div class="line number3 index3 alt2"><code class="applescript color3">using</code> <code class="applescript plain">Unity.Mathematics;</code></div><div class="line number4 index4 alt1">&nbsp;</div><div class="line number5 index5 alt2"><code class="applescript plain">public </code><code class="applescript color3">class</code> <code class="applescript plain">MovementSystem </code><code class="applescript color1">:</code> <code class="applescript plain">ComponentSystem</code></div><div class="line number6 index6 alt1"><code class="applescript color1">{</code></div><div class="line number7 index7 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">protected override void OnUpdate</code><code class="applescript color1">(</code><code class="applescript color1">)</code></div><div class="line number8 index8 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">{</code></div><div class="line number9 index9 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">}</code></div><div class="line number10 index10 alt1"><code class="applescript color1">}</code></div></div></td></tr></tbody></table></div></div></div><br>
<div align="left">Unity ECS帮我们简化了获取实体再获取组件的过程，现在可以直接获取不同的组件。也就是说我们可以获取被EntityManager管理的实体上我们想要的组件组成的集合。</div><br>
<br>
<div align="left">利用一个特性：[Inject]：</div><br>
&nbsp; &nbsp; <div style="padding:15px 0;"><div style="font-size:12px;">[AppleScript] <em class="viewsource" style="cursor:pointer;font-size:12px;color:#369 !important;" num="6">纯文本查看</em> <em class="copycode" style="cursor:pointer;font-size:12px;color:#369 !important;" num="6">复制代码</em></div><div><div id="highlighter_579711" class="syntaxhighlighter notranslate applescript"><div class="toolbar"><span><a href="#" class="toolbar_item command_help help">?</a></span></div><table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td class="gutter"><div class="line number0 index0 alt1 break">&nbsp;</div><div class="line number1 index1 alt2">01</div><div class="line number2 index2 alt1">02</div><div class="line number3 index3 alt2">03</div><div class="line number4 index4 alt1">04</div><div class="line number5 index5 alt2">05</div><div class="line number6 index6 alt1">06</div><div class="line number7 index7 alt2">07</div><div class="line number8 index8 alt1">08</div><div class="line number9 index9 alt2">09</div><div class="line number10 index10 alt1">10</div><div class="line number11 index11 alt2">11</div><div class="line number12 index12 alt1">12</div><div class="line number13 index13 alt2">13</div><div class="line number14 index14 alt1">14</div><div class="line number15 index15 alt2">15</div><div class="line number16 index16 alt1">16</div><div class="line number17 index17 alt2">17</div><div class="line number18 index18 alt1">18</div><div class="line number19 index19 alt2">19</div><div class="line number20 index20 alt1">20</div><div class="line number21 index21 alt2">21</div><div class="line number22 index22 alt1">22</div><div class="line number23 index23 alt2">23</div><div class="line number24 index24 alt1">24</div><div class="line number25 index25 alt2">25</div></td><td class="code"><div class="container"><div class="line number0 index0 alt1 break"><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">这里声明一个结构</code><code class="applescript color1">,</code> <code class="applescript plain">其中包含我们定义的过滤条件</code><code class="applescript color1">,</code> <code class="applescript plain">也就是必须拥有CameraComponent组件才会被注入。</code></div><div class="line number1 index1 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">public struct Group</code></div><div class="line number2 index2 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">{</code></div><div class="line number3 index3 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">public readonly int Length;</code></div><div class="line number4 index4 alt1">&nbsp;</div><div class="line number5 index5 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">public ComponentDataArray</code><code class="applescript color2">&lt;</code><code class="applescript plain">Position</code><code class="applescript color2">&gt;</code> <code class="applescript plain">Positions;</code></div><div class="line number6 index6 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">}</code></div><div class="line number7 index7 alt2">&nbsp;</div><div class="line number8 index8 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">然后声明结构类型的字段</code><code class="applescript color1">,</code> <code class="applescript plain">并且加上[Inject]</code></div><div class="line number9 index9 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">[Inject] Group </code><code class="applescript color3">data</code><code class="applescript plain">;</code></div><div class="line number10 index10 alt1">&nbsp;</div><div class="line number11 index11 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">protected override void OnUpdate</code><code class="applescript color1">(</code><code class="applescript color1">)</code></div><div class="line number12 index12 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">{</code></div><div class="line number13 index13 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">float deltaTime </code><code class="applescript color2">=</code> <code class="applescript plain">Time.deltaTime;</code></div><div class="line number14 index14 alt1">&nbsp;</div><div class="line number15 index15 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color3">for</code> <code class="applescript color1">(</code><code class="applescript plain">int i </code><code class="applescript color2">=</code> <code class="applescript color1">0</code><code class="applescript plain">; i </code><code class="applescript color2">&lt;</code> <code class="applescript color3">data</code><code class="applescript plain">.Length; i</code><code class="applescript color2">+</code><code class="applescript color2">+</code><code class="applescript color1">)</code></div><div class="line number16 index16 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">{</code></div><div class="line number17 index17 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">float</code><code class="applescript color1">3</code> <code class="applescript color3">up</code> <code class="applescript color2">=</code> <code class="applescript color3">new</code> <code class="applescript plain">float</code><code class="applescript color1">3</code><code class="applescript color1">(</code><code class="applescript color1">0</code><code class="applescript color1">,</code> <code class="applescript color1">1</code><code class="applescript color1">,</code> <code class="applescript color1">0</code><code class="applescript color1">)</code><code class="applescript plain">;</code></div><div class="line number18 index18 alt1">&nbsp;</div><div class="line number19 index19 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">float</code><code class="applescript color1">3</code> <code class="applescript plain">pos </code><code class="applescript color2">=</code> <code class="applescript color3">data</code><code class="applescript plain">.Positions.Value; </code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">Read</code></div><div class="line number20 index20 alt1">&nbsp;</div><div class="line number21 index21 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">pos </code><code class="applescript color2">+</code><code class="applescript color2">=</code> <code class="applescript color3">up</code> <code class="applescript color2">*</code> <code class="applescript plain">deltaTime;</code></div><div class="line number22 index22 alt1">&nbsp;</div><div class="line number23 index23 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color3">data</code><code class="applescript plain">.Positions </code><code class="applescript color2">=</code> <code class="applescript color3">new</code> <code class="applescript plain">Position </code><code class="applescript color1">{</code> <code class="applescript plain">Value </code><code class="applescript color2">=</code> <code class="applescript plain">pos </code><code class="applescript color1">}</code><code class="applescript plain">; </code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">Write</code></div><div class="line number24 index24 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">}</code></div><div class="line number25 index25 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">}</code></div></div></td></tr></tbody></table></div></div></div></font></font></font>  

<font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
</font></font></font>  

<font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
<blockquote>声明了Length属性后，Length会被自动注入，它代表结构中每个数组的总元素数量，方便进行for循环迭代。</blockquote><div align="left">[Inject]会从所有Entity中寻找同时拥有PlayerComponent与Position组件的实体，接着获取他们的这些组件，注入我们声明的不同数组中。</div><div align="left">我们只需要在结构中声明好筛选的条件与我们需要的组件，ECS就会在背后帮我们处理，给我们想要的结果。</div><br>
<br>
<br>
<div align="left">运行后player果然升天了：</div><div align="center">
<ignore_js_op>

<img id="aimg_169560" aid="169560" src="http://img.manew.com/data/attachment/forum/201811/12/121437ewao3333wzfy3gzo.gif" zoomfile="http://img.manew.com/data/attachment/forum/201811/12/121437ewao3333wzfy3gzo.gif" file="http://img.manew.com/data/attachment/forum/201811/12/121437ewao3333wzfy3gzo.gif" class="zoom" onclick="zoom(this, this.src, 0, 0, 0)" width="350" inpost="1" onmouseover="showMenu({'ctrlid':this.id,'pos':'12'})" lazyloaded="true" _load="1" initialized="true" style="">

<div class="tip tip_4 aimg_tip" id="aimg_169560_menu" style="position: absolute; z-index: 301; left: 626.625px; top: 12466px; display: none;" disautofocus="true" initialized="true">
<div class="xs0">
<p><strong>1.2.gif</strong> <em class="xg1">(69.88 KB, 下载次数: 6)</em></p>
<p>
<a href="http://www.manew.com/forum.php?mod=attachment&amp;aid=MTY5NTYwfGEwNGJkMzU1fDE1NDQ0MDQxNjR8Mjk4NzA5fDE0MTg5MA%3D%3D&amp;nothumb=yes" target="_blank">下载附件</a>

&nbsp;<a href="javascript:;" onclick="showWindow(this.id, this.getAttribute('url'), 'get', 0);" id="savephoto_169560" url="home.php?mod=spacecp&amp;ac=album&amp;op=saveforumphoto&amp;aid=169560&amp;handlekey=savephoto_169560">保存到相册</a>

</p>

<p class="xg1 y">2018-11-12 12:14 上传</p>

</div>
<div class="tip_horn"></div>
</div>

</ignore_js_op>
<br>
</div><br>
<br>
<div align="left">趁热打铁，现在我们想自己通过输入控制小球在平面上移动。</div><div align="left">先声明一个组件InputComponent作为一个标识：</div><br>
<br>
<div style="padding:15px 0;"><div style="font-size:12px;">[AppleScript] <em class="viewsource" style="cursor:pointer;font-size:12px;color:#369 !important;" num="7">纯文本查看</em> <em class="copycode" style="cursor:pointer;font-size:12px;color:#369 !important;" num="7">复制代码</em></div><div><div id="highlighter_514716" class="syntaxhighlighter notranslate applescript"><div class="toolbar"><span><a href="#" class="toolbar_item command_help help">?</a></span></div><table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td class="gutter"><div class="line number0 index0 alt1 break">&nbsp;</div><div class="line number1 index1 alt2">1</div><div class="line number2 index2 alt1">2</div><div class="line number3 index3 alt2">3</div><div class="line number4 index4 alt1">4</div></td><td class="code"><div class="container"><div class="line number0 index0 alt1 break"><code class="applescript color3">using</code> <code class="applescript plain">Unity.Entities;</code></div><div class="line number1 index1 alt2">&nbsp;</div><div class="line number2 index2 alt1"><code class="applescript plain">public struct InputComponent </code><code class="applescript color1">:</code> <code class="applescript plain">IComponentData</code></div><div class="line number3 index3 alt2"><code class="applescript color1">{</code></div><div class="line number4 index4 alt1"><code class="applescript color1">}</code></div></div></td></tr></tbody></table></div></div></div></font></font></font>  

<font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
</font></font></font>  

<font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
<div align="left">然后再声明一个组件VelocityComponent保存我们的输入向量：</div><br>
<div style="padding:15px 0;"><div style="font-size:12px;">[AppleScript] <em class="viewsource" style="cursor:pointer;font-size:12px;color:#369 !important;" num="8">纯文本查看</em> <em class="copycode" style="cursor:pointer;font-size:12px;color:#369 !important;" num="8">复制代码</em></div><div><div id="highlighter_626178" class="syntaxhighlighter notranslate applescript"><div class="toolbar"><span><a href="#" class="toolbar_item command_help help">?</a></span></div><table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td class="gutter"><div class="line number0 index0 alt1 break">&nbsp;</div><div class="line number1 index1 alt2">1</div><div class="line number2 index2 alt1">2</div><div class="line number3 index3 alt2">3</div><div class="line number4 index4 alt1">4</div><div class="line number5 index5 alt2">5</div><div class="line number6 index6 alt1">6</div></td><td class="code"><div class="container"><div class="line number0 index0 alt1 break"><code class="applescript color3">using</code> <code class="applescript plain">Unity.Entities;</code></div><div class="line number1 index1 alt2"><code class="applescript color3">using</code> <code class="applescript plain">Unity.Mathematics;</code></div><div class="line number2 index2 alt1">&nbsp;</div><div class="line number3 index3 alt2"><code class="applescript plain">public struct VelocityComponent </code><code class="applescript color1">:</code> <code class="applescript plain">IComponentData</code></div><div class="line number4 index4 alt1"><code class="applescript color1">{</code></div><div class="line number5 index5 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">public float</code><code class="applescript color1">3</code> <code class="applescript plain">moveDir;&nbsp;&nbsp; </code></div><div class="line number6 index6 alt1"><code class="applescript color1">}</code></div></div></td></tr></tbody></table></div></div></div><br>
<blockquote>我们默认player的速度为1就不单独声明速度值了。</blockquote><div align="left">接着创建InputSystem来更改VelocityComponent的值，接下来的工作就是照猫画虎了：</div><div style="padding:15px 0;"><div style="font-size:12px;">[AppleScript] <em class="viewsource" style="cursor:pointer;font-size:12px;color:#369 !important;" num="9">纯文本查看</em> <em class="copycode" style="cursor:pointer;font-size:12px;color:#369 !important;" num="9">复制代码</em></div><div><div id="highlighter_374341" class="syntaxhighlighter notranslate applescript"><div class="toolbar"><span><a href="#" class="toolbar_item command_help help">?</a></span></div><table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td class="gutter"><div class="line number0 index0 alt1 break">&nbsp;</div><div class="line number1 index1 alt2">01</div><div class="line number2 index2 alt1">02</div><div class="line number3 index3 alt2">03</div><div class="line number4 index4 alt1">04</div><div class="line number5 index5 alt2">05</div><div class="line number6 index6 alt1">06</div><div class="line number7 index7 alt2">07</div><div class="line number8 index8 alt1">08</div><div class="line number9 index9 alt2">09</div><div class="line number10 index10 alt1">10</div><div class="line number11 index11 alt2">11</div><div class="line number12 index12 alt1">12</div><div class="line number13 index13 alt2">13</div><div class="line number14 index14 alt1">14</div><div class="line number15 index15 alt2">15</div><div class="line number16 index16 alt1">16</div><div class="line number17 index17 alt2">17</div><div class="line number18 index18 alt1">18</div><div class="line number19 index19 alt2">19</div><div class="line number20 index20 alt1">20</div><div class="line number21 index21 alt2">21</div><div class="line number22 index22 alt1">22</div><div class="line number23 index23 alt2">23</div><div class="line number24 index24 alt1">24</div><div class="line number25 index25 alt2">25</div><div class="line number26 index26 alt1">26</div><div class="line number27 index27 alt2">27</div><div class="line number28 index28 alt1">28</div><div class="line number29 index29 alt2">29</div><div class="line number30 index30 alt1">30</div><div class="line number31 index31 alt2">31</div><div class="line number32 index32 alt1">32</div></td><td class="code"><div class="container"><div class="line number0 index0 alt1 break"><code class="applescript color3">using</code> <code class="applescript plain">UnityEngine;</code></div><div class="line number1 index1 alt2"><code class="applescript color3">using</code> <code class="applescript plain">Unity.Entities;</code></div><div class="line number2 index2 alt1"><code class="applescript color3">using</code> <code class="applescript plain">Unity.Mathematics;</code></div><div class="line number3 index3 alt2">&nbsp;</div><div class="line number4 index4 alt1"><code class="applescript plain">public </code><code class="applescript color3">class</code> <code class="applescript plain">InputSystem </code><code class="applescript color1">:</code> <code class="applescript plain">ComponentSystem</code></div><div class="line number5 index5 alt2"><code class="applescript color1">{</code></div><div class="line number6 index6 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">public struct Group</code></div><div class="line number7 index7 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">{</code></div><div class="line number8 index8 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">public readonly int Length;</code></div><div class="line number9 index9 alt2">&nbsp;</div><div class="line number10 index10 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">public ComponentDataArray</code><code class="applescript color2">&lt;</code><code class="applescript plain">PlayerComponent</code><code class="applescript color2">&gt;</code> <code class="applescript plain">Players;</code></div><div class="line number11 index11 alt2">&nbsp;</div><div class="line number12 index12 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">public ComponentDataArray</code><code class="applescript color2">&lt;</code><code class="applescript plain">InputComponent</code><code class="applescript color2">&gt;</code> <code class="applescript plain">Inputs;</code></div><div class="line number13 index13 alt2">&nbsp;</div><div class="line number14 index14 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">public ComponentDataArray</code><code class="applescript color2">&lt;</code><code class="applescript plain">VelocityComponent</code><code class="applescript color2">&gt;</code> <code class="applescript plain">Velocities;</code></div><div class="line number15 index15 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">}</code></div><div class="line number16 index16 alt1">&nbsp;</div><div class="line number17 index17 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">[Inject] Group </code><code class="applescript color3">data</code><code class="applescript plain">;</code></div><div class="line number18 index18 alt1">&nbsp;</div><div class="line number19 index19 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">protected override void OnUpdate</code><code class="applescript color1">(</code><code class="applescript color1">)</code></div><div class="line number20 index20 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">{</code></div><div class="line number21 index21 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color3">for</code> <code class="applescript color1">(</code><code class="applescript plain">int i </code><code class="applescript color2">=</code> <code class="applescript color1">0</code><code class="applescript plain">; i </code><code class="applescript color2">&lt;</code> <code class="applescript color3">data</code><code class="applescript plain">.Length; i</code><code class="applescript color2">+</code><code class="applescript color2">+</code><code class="applescript color1">)</code></div><div class="line number22 index22 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">{</code></div><div class="line number23 index23 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">float x </code><code class="applescript color2">=</code> <code class="applescript plain">Input.GetAxisRaw</code><code class="applescript color1">(</code><code class="applescript string">"Horizontal"</code><code class="applescript color1">)</code><code class="applescript plain">;</code></div><div class="line number24 index24 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">float z </code><code class="applescript color2">=</code> <code class="applescript plain">Input.GetAxisRaw</code><code class="applescript color1">(</code><code class="applescript string">"Vertical"</code><code class="applescript color1">)</code><code class="applescript plain">;</code></div><div class="line number25 index25 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">float</code><code class="applescript color1">3</code> <code class="applescript plain">normalized </code><code class="applescript color2">=</code> <code class="applescript color3">new</code> <code class="applescript plain">float</code><code class="applescript color1">3</code><code class="applescript color1">(</code><code class="applescript color1">)</code><code class="applescript plain">;</code></div><div class="line number26 index26 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript keyword">if</code><code class="applescript color1">(</code><code class="applescript plain">x !</code><code class="applescript color2">=</code> <code class="applescript color1">0</code> <code class="applescript plain">|| y !</code><code class="applescript color2">=</code> <code class="applescript color1">0</code><code class="applescript color1">)</code></div><div class="line number27 index27 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">normalized </code><code class="applescript color2">=</code> <code class="applescript plain">math.normalize</code><code class="applescript color1">(</code><code class="applescript color3">new</code> <code class="applescript plain">float</code><code class="applescript color1">3</code><code class="applescript color1">(</code><code class="applescript plain">x</code><code class="applescript color1">,</code> <code class="applescript color1">0</code><code class="applescript color1">,</code> <code class="applescript plain">z</code><code class="applescript color1">)</code><code class="applescript color1">)</code><code class="applescript plain">;</code></div><div class="line number28 index28 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>&nbsp;</div><div class="line number29 index29 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color3">data</code><code class="applescript plain">.Velocities </code><code class="applescript color2">=</code> <code class="applescript color3">new</code> <code class="applescript plain">VelocityComponent </code><code class="applescript color1">{</code> <code class="applescript plain">moveDir </code><code class="applescript color2">=</code> <code class="applescript plain">normalized </code><code class="applescript color1">}</code><code class="applescript plain">; </code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">Write</code></div><div class="line number30 index30 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">}</code></div><div class="line number31 index31 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">}</code></div><div class="line number32 index32 alt1"><code class="applescript color1">}</code></div></div></td></tr></tbody></table></div></div></div><br>
<blockquote>比较麻烦的一点就是，在游戏初期没有确立基础的组件与系统时需要频繁修改。</blockquote><div align="left">移动系统也需要修改：</div><br>
<br>
&nbsp;&nbsp;<div style="padding:15px 0;"><div style="font-size:12px;">[AppleScript] <em class="viewsource" style="cursor:pointer;font-size:12px;color:#369 !important;" num="10">纯文本查看</em> <em class="copycode" style="cursor:pointer;font-size:12px;color:#369 !important;" num="10">复制代码</em></div><div><div id="highlighter_452132" class="syntaxhighlighter notranslate applescript"><div class="toolbar"><span><a href="#" class="toolbar_item command_help help">?</a></span></div><table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td class="gutter"><div class="line number0 index0 alt1 break">&nbsp;</div><div class="line number1 index1 alt2">01</div><div class="line number2 index2 alt1">02</div><div class="line number3 index3 alt2">03</div><div class="line number4 index4 alt1">04</div><div class="line number5 index5 alt2">05</div><div class="line number6 index6 alt1">06</div><div class="line number7 index7 alt2">07</div><div class="line number8 index8 alt1">08</div><div class="line number9 index9 alt2">09</div><div class="line number10 index10 alt1">10</div><div class="line number11 index11 alt2">11</div><div class="line number12 index12 alt1">12</div><div class="line number13 index13 alt2">13</div><div class="line number14 index14 alt1">14</div><div class="line number15 index15 alt2">15</div><div class="line number16 index16 alt1">16</div><div class="line number17 index17 alt2">17</div><div class="line number18 index18 alt1">18</div><div class="line number19 index19 alt2">19</div><div class="line number20 index20 alt1">20</div><div class="line number21 index21 alt2">21</div><div class="line number22 index22 alt1">22</div><div class="line number23 index23 alt2">23</div><div class="line number24 index24 alt1">24</div><div class="line number25 index25 alt2">25</div><div class="line number26 index26 alt1">26</div></td><td class="code"><div class="container"><div class="line number0 index0 alt1 break"><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">这里声明一个结构</code><code class="applescript color1">,</code> <code class="applescript plain">其中包含我们定义的过滤条件</code><code class="applescript color1">,</code> <code class="applescript plain">也就是必须拥有CameraComponent组件才会被注入。</code></div><div class="line number1 index1 alt2"><code class="applescript spaces">&nbsp;&nbsp;</code><code class="applescript plain">public struct Group</code></div><div class="line number2 index2 alt1"><code class="applescript spaces">&nbsp;&nbsp;</code><code class="applescript color1">{</code></div><div class="line number3 index3 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">public readonly int Length;</code></div><div class="line number4 index4 alt1">&nbsp;</div><div class="line number5 index5 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">public ComponentDataArray</code><code class="applescript color2">&lt;</code><code class="applescript plain">VelocityComponent</code><code class="applescript color2">&gt;</code> <code class="applescript plain">Velocities;</code></div><div class="line number6 index6 alt1">&nbsp;</div><div class="line number7 index7 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">public ComponentDataArray</code><code class="applescript color2">&lt;</code><code class="applescript plain">Position</code><code class="applescript color2">&gt;</code> <code class="applescript plain">Positions;</code></div><div class="line number8 index8 alt1"><code class="applescript spaces">&nbsp;&nbsp;</code><code class="applescript color1">}</code></div><div class="line number9 index9 alt2">&nbsp;</div><div class="line number10 index10 alt1"><code class="applescript spaces">&nbsp;&nbsp;</code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">然后声明结构类型的字段</code><code class="applescript color1">,</code> <code class="applescript plain">并且加上[Inject]</code></div><div class="line number11 index11 alt2"><code class="applescript spaces">&nbsp;&nbsp;</code><code class="applescript plain">[Inject] Group </code><code class="applescript color3">data</code><code class="applescript plain">;</code></div><div class="line number12 index12 alt1">&nbsp;</div><div class="line number13 index13 alt2"><code class="applescript spaces">&nbsp;&nbsp;</code><code class="applescript plain">protected override void OnUpdate</code><code class="applescript color1">(</code><code class="applescript color1">)</code></div><div class="line number14 index14 alt1"><code class="applescript spaces">&nbsp;&nbsp;</code><code class="applescript color1">{</code></div><div class="line number15 index15 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">float deltaTime </code><code class="applescript color2">=</code> <code class="applescript plain">Time.deltaTime;</code></div><div class="line number16 index16 alt1">&nbsp;</div><div class="line number17 index17 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color3">for</code> <code class="applescript color1">(</code><code class="applescript plain">int i </code><code class="applescript color2">=</code> <code class="applescript color1">0</code><code class="applescript plain">; i </code><code class="applescript color2">&lt;</code> <code class="applescript color3">data</code><code class="applescript plain">.Length; i</code><code class="applescript color2">+</code><code class="applescript color2">+</code><code class="applescript color1">)</code></div><div class="line number18 index18 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">{</code></div><div class="line number19 index19 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">float</code><code class="applescript color1">3</code> <code class="applescript plain">pos </code><code class="applescript color2">=</code> <code class="applescript color3">data</code><code class="applescript plain">.Positions.Value;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">Read</code></div><div class="line number20 index20 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">float</code><code class="applescript color1">3</code> <code class="applescript plain">vector </code><code class="applescript color2">=</code> <code class="applescript color3">data</code><code class="applescript plain">.Velocities.moveDir;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">Read</code></div><div class="line number21 index21 alt2">&nbsp;</div><div class="line number22 index22 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">pos </code><code class="applescript color2">+</code><code class="applescript color2">=</code> <code class="applescript plain">vector </code><code class="applescript color2">*</code> <code class="applescript plain">deltaTime; </code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">Move</code></div><div class="line number23 index23 alt2">&nbsp;</div><div class="line number24 index24 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color3">data</code><code class="applescript plain">.Positions </code><code class="applescript color2">=</code> <code class="applescript color3">new</code> <code class="applescript plain">Position </code><code class="applescript color1">{</code> <code class="applescript plain">Value </code><code class="applescript color2">=</code> <code class="applescript plain">pos </code><code class="applescript color1">}</code><code class="applescript plain">; </code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">Write</code></div><div class="line number25 index25 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">}</code></div><div class="line number26 index26 alt1"><code class="applescript spaces">&nbsp;&nbsp;</code><code class="applescript color1">}</code></div></div></td></tr></tbody></table></div></div></div></font></font></font>  

<font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
<div align="left">还要回到Bootstrap.Start中，向我们的player继续添加这两个组件：</div><br>
&nbsp; &nbsp;&nbsp; &nbsp; <div style="padding:15px 0;"><div style="font-size:12px;">[AppleScript] <em class="viewsource" style="cursor:pointer;font-size:12px;color:#369 !important;" num="11">纯文本查看</em> <em class="copycode" style="cursor:pointer;font-size:12px;color:#369 !important;" num="11">复制代码</em></div><div><div id="highlighter_777603" class="syntaxhighlighter notranslate applescript"><div class="toolbar"><span><a href="#" class="toolbar_item command_help help">?</a></span></div><table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td class="gutter"><div class="line number0 index0 alt1 break">&nbsp;</div><div class="line number1 index1 alt2">01</div><div class="line number2 index2 alt1">02</div><div class="line number3 index3 alt2">03</div><div class="line number4 index4 alt1">04</div><div class="line number5 index5 alt2">05</div><div class="line number6 index6 alt1">06</div><div class="line number7 index7 alt2">07</div><div class="line number8 index8 alt1">08</div><div class="line number9 index9 alt2">09</div><div class="line number10 index10 alt1">10</div></td><td class="code"><div class="container"><div class="line number0 index0 alt1 break"><code class="applescript plain">Entity player </code><code class="applescript color2">=</code> <code class="applescript plain">entityManager.CreateEntity</code><code class="applescript color1">(</code><code class="applescript plain">playerArchetype</code><code class="applescript color1">)</code><code class="applescript plain">;&nbsp;&nbsp;&nbsp; </code><code class="applescript color2">/</code><code class="applescript color2">/</code> <code class="applescript plain">Position</code></div><div class="line number1 index1 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">添加PlayerComponent组件</code></div><div class="line number2 index2 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">entityManager.AddComponentData</code><code class="applescript color1">(</code><code class="applescript plain">player</code><code class="applescript color1">,</code> <code class="applescript color3">new</code> <code class="applescript plain">PlayerComponent</code><code class="applescript color1">(</code><code class="applescript color1">)</code><code class="applescript color1">)</code><code class="applescript plain">;&nbsp; </code><code class="applescript color2">/</code><code class="applescript color2">/</code> <code class="applescript plain">PlayerComponent</code></div><div class="line number3 index3 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">entityManager.AddComponentData</code><code class="applescript color1">(</code><code class="applescript plain">player</code><code class="applescript color1">,</code> <code class="applescript color3">new</code> <code class="applescript plain">VelocityComponent</code><code class="applescript color1">(</code><code class="applescript color1">)</code><code class="applescript color1">)</code><code class="applescript plain">;</code><code class="applescript color2">/</code><code class="applescript color2">/</code> <code class="applescript plain">VelocityComponent</code></div><div class="line number4 index4 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">entityManager.AddComponentData</code><code class="applescript color1">(</code><code class="applescript plain">player</code><code class="applescript color1">,</code> <code class="applescript color3">new</code> <code class="applescript plain">InputComponent</code><code class="applescript color1">(</code><code class="applescript color1">)</code><code class="applescript color1">)</code><code class="applescript plain">;&nbsp;&nbsp; </code><code class="applescript color2">/</code><code class="applescript color2">/</code> <code class="applescript plain">InputComponent</code></div><div class="line number5 index5 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color2">/</code><code class="applescript color2">/</code> <code class="applescript plain">向实体添加共享的数据</code></div><div class="line number6 index6 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">entityManager.AddSharedComponentData</code><code class="applescript color1">(</code><code class="applescript plain">player</code><code class="applescript color1">,</code> <code class="applescript plain">playerRenderer</code><code class="applescript color1">)</code><code class="applescript plain">;&nbsp;&nbsp; </code><code class="applescript color2">/</code><code class="applescript color2">/</code> <code class="applescript plain">MeshInstanceRenderer</code></div><div class="line number7 index7 alt2">&nbsp;</div><div class="line number8 index8 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">修改实体的Position组件</code></div><div class="line number9 index9 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">entityManager.SetComponentData</code><code class="applescript color1">(</code><code class="applescript plain">player</code><code class="applescript color1">,</code> <code class="applescript color3">new</code> <code class="applescript plain">Position </code></div><div class="line number10 index10 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">{</code> <code class="applescript plain">Value </code><code class="applescript color2">=</code> <code class="applescript color3">new</code> <code class="applescript plain">Unity.Mathematics.float</code><code class="applescript color1">3</code><code class="applescript color1">(</code><code class="applescript color1">0</code><code class="applescript color1">,</code> <code class="applescript color1">0.5</code><code class="applescript plain">f</code><code class="applescript color1">,</code> <code class="applescript color1">0</code><code class="applescript color1">)</code> <code class="applescript color1">}</code><code class="applescript color1">)</code><code class="applescript plain">;</code></div></div></td></tr></tbody></table></div></div></div><br>
<div align="left">Duang：</div><div align="center">
<ignore_js_op>

<img id="aimg_169561" aid="169561" src="http://img.manew.com/data/attachment/forum/201811/12/121609ox4ppxun4tpntu6n.gif" zoomfile="http://img.manew.com/data/attachment/forum/201811/12/121609ox4ppxun4tpntu6n.gif" file="http://img.manew.com/data/attachment/forum/201811/12/121609ox4ppxun4tpntu6n.gif" class="zoom" onclick="zoom(this, this.src, 0, 0, 0)" width="363" inpost="1" onmouseover="showMenu({'ctrlid':this.id,'pos':'12'})" lazyloaded="true" _load="1" initialized="true" style="">

<div class="tip tip_4 aimg_tip" id="aimg_169561_menu" style="position: absolute; z-index: 301; left: 620.125px; top: 15603px; display: none;" disautofocus="true" initialized="true">
<div class="xs0">
<p><strong>1.3.gif</strong> <em class="xg1">(93.59 KB, 下载次数: 7)</em></p>
<p>
<a href="http://www.manew.com/forum.php?mod=attachment&amp;aid=MTY5NTYxfDQwOTQ0NDAyfDE1NDQ0MDQxNjR8Mjk4NzA5fDE0MTg5MA%3D%3D&amp;nothumb=yes" target="_blank">下载附件</a>

&nbsp;<a href="javascript:;" onclick="showWindow(this.id, this.getAttribute('url'), 'get', 0);" id="savephoto_169561" url="home.php?mod=spacecp&amp;ac=album&amp;op=saveforumphoto&amp;aid=169561&amp;handlekey=savephoto_169561">保存到相册</a>

</p>

<p class="xg1 y">2018-11-12 12:16 上传</p>

</div>
<div class="tip_horn"></div>
</div>

</ignore_js_op>
<br>
</div><div align="center">可以WASD操控player移动了</div><br>
<br>
<div align="left">Unity ECS只提供了渲染系统并没有提供物理系统，如果要跟以前的项目结合，我们还需要能够访问场景中的游戏物体，比如一个经典的Cube。</div><br>
<div align="center">
<ignore_js_op>

<div style="width: 100px; height: 100px;background: url(static/image/common/loading.gif) no-repeat center center;"></div><img style="cursor: pointer; height: 1px; width: 1px;" id="aimg_169562" aid="169562" src="http://img.manew.com/data/attachment/forum/201811/12/121729p7133x841a8661ib.jpg.thumb.jpg" onclick="zoom(this, this.getAttribute('zoomfile'), 0, 0, '0')" zoomfile="http://img.manew.com/data/attachment/forum/201811/12/121729p7133x841a8661ib.jpg" file="http://img.manew.com/data/attachment/forum/201811/12/121729p7133x841a8661ib.jpg.thumb.jpg" inpost="1" onmouseover="showMenu({'ctrlid':this.id,'pos':'12'})" lazyloaded="true">

<div class="tip tip_4 aimg_tip" id="aimg_169562_menu" style="position: absolute; display: none" disautofocus="true">
<div class="xs0">
<p><strong>17.jpg</strong> <em class="xg1">(3.64 KB, 下载次数: 7)</em></p>
<p>
<a href="http://www.manew.com/forum.php?mod=attachment&amp;aid=MTY5NTYyfDU5MmNmMTllfDE1NDQ0MDQxNjR8Mjk4NzA5fDE0MTg5MA%3D%3D&amp;nothumb=yes" target="_blank">下载附件</a>

&nbsp;<a href="javascript:;" onclick="showWindow(this.id, this.getAttribute('url'), 'get', 0);" id="savephoto_169562" url="home.php?mod=spacecp&amp;ac=album&amp;op=saveforumphoto&amp;aid=169562&amp;handlekey=savephoto_169562">保存到相册</a>

</p>

<p class="xg1 y">2018-11-12 12:17 上传</p>

</div>
<div class="tip_horn"></div>
</div>

</ignore_js_op>
</div><br>
我们的目的是让这个立方体升天</font></font></font>  

<font face="微软雅黑"><font size="3"><font color="#006000"><br>
</font></font></font>  

<font face="微软雅黑"><font size="3"><font color="#006000"><br>
</font></font></font>

<font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><div align="left">在Bootstrap.Start中获取我们的Cube，并且加上GameObjectEntity组件。</div><blockquote>GameObjectEntity 确实叫这个名, 是Unity提供的组件。</blockquote><div align="left">添加上这个组件后Cube就可以被entityManager关注，并且可以获取Cube上的任意组件：</div><br>
&nbsp; &nbsp;&nbsp; &nbsp; <div style="padding:15px 0;"><div style="font-size:12px;">[AppleScript] <em class="viewsource" style="cursor:pointer;font-size:12px;color:#369 !important;" num="12">纯文本查看</em> <em class="copycode" style="cursor:pointer;font-size:12px;color:#369 !important;" num="12">复制代码</em></div><div><div id="highlighter_860064" class="syntaxhighlighter notranslate applescript"><div class="toolbar"><span><a href="#" class="toolbar_item command_help help">?</a></span></div><table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td class="gutter"><div class="line number0 index0 alt1 break">&nbsp;</div><div class="line number1 index1 alt2">1</div><div class="line number2 index2 alt1">2</div><div class="line number3 index3 alt2">3</div><div class="line number4 index4 alt1">4</div><div class="line number5 index5 alt2">5</div></td><td class="code"><div class="container"><div class="line number0 index0 alt1 break"><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">获取Cube&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </code></div><div class="line number1 index1 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">GameObjectEntity cubeEntity </code><code class="applescript color2">=</code> <code class="applescript plain">GameObject.Find</code><code class="applescript color1">(</code><code class="applescript string">"Cube"</code><code class="applescript color1">)</code><code class="applescript plain">.AddComponent</code><code class="applescript color2">&lt;</code><code class="applescript plain">GameObjectEntity</code><code class="applescript color2">&gt;</code><code class="applescript color1">(</code><code class="applescript color1">)</code><code class="applescript plain">;</code></div><div class="line number2 index2 alt1">&nbsp;</div><div class="line number3 index3 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">添加Velocity组件</code></div><div class="line number4 index4 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">entityManager.AddComponentData</code><code class="applescript color1">(</code><code class="applescript plain">cubeEntity.Entity</code><code class="applescript color1">,</code> <code class="applescript color3">new</code> <code class="applescript plain">VelocityComponent</code></div><div class="line number5 index5 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">{</code> <code class="applescript plain">moveDir </code><code class="applescript color2">=</code> <code class="applescript color3">new</code> <code class="applescript plain">Unity.Mathematics.float</code><code class="applescript color1">3</code><code class="applescript color1">(</code><code class="applescript color1">0</code><code class="applescript color1">,</code> <code class="applescript color1">1</code><code class="applescript color1">,</code> <code class="applescript color1">0</code><code class="applescript color1">)</code> <code class="applescript color1">}</code><code class="applescript color1">)</code><code class="applescript plain">;</code></div></div></td></tr></tbody></table></div></div></div></font></font></font>  

<font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
</font></font></font>  

<font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
<div align="left">我们向Cube添加了一个VelocityComponent组件，在MovementSystem加上这些代码：</div><br>
<br>
&nbsp; &nbsp;<div style="padding:15px 0;"><div style="font-size:12px;">[AppleScript] <em class="viewsource" style="cursor:pointer;font-size:12px;color:#369 !important;" num="13">纯文本查看</em> <em class="copycode" style="cursor:pointer;font-size:12px;color:#369 !important;" num="13">复制代码</em></div><div><div id="highlighter_307500" class="syntaxhighlighter notranslate applescript"><div class="toolbar"><span><a href="#" class="toolbar_item command_help help">?</a></span></div><table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td class="gutter"><div class="line number0 index0 alt1 break">&nbsp;</div><div class="line number1 index1 alt2">1</div><div class="line number2 index2 alt1">2</div><div class="line number3 index3 alt2">3</div><div class="line number4 index4 alt1">4</div><div class="line number5 index5 alt2">5</div><div class="line number6 index6 alt1">6</div><div class="line number7 index7 alt2">7</div><div class="line number8 index8 alt1">8</div><div class="line number9 index9 alt2">9</div></td><td class="code"><div class="container"><div class="line number0 index0 alt1 break"><code class="applescript plain">public struct GameObject</code></div><div class="line number1 index1 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">{</code></div><div class="line number2 index2 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">public readonly int Length;</code></div><div class="line number3 index3 alt2">&nbsp;</div><div class="line number4 index4 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">public ComponentArray</code><code class="applescript color2">&lt;</code><code class="applescript plain">Transform</code><code class="applescript color2">&gt;</code> <code class="applescript plain">Transforms; </code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">该数组可以获取传统的Component</code></div><div class="line number5 index5 alt2">&nbsp;</div><div class="line number6 index6 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">public ComponentDataArray</code><code class="applescript color2">&lt;</code><code class="applescript plain">VelocityComponent</code><code class="applescript color2">&gt;</code> <code class="applescript plain">Velocities;</code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">该数组获取继承IComponentData的</code></div><div class="line number7 index7 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">}</code></div><div class="line number8 index8 alt1">&nbsp;</div><div class="line number9 index9 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">[Inject] GameObject go;</code></div></div></td></tr></tbody></table></div></div></div><br>
<div align="left">在OnUpdate中加上这些代码，针对Transform进行操作：</div>&nbsp; &nbsp;&nbsp; &nbsp;<div style="padding:15px 0;"><div style="font-size:12px;">[AppleScript] <em class="viewsource" style="cursor:pointer;font-size:12px;color:#369 !important;" num="14">纯文本查看</em> <em class="copycode" style="cursor:pointer;font-size:12px;color:#369 !important;" num="14">复制代码</em></div><div><div id="highlighter_471210" class="syntaxhighlighter notranslate applescript"><div class="toolbar"><span><a href="#" class="toolbar_item command_help help">?</a></span></div><table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td class="gutter"><div class="line number0 index0 alt1 break">&nbsp;</div><div class="line number1 index1 alt2">1</div><div class="line number2 index2 alt1">2</div><div class="line number3 index3 alt2">3</div><div class="line number4 index4 alt1">4</div><div class="line number5 index5 alt2">5</div><div class="line number6 index6 alt1">6</div><div class="line number7 index7 alt2">7</div><div class="line number8 index8 alt1">8</div></td><td class="code"><div class="container"><div class="line number0 index0 alt1 break"><code class="applescript color3">for</code> <code class="applescript color1">(</code><code class="applescript plain">int i </code><code class="applescript color2">=</code> <code class="applescript color1">0</code><code class="applescript plain">; i </code><code class="applescript color2">&lt;</code> <code class="applescript plain">go.Length; i</code><code class="applescript color2">+</code><code class="applescript color2">+</code><code class="applescript color1">)</code></div><div class="line number1 index1 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">{</code></div><div class="line number2 index2 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">float</code><code class="applescript color1">3</code> <code class="applescript plain">pos </code><code class="applescript color2">=</code> <code class="applescript plain">go.Transforms.</code><code class="applescript color3">position</code><code class="applescript plain">; </code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">Read</code></div><div class="line number3 index3 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">float</code><code class="applescript color1">3</code> <code class="applescript plain">vector </code><code class="applescript color2">=</code> <code class="applescript plain">go.Velocities.moveDir; </code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">Read</code></div><div class="line number4 index4 alt1">&nbsp;</div><div class="line number5 index5 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">pos </code><code class="applescript color2">+</code><code class="applescript color2">=</code> <code class="applescript plain">vector </code><code class="applescript color2">*</code> <code class="applescript plain">deltaTime; </code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">Move</code></div><div class="line number6 index6 alt1">&nbsp;</div><div class="line number7 index7 alt2"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript plain">go.Transforms.</code><code class="applescript color3">position</code> <code class="applescript color2">=</code> <code class="applescript plain">pos; </code><code class="applescript color2">/</code><code class="applescript color2">/</code><code class="applescript plain">Write</code></div><div class="line number8 index8 alt1"><code class="applescript spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="applescript color1">}</code></div></div></td></tr></tbody></table></div></div></div></font></font></font>  

<font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
<div align="left">运行游戏后，我们可以看到：</div><div align="center">
<ignore_js_op>

<img id="aimg_169563" aid="169563" src="http://img.manew.com/data/attachment/forum/201811/12/121828t7bulfb7luawzp8w.gif" zoomfile="http://img.manew.com/data/attachment/forum/201811/12/121828t7bulfb7luawzp8w.gif" file="http://img.manew.com/data/attachment/forum/201811/12/121828t7bulfb7luawzp8w.gif" class="zoom" onclick="zoom(this, this.src, 0, 0, 0)" width="363" inpost="1" onmouseover="showMenu({'ctrlid':this.id,'pos':'12'})" lazyloaded="true" _load="1" style="">

<div class="tip tip_4 aimg_tip" id="aimg_169563_menu" style="position: absolute; display: none" disautofocus="true">
<div class="xs0">
<p><strong>1.4.gif</strong> <em class="xg1">(108.21 KB, 下载次数: 7)</em></p>
<p>
<a href="http://www.manew.com/forum.php?mod=attachment&amp;aid=MTY5NTYzfDNkOTE3OTBhfDE1NDQ0MDQxNjR8Mjk4NzA5fDE0MTg5MA%3D%3D&amp;nothumb=yes" target="_blank">下载附件</a>

&nbsp;<a href="javascript:;" onclick="showWindow(this.id, this.getAttribute('url'), 'get', 0);" id="savephoto_169563" url="home.php?mod=spacecp&amp;ac=album&amp;op=saveforumphoto&amp;aid=169563&amp;handlekey=savephoto_169563">保存到相册</a>

</p>

<p class="xg1 y">2018-11-12 12:18 上传</p>

</div>
<div class="tip_horn"></div>
</div>

</ignore_js_op>
<br>
</div><div align="center"><font color="#1a1a1a">Cube果然上天了</font></div><br>
<div align="center"><font color="#1a1a1a"><br>
</font></div><br>
<div align="center"><font color="#1a1a1a"><br>
</font></div><br>
<div align="left">Cube跟player的移动其实是被不同的系统实现的，player是因为被默认存在的渲染系统关注了所以实现了移动，而Cube是我们自己的MovementSystem实现的。</div><br>
<br>
<div align="left">如果想在ECS中用到之前的物理系统最好是自己写一个单独的系统并关注Rigidbody，BoxCollider这些传统组件，然后在OnUpdate中使用它们。</div><font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
</font></font></font><br>
<font face="微软雅黑"><font size="3"><font style="color:rgb(26, 26, 26)"><br>
</font></font></font><br>
<br>
<div align="left">看到这里你应该已经明白了ECS特点与Unity ECS的用法了，希望可以勾起你们对于ECS的兴趣，在以后针对多核开发的时代，相信ECS会成为高性能的代表。</div><div align="left">介于篇幅原因，JobComponentSystem，NativeArray，System并行，组件的先后顺序，读写权限这些跟性能优化相关的点就没有介绍了，感兴趣的话可以去Unity ECS官网了解。</div><br>
<div align="left"><a href="http://link.zhihu.com/?target=https%3A//github.com/Unity-Technologies/EntityComponentSystemSamples" target="_blank">https://github.com/Unity-Technologies/EntityComponentSystemSamples​github.com</a></div><br>
<br>
<div align="left">附上项目下载地址：</div><br>
<div align="left"><a href="http://link.zhihu.com/?target=https%3A//pan.baidu.com/s/1JI92OmEy0wEDz1IDhitIGw" target="_blank">https://pan.baidu.com/s/1JI92OmEy0wEDz1IDhitIGw​pan.baidu.com</a></div><br>
</font></font></font>_<font face="微软雅黑"><font size="3">密码：altt<br>
<br>
</font></font>

<font face="微软雅黑"><font size="3">等以后Unity ECS更完善时再出一期。这期文章就到这里了，拜拜咯。</font></font>_
