# 窗体透明

## 第一种方案：结合了系统API和shader

偶然在国外网站上看到的一个脚本，通过纯色背景抠像的方法，把windows的窗体透明化，可以用来做无背景的小头像、桌面精灵等。

首先新建一个叫TransparentWindow的CS脚本，输入如下代码：

```C#
using System;
using System.Runtime.InteropServices;
using UnityEngine;

public class TransparentWindow : MonoBehaviour
{
    [SerializeField]
    private Material m_Material;

    private struct MARGINS
    {
        public int cxLeftWidth;
        public int cxRightWidth;
        public int cyTopHeight;
        public int cyBottomHeight;
    }

    // Define function signatures to import from Windows APIs

    [DllImport("user32.dll")]
    private static extern IntPtr GetActiveWindow();

    [DllImport("user32.dll")]
    private static extern int SetWindowLong(IntPtr hWnd, int nIndex, uint dwNewLong);

    [DllImport("Dwmapi.dll")]
    private static extern uint DwmExtendFrameIntoClientArea(IntPtr hWnd, ref MARGINS margins);

    // Definitions of window styles
    const int GWL_STYLE = -16;
    const uint WS_POPUP = 0x80000000;
    const uint WS_VISIBLE = 0x10000000;

    void Start()
    {
#if !UNITY_EDITOR
        var margins = new MARGINS() { cxLeftWidth = -1 };

        // Get a handle to the window
        var hwnd = GetActiveWindow();

        // Set properties of the window
        // See: https://msdn.microsoft.com/en-us/library/windows/desktop/ms633591%28v=vs.85%29.aspx
        SetWindowLong(hwnd, GWL_STYLE, WS_POPUP | WS_VISIBLE);

        // Extend the window into the client area
        //See: https://msdn.microsoft.com/en-us/library/windows/desktop/aa969512%28v=vs.85%29.aspx 
        DwmExtendFrameIntoClientArea(hwnd, ref margins);
#endif
    }

    // Pass the output of the camera to the custom material
    // for chroma replacement
    void OnRenderImage(RenderTexture from, RenderTexture to)
    {
        Graphics.Blit(from, to, m_Material);
    }
}
```

然后新建一个叫ChromakeyTransparent的Shader：
```js
Shader "Custom/ChromakeyTransparent" {
    Properties{
        _MainTex("Base (RGB)", 2D) = "white" {}
        _TransparentColourKey("Transparent Colour Key", Color) = (0,0,0,1)
        _TransparencyTolerance("Transparency Tolerance", Float) = 0.01
    }

    SubShader{
        Pass{
            Tags{ "RenderType" = "Opaque" }
            LOD 200

            CGPROGRAM

            #pragma vertex vert
            #pragma fragment frag
            #include "UnityCG.cginc"

            struct a2v
            {
                float4 pos : POSITION;
                float2 uv : TEXCOORD0;
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
                float2 uv : TEXCOORD0;
            };

            v2f vert(a2v input)
            {
                v2f output;
                output.pos = UnityObjectToClipPos(input.pos);
                output.uv = input.uv;
                return output;
            }

            sampler2D _MainTex;
            float3 _TransparentColourKey;
            float _TransparencyTolerance;

            float4 frag(v2f input) : SV_Target
            {
                // What is the colour that *would* be rendered here?
                float4 colour = tex2D(_MainTex, input.uv);

                // Calculate the different in each component from the chosen transparency colour
                float deltaR = abs(colour.r - _TransparentColourKey.r);
                float deltaG = abs(colour.g - _TransparentColourKey.g);
                float deltaB = abs(colour.b - _TransparentColourKey.b);

                // If colour is within tolerance, write a transparent pixel
                if (deltaR < _TransparencyTolerance && deltaG < _TransparencyTolerance && deltaB < _TransparencyTolerance)
                {
                    return float4(0.0f, 0.0f, 0.0f, 0.0f);
                }

                // Otherwise, return the regular colour
                return colour;
            }
            ENDCG
        }
    }
}
```


