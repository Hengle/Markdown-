# VRDevice文档说明 1127
[TOC]

## 一、接口调用流程
将组件内容文件夹内所有文件及文件夹拷贝到根目录下，添加对VRDevice.dll的引用，添加命名空间GosuncnVRDevice。
>1.创建设备实例：``` new VRDevice() ``` 
>2.监听委托事件 ``` VRDevice.HasInited;VRDevice.RightClick ```
>3.初始化设备 ``` VRDevice.Init() ```
>4.推送场景数据 ``` VRDevice.InputFrameData() ```
>5.······其他操作
>
>6.关闭设备 ``` VRDevice.CloseVRDevice() ```

## 二、接口函数说明
>### 1.public VRDevice (IntPtr Container, DeviceType DType, int DeviceID)
>>创建VR设备
>>Container：承载设备输出容器（建议为pannel）的句柄，关闭该容器时必须调用CloseVRDevice方法关闭设备
>>DType：VR设备类型
>>DeviceID：VR设备ID(自定义值，实为socket通信端口号，因此必须保证该端口空闲)

>### 2.public IntPtr Init(int ProcessId)
>>初始化设备
>>ProcessId: 主程序进程id，用于主程序异常退出时VR设备自杀进程
>>返回值：VRDevice窗口句柄，可用于窗口大小变化等操作

>### 3.public string InputFrameData(string ImageUrl)
>>送入场景帧数据
>>ImageUrl：图像文件Url（支持网络地址或本地文件路径）
>>返回值：推送结果描述

>### 4.public string InputFrameData(byte[] data)
>>送入场景帧数据重载
>>data：RGB图像数据
>>返回值：推送结果描述

>### 5.public string InputFrameData(IntPtr pBuf,uint nSize)
>>送入场景帧数据重载
>>pBuf：RGB图像数据内存指针
>>nSize：数据长度
>>返回值：推送结果描述

>### 6.public void CloseVRDevice()
>>关闭设备：退出VRDevice，关闭socket通信
>>无参无返回值

>### 7.public static Vector3 GetlocalPosition(Vector3 point)
>>获取指定坐标点在窗口的位置
>>point：坐标点值
>>返回值：窗口位置坐标值

>### 8.public static Vector3 GetWorldPositon(float x ,float y)
>>获取屏幕坐标对应的VR空间世界坐标
>>x：屏幕坐标x值
>>y：屏幕坐标y值
>>返回值：VR空间世界坐标值

>### 9.public void AutoCameraRotation(bool auto,int horizontal, int vertical ,bool canViewReversal = false)
>>控制视角相机转动
>>auto：是否开始转动
>>horizontal：水平按下转动速度，单位为度每秒
>>vertical：垂直方向转动速度，单位为度每秒
>>canViewReversal：视角是否允许上下颠倒，可选参数，默认为false，即不可颠倒

>### 10.public void LookTarget(Vector3 target, int FOV = 0)
>>视角导航至目标点
>>target：目标点三维坐标值
>>FOV：相机视角，可选参数，默认为0代表保持当前视角值

>### 11.public void CameraZoom(int value)
>>相机缩放控制，通过改变相机视野大小实现缩放，相机视野取值范围设定为[20,100]
>>value：速度值，每秒缩放度数，正值为放大，反之缩小

>>### 12.public void CameraZoomTarget(float value)
>>视野缩放到指定系数
>>value：缩放系数值，取值-1到1(1为放大原来一倍，-1为缩小原来一倍)。

>>### 13.public void StopCameraControl()
>>停止云台操作，包含转动和缩放

>>### 14.public int GetFieldOfView()
>>获取当前VR空间相机视角值
>>返回值：视角度数值

>>### 15.public void PauseVR(bool value)
>>暂停VRDevice
>>value：true为暂停，false为取消暂停

## 三、委托回调
>### 1.public Action HasInited
>>设备初始化完成Action，用于监听设备初始化是否完成

>### 2.public Action<Vector3> RightClick
>>右键事件Action，监听鼠标右键事件
>>Vector3：右键时VR空间坐标值

## 四、枚举说明
>### 1.VR设备类型枚举
>>```CSharp
>>public enum DeviceType
>>{
>>    /// <summary>
>>    /// 图片数据（bytes）型设备
>>    /// </summary>
>>    ImageData = 0,
>>
>>    /// <summary>
>>    /// 图片url型设备
>>    /// </summary>
>>    ImageUrl = 1,
>>
>>    /// <summary>
>>    /// 视频流型设备
>>    /// </summary>
>>    Video = 2
>>}
>>```

## 五、Vector3类
>三维坐标（向量）类：实现类部分向量基本操作

## 六、调用实例参考
```CSharp
        [DllImport("User32.dll")]
        static extern bool MoveWindow(IntPtr handle, int x, int y, int width, int height, bool redraw);

        internal delegate int WindowEnumProc(IntPtr hwnd, IntPtr lparam);
        [DllImport("user32.dll")]
        internal static extern bool EnumChildWindows(IntPtr hwnd, WindowEnumProc func, IntPtr lParam);

        [DllImport("user32.dll")]
        static extern int SendMessage(IntPtr hWnd, int msg, IntPtr wParam, IntPtr lParam);

        private IntPtr unityHWND = IntPtr.Zero;
        private VRDevice vrDevice;
        private const int WM_ACTIVATE = 0x0006;
        private readonly IntPtr WA_ACTIVE = new IntPtr(1);
        private readonly IntPtr WA_INACTIVE = new IntPtr(0);
        private Vector3 tempPoint;

        public MyForm()
        {
            InitializeComponent();

            //定义设备，类型为图片URL，socket端口为500
            vrDevice = new VRDevice(panel1.Handle, DeviceType.ImageUrl, 500);

            //监听初始化完成和鼠标右键委托
            vrDevice.HasInited += InitFinish;
            vrDevice.RightClick += HandleRightClick;

            //初始化设备，并获取窗口句柄
            unityHWND = vrDevice.Init(Process.GetCurrentProcess().Id);
            tempPoint = new Vector3(0, 0, 0);
        }

        private void HandleRightClick(Vector3 point)
        {
            tempPoint = point;
        }

        private void InitFinish()
        {
            //推送场景图像数据，注意必须要初始化完成后才能进行
            //string mes = vrDevice.InputFrameData("http://192.168.38.25:8088/test.png");
            string mes = vrDevice.InputFrameData("D:/leaf/MyServerRoot/test.png");
        }

        private void ActivateUnityWindow()
        {
            SendMessage(unityHWND, WM_ACTIVATE, WA_ACTIVE, IntPtr.Zero);
        }

        private void DeactivateUnityWindow()
        {
            SendMessage(unityHWND, WM_ACTIVATE, WA_INACTIVE, IntPtr.Zero);
        }

        //改变窗口大小
        private void panel1_Resize(object sender, EventArgs e)
        {
            MoveWindow(unityHWND, 0, 0, panel1.Width, panel1.Height, true);
            ActivateUnityWindow();
        }

        // 窗口关闭时关闭设备
        private void MyForm_FormClosed(object sender, FormClosedEventArgs e)
        {
            vrDevice.CloseVRDevice();
        }

        private void MyForm_Activated(object sender, EventArgs e)
        {
            ActivateUnityWindow();
        }

        private void MyForm_Deactivate(object sender, EventArgs e)
        {
            DeactivateUnityWindow();
        }
```