新建一个材质，选择我们刚刚写好的Shader，将TransparentWindow挂载到摄像机上，摄像机的Clear Flags选择Solid Color，Background选择和材质的Transparent Color Key相同的颜色（建议选择与模型边缘颜色相近的颜色，不然会出现较明显的毛边），将材质拖拽给TransparentWindow的Material变量。   
![摄像机](https://img-blog.csdn.net/20170421141255821?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZGFyazAwODAw/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![材质](https://img-blog.csdn.net/20170421141307704?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZGFyazAwODAw/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![场景](https://img-blog.csdn.net/20170421141322158?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZGFyazAwODAw/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

运行之后可以看到背景颜色已经被扣掉（黑色其实是透明）：   
![编辑器效果](https://img-blog.csdn.net/20170421141431525?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZGFyazAwODAw/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

Build一下之后我们就可以看到类似桌面精灵的一个程序了：   
![效果](https://img-blog.csdn.net/20170421142522533?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZGFyazAwODAw/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 第二种方案：调用系统自带API

首先是去边框:

去边框就是调用系统自带函数来实现去边框的效果。获取窗体句柄FindWindow()设置窗体属性SetWindowLong()设置窗体大小置顶SetWindowPos()三个函数就可以解决置顶去边框的问题

窗体透明以及窗体穿透:

和DwmExtendFrameIntoClientArea()组合使用来实现透明玻璃并且穿透窗口的效果

专门下了个unityChan看下效果：
![效果](https://img-blog.csdn.net/20170325164842353?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcTQ5MzIwMTY4MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

以下是代码

```C# unity代码

using UnityEngine;
using System.Collections;
using System;
using System.Runtime.InteropServices;
using System.IO;
/// <summary>
/// 一共可选择三种样式
/// </summary>
public enum enumWinStyle
{
    /// <summary>
    /// 置顶
    /// </summary>
    WinTop,
    /// <summary>
    /// 置顶并且透明
    /// </summary>
    WinTopApha,
    /// <summary>
    /// 置顶透明并且可以穿透
    /// </summary>
    WinTopAphaPenetrate
}
public class WinSetting : MonoBehaviour
{
     #region Win函数常量
    private struct MARGINS
    {
        public int cxLeftWidth;
        public int cxRightWidth;
        public int cyTopHeight;
        public int cyBottomHeight;
    }
    [DllImport("user32.dll")]
    static extern IntPtr FindWindow(string lpClassName, string lpWindowName);
    [DllImport("user32.dll")]
    static extern int SetWindowLong(IntPtr hWnd, int nIndex, int dwNewLong);
    [DllImport("user32.dll")]
    static extern int GetWindowLong(IntPtr hWnd, int nIndex);
    [DllImport("user32.dll")]
    static extern int SetWindowPos(IntPtr hWnd, int hWndInsertAfter, int X, int Y, int cx, int cy, int uFlags); 
    [DllImport("user32.dll")]
    static extern int SetLayeredWindowAttributes(IntPtr hwnd, int crKey, int bAlpha, int dwFlags);
    [DllImport("Dwmapi.dll")]
    static extern uint DwmExtendFrameIntoClientArea(IntPtr hWnd, ref MARGINS margins);
    [DllImport("user32.dll")]
    private static extern int SetWindowLong(IntPtr hWnd, int nIndex, uint dwNewLong);
    private const int WS_POPUP = 0x800000;
    private const int GWL_EXSTYLE = -20;
    private const int GWL_STYLE = -16;
    private const int WS_EX_LAYERED = 0x00080000;
    private const int WS_BORDER = 0x00800000;
    private const int WS_CAPTION = 0x00C00000;
    private const int SWP_SHOWWINDOW = 0x0040;
    private const int LWA_COLORKEY = 0x00000001;
    private const int LWA_ALPHA = 0x00000002;
    private const int WS_EX_TRANSPARENT = 0x20;
    //
    private const int ULW_COLORKEY = 0x00000001;
    private const int ULW_ALPHA = 0x00000002;
    private const int ULW_OPAQUE = 0x00000004;
    private const int ULW_EX_NORESIZE = 0x00000008;
    #endregion
    //
    public string strProduct;//项目名称
    public enumWinStyle WinStyle = enumWinStyle.WinTop;//窗体样式
    //
    public int ResWidth;//窗口宽度
    public int ResHeight;//窗口高度
    //
    public int currentX;//窗口左上角坐标x
    public int currentY;//窗口左上角坐标y
    //
    private bool isApha;//是否透明
    private bool isAphaPenetrate;//是否要穿透窗体
    // Use this for initialization
    void Awake()
    {
        Screen.fullScreen = false;
        #if UNITY_EDITOR
        print("编辑模式不更改窗体");
        #else
        switch (WinStyle)
        {
            case enumWinStyle.WinTop:
                isApha = false;
                isAphaPenetrate = false;
                break;
            case enumWinStyle.WinTopApha:
                isApha = true;
                isAphaPenetrate = false;
                break;
            case enumWinStyle.WinTopAphaPenetrate:
                isApha = true;
                isAphaPenetrate = true;
                break;
        }
        //
        IntPtr hwnd = FindWindow(null, strProduct);
        //
        if (isApha)
        {
            //去边框并且透明
            SetWindowLong(hwnd, GWL_EXSTYLE, WS_EX_LAYERED);
            int intExTemp = GetWindowLong(hwnd, GWL_EXSTYLE);
            if (isAphaPenetrate)//是否透明穿透窗体
            {
                SetWindowLong(hwnd, GWL_EXSTYLE, intExTemp | WS_EX_TRANSPARENT | WS_EX_LAYERED);
            }
            //
            SetWindowLong(hwnd, GWL_STYLE, GetWindowLong(hwnd, GWL_STYLE) & ~WS_BORDER & ~WS_CAPTION);
            SetWindowPos(hwnd, -1, currentX, currentY, ResWidth, ResHeight, SWP_SHOWWINDOW);
            var margins = new MARGINS() { cxLeftWidth = -1 };
            //
            DwmExtendFrameIntoClientArea(hwnd, ref margins);
        }
        else
        {
            //单纯去边框
            SetWindowLong(hwnd, GWL_STYLE, WS_POPUP);
            SetWindowPos(hwnd, -1, currentX, currentY, ResWidth, ResHeight, SWP_SHOWWINDOW);
        }
        #endif
    }
}
```