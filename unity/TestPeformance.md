# 1. 快速入门

由于Unity Performance Test API还处于预览阶段，导致许多版本无法使用。目前可以使用的版本是Unity 2019.2。 Performance Test API版本是1.2.6-preview。

https://docs.unity3d.com/Packages/com.unity.test-framework.performance@1.3/manual/index.html

Packages/manifest.json加入：
``` "com.unity.test-framework.performance": "1.2.6-preview", ```


manifest.json内容应该如下：
```
{
  "dependencies": {
    "com.unity.test-framework.performance": "1.2.6-preview",
    "com.unity.collab-proxy": "1.2.16",
    "com.unity.ext.nunit": "1.0.0",
    "com.unity.ide.rider": "1.1.4",
    "com.unity.ide.vscode": "1.1.3",
    "com.unity.package-manager-ui": "2.2.0",
    "com.unity.test-framework": "1.0.18",
    "com.unity.textmeshpro": "2.0.1",
    "com.unity.timeline": "1.1.0",
    "com.unity.ugui": "1.0.0",
    "com.unity.modules.ai": "1.0.0",
    "com.unity.modules.androidjni": "1.0.0",
    "com.unity.modules.animation": "1.0.0",
    "com.unity.modules.assetbundle": "1.0.0",
    "com.unity.modules.audio": "1.0.0",
    "com.unity.modules.cloth": "1.0.0",
    "com.unity.modules.director": "1.0.0",
    "com.unity.modules.imageconversion": "1.0.0",
    "com.unity.modules.imgui": "1.0.0",
    "com.unity.modules.jsonserialize": "1.0.0",
    "com.unity.modules.particlesystem": "1.0.0",
    "com.unity.modules.physics": "1.0.0",
    "com.unity.modules.physics2d": "1.0.0",
    "com.unity.modules.screencapture": "1.0.0",
    "com.unity.modules.terrain": "1.0.0",
    "com.unity.modules.terrainphysics": "1.0.0",
    "com.unity.modules.tilemap": "1.0.0",
    "com.unity.modules.ui": "1.0.0",
    "com.unity.modules.uielements": "1.0.0",
    "com.unity.modules.umbra": "1.0.0",
    "com.unity.modules.unityanalytics": "1.0.0",
    "com.unity.modules.unitywebrequest": "1.0.0",
    "com.unity.modules.unitywebrequestassetbundle": "1.0.0",
    "com.unity.modules.unitywebrequestaudio": "1.0.0",
    "com.unity.modules.unitywebrequesttexture": "1.0.0",
    "com.unity.modules.unitywebrequestwww": "1.0.0",
    "com.unity.modules.vehicles": "1.0.0",
    "com.unity.modules.video": "1.0.0",
    "com.unity.modules.vr": "1.0.0",
    "com.unity.modules.wind": "1.0.0",
    "com.unity.modules.xr": "1.0.0"
  }
}
```

来下载安装Performance Test API。

创建Tests Assembly文件夹，在Tests Defination中的Assembly Definition References中加入Unity.PerformanceTesting。创建一个Test脚本。脚本引用
using Unity.PerformanceTesting;

## 1.1 Measure.Method 测量方法效率

* WarmupCount（int n） -收集测量值之前要执行的次数。如果未指定，则执行默认预热。默认预热时间为7毫秒。但是，如果在这段时间内完成了少于3个方法执行，则预热将延长到3个方法执行完成。
* MeasurementCount（int n） -要捕获的采样数。如果未指定，则默认值为7。
* IterationsPerMeasurement（int n） -要使用的每个采样的方法执行次数。如果未指定该值，则该方法将被执行若干次（也可能是一次）次，直到大于1毫秒过去。
* GC（） -如果指定，将测量垃圾回收分配调用的总数。

我们以测试生成一千个Cube的效率为例。

```C#
void SpawnAThousandCube()
{
    for(int i = 0; i < 1000; i++)
    {
        GameObject.CreatePrimitive(PrimitiveType.Cube);
    }
    Debug.Log("A Thousand Cube Created!");
}
```

### 1.1.1 使用默认值测量

```C#
[Test, Performance, Version("1")] //注意这里需要加入PerformanceAttribute为方法标注效能测试，而不是单纯的单元测试。VersionAttribute是可选的，定义测试版本。
public void PerformanceTestSimplePasses()
{
    Measure.Method(SpawnAThousandCube).Run();
}
```

TestRunner运行测试。
结果如下（和解释）：

```
PerformanceTestSimplePasses (0.305s)//总用时0.305s
---
A Thousand Cube Created!
A Thousand Cube Created!
A Thousand Cube Created!//前三次执行是默认的WarmUp
A Thousand Cube Created!//后七次支持是实际测量采样
A Thousand Cube Created!
A Thousand Cube Created!
A Thousand Cube Created!
A Thousand Cube Created!
A Thousand Cube Created!
A Thousand Cube Created!

 //结果概括。默认测试方法执行时间、单位是毫秒。打印出测量结果中的中位数、最小值、最大值、平均值、标准差、零值次数、采样次数、总和
Time Millisecond Median:20.39 Min:19.53 Max:28.99 Avg:21.63 Std:3.09 Zeroes:0 SampleCount: 7 Sum: 151.40

//详细结果
##performancetestresult:{ 
//测试名 ${namespace}.{classname}.{functionname}
   "TestName":"Tests.PerformanceTest.PerformanceTestSimplePasses",
//测试分类 
   "TestCategories":[ 
      "Performance"
   ],
   "TestVersion":"1", //测试版本，可以通过VersionAttribute控制
   "StartTime":1575441992210.7285,//开始测试时间
   "EndTime":1575441992469.58,//结束测试时间
   "SampleGroups":[ //测试结果
      { 
         "Definition":{ //测试标准
            "Name":"Time",//测量值 时间
            "SampleUnit":2,//SampleUnit.Millisecond = 2， 测试单位毫秒
            "AggregationType":3,//Aggregation.Median = 3， 中位数优先
            "Threshold":0.15,
            "IncreaseIsBetter":false,
            "Percentile":0.0,
            "FailOnBaseline":true
         },
         "Samples":[ //每次采样的测量结果
            21.155500000000005,
            21.658200000000009,
            20.39139999999999,
            19.53030000000001,
            19.542,
            20.13579999999999,
            28.99170000000001
         ],
         "Min":19.53030000000001,//最小值
         "Max":28.99170000000001,//最大值
         "Median":20.39139999999999,//中间值
         "Average":21.62927142857143,//平均值
         "StandardDeviation":3.0927522729828906,//方差
         "PercentileValue":0.0,//百分比
         "Sum":151.4049,//总和
         "Zeroes":0,
         "SampleCount":7//采样次数
      }
   ]
}
```

很遗憾的是，现在Unity PerformanceTesting没有把performanceresult规格化，在Unity中显示是很长的一行，阅读体验很差，可以找个json规格化的web规格化下。

### 1.1.2 自定义Measure.Method属性

```C#
[Test, Performance, Version("2")] 
public void PerformanceTestSimplePasses()
{
    Measure.Method(SpawnAThousandCube)
               .WarmupCount(10)	//热身执行方法次数
               .MeasurementCount(10) //采样次数
               .IterationsPerMeasurement(5) //每次采样执行次数
               .GC() //测量GC调用
               .Run();
}
```

结果如下：

```
PerformanceTestSimplePasses (1.656s)
---
A Thousand Cube Created! x 60 //热身10次+采样次数10次*每次采样执行5次 = 60 个打印

//GC调用次数，测量结果
Time.GC() None Median:1485.00 Min:1485.00 Max:1492.00 Avg:1487.30 Std:3.13 Zeroes:0 SampleCount: 10 Sum: 14873.00
//测量结果， 注意此时每次采样 方法执行五次 每次采样测量的是这五次执行的总体时间
Time Millisecond Median:123.18 Min:101.80 Max:140.01 Avg:120.67 Std:11.55 Zeroes:0 SampleCount: 10 Sum: 1206.74


//详细结果
##performancetestresult:{ 
   "TestName":"Tests.PerformanceTest.PerformanceTestSimplePasses",
   "TestCategories":[ 
      "Performance"
   ],
   "TestVersion":"2",
   "StartTime":1575444388173.724,
   "EndTime":1575444389701.6646,
   "SampleGroups":[ 
      { 
         "Definition":{ 
            "Name":"Time.GC()",
            "SampleUnit":8,
            "AggregationType":3,
            "Threshold":0.15,
            "IncreaseIsBetter":false,
            "Percentile":0.0,
            "FailOnBaseline":true
         },
         "Samples":[ 
            1487.0,
            1492.0,
            1485.0,
            1485.0,
            1485.0,
            1492.0,
            1485.0,
            1485.0,
            1485.0,
            1492.0
         ],
         "Min":1485.0,
         "Max":1492.0,
         "Median":1485.0,
         "Average":1487.3,
         "StandardDeviation":3.132091952673165,
         "PercentileValue":0.0,
         "Sum":14873.0,
         "Zeroes":0,
         "SampleCount":10
      },
      { 
         "Definition":{ 
            "Name":"Time",
            "SampleUnit":2,
            "AggregationType":3,
            "Threshold":0.15,
            "IncreaseIsBetter":false,
            "Percentile":0.0,
            "FailOnBaseline":true
         },
         "Samples":[ 
            117.64949999999999,
            118.23939999999999,
            140.0077,
            101.79700000000003,
            135.38440000000004,
            105.08489999999995,
            112.89920000000007,
            126.04239999999982,
            126.45709999999986,
            123.17879999999991
         ],
         "Min":101.79700000000003,
         "Max":140.0077,
         "Median":123.17879999999991,
         "Average":120.67403999999995,
         "StandardDeviation":11.548200601409715,
         "PercentileValue":0.0,
         "Sum":1206.7403999999995,
         "Zeroes":0,
         "SampleCount":10
      }
   ]
}
```

## 1.2 Measure.Frames 测量每帧时间

* WarmupCount（int n） -收集测量值之前要执行的次数。如果未指定，则执行默认预热。默认预热80毫秒。但是，如果在这段时间内渲染的完整帧少于3个，则预热将等到渲染完整的3个完整帧。
* MeasurementCount（int n） -捕获帧的采样数。如果未指定此值，则将尽可能多地捕获帧，直到经过约500 ms。
* DontRecordFrametime（） -禁用帧时间测量。
* ProfilerMarkers（...） -每帧样本分析器标记。不适用于Deep Profile和Profiler.BeginSample()。
* Scope（） -测量给定Cortinue范围内的帧时间。

### 1.2.1 使用默认值测量

```C#
[UnityTest, Performance, Version("1")]
public IEnumerator PerformanceTestWithEnumeratorPasses()
{
    UnityEngine.QualitySettings.vSyncCount = 0;//关闭VSync
    Application.targetFrameRate = -1;//目标帧数设置为平台默认帧数,或者一个足够大的数

    SpawnAThousandCube();

//测量渲染帧时间，添加相机用于渲染
    Camera camera = new GameObject("Test Performance Camera").AddComponent<Camera>();
    camera.transform.position = new Vector3(0, 0, -10.0f);

//使用默认值测量
    yield return Measure.Frames().Run();
}
```

测量结果：

```
PerformanceTestWithEnumeratorPasses (0.940s)//总用时0.904秒
---
A Thousand Cube Created! 
//结果概要：每帧时间单位毫秒
Time Millisecond Median:3.98 Min:2.94 Max:4.51 Avg:3.98 Std:0.19 Zeroes:0 SampleCount: 120 Sum: 477.68

###performancetestresult: ...//就不贴了、太长

1.2.2 更换采样测试目标，禁用测量帧时间
{
    .....

//使用默认值测量
   yield return Measure.Frames()
                .ProfilerMarkers("Drawing")//测量摄像机绘制时长
                .DontRecordFrametime()//禁止测量帧时长
                .Run();
}
```

有关ProfilerMaker的标记使用，将在后面的详细讲。结果如下：
```
PerformanceTestWithEnumeratorPasses (0.614s)
---
A Thousand Cube Created! 
Drawing Millisecond Median:0.90 Min:0.00 Max:1.34 Avg:0.91 Std:0.16 Zeroes:1 SampleCount: 98 Sum: 89.25
```

### 1.2.3 指定相关真参数控制

```C#
{
.....

yield return Measure.Frames()
                .WarmupCount(5)//热身帧数设定
                .MeasurementCount(10)//采样帧数设定
                .Run();
}
```

结果如下：

```
PerformanceTestWithEnumeratorPasses (0.202s)
---
A Thousand Cube Created! 
Time Millisecond Median:6.92 Min:3.78 Max:10.39 Avg:6.26 Std:2.40 Zeroes:0 SampleCount: 10 Sum: 62.58

1.3 Measure.Scope() 适用于同步和协程方法，记录Scope内部时间。
[UnityTest, Performance, Version("1.3")]
public IEnumerator PerformanceTestWithEnumeratorPasses()
{
     UnityEngine.QualitySettings.vSyncCount = 0;
     Application.targetFrameRate = -1;

     using (Measure.Scope())
     {
         for(int i = 0; i < 10; i++)//连续10帧，每帧生成1000个Cube
         {
             SpawnAThousandCube();
             yield return null;
          }
     }
}
```

结果如下

```
PerformanceTestWithEnumeratorPasses (0.369s)
---
A Thousand Cube Created! x10

Time 298.66 Milliseco
```

## 1.4 Measure.ProfilerMakers()

用于Profiler分析其数据记录。Profiler标记计时将自动确定，并在using语句范围内进行采样。名称SampleGroupDefinition应与Profiler标记名称匹配。请注意，Deep Profiler和Editor Profiler分析不可用。Profiler.BeginSample()不支持。

现在我们准备测试Gameobject实例化的效率，让我们稍微修改SpawnAThousandCube方法，使用实例化的方法来生成。

```C#
 	void SpawnAThousandCube()
        {
            var go = GameObject.CreatePrimitive(PrimitiveType.Cube);
            for (int i = 0; i < 999; i++)
            {
                GameObject.Instantiate(go);
            }
            Debug.Log("A Thousand Cube Created! ");
        }
测试代码如下：
        [UnityTest, Performance, Version("1.4")]
        public IEnumerator PerformanceTestWithEnumeratorPasses()
        {
            UnityEngine.QualitySettings.vSyncCount = 0;
            Application.targetFrameRate = -1;

//定义Profiler标记
            SampleGroupDefinition[] definitions = {
                new SampleGroupDefinition("Instantiate")
                ,new SampleGroupDefinition("Instantiate.Copy")
                ,new SampleGroupDefinition("Instantiate.Produce")
                ,new SampleGroupDefinition("Instantiate.Awake")
            };

            //测量Profiler的标记
            using (Measure.ProfilerMarkers(definitions))
            {
                for (int i = 0; i < 10; i++)
                {
                    SpawnAThousandCube();
                    yield return null;
                }              
            }
        }
```

结果如下
```
PerformanceTestWithEnumeratorPasses (0.338s)
---
A Thousand Cube Created!  x10
Instantiate Millisecond Median:13.92 Min:0.00 Max:20.28 Avg:13.90 Std:5.28 Zeroes:1 SampleCount: 10 Sum: 139.00
Instantiate.Copy Millisecond Median:2.04 Min:0.00 Max:3.14 Avg:1.94 Std:0.73 Zeroes:1 SampleCount: 10 Sum: 19.45
Instantiate.Produce Millisecond Median:5.82 Min:0.00 Max:12.43 Avg:6.29 Std:2.91 Zeroes:1 SampleCount: 10 Sum: 62.88
Instantiate.Awake Millisecond Median:5.26 Min:0.00 Max:7.85 Avg:5.00 Std:1.84 Zeroes:1 SampleCount: 10 Sum: 50.03
```

## 1.5 Measure.Custom()

如果要记录帧时间，方法时间或Profiler标记之外的测量对象，使用Measure.Custom测量。它需要样本组定义。示例：
```C#
[Test, Performance, Version("1.5")]
public void PerformanceTestSimplePasses()
{
SpawnAThousandCube();
   var definition = new SampleGroupDefinition("TotalAllocatedMemory", SampleUnit.Megabyte);
    Measure.Custom(definition, UnityEngine.Profiling.Profiler.GetTotalAllocatedMemoryLong() / 1048576f);
}
```
结果：
```
PerformanceTestSimplePasses (0.058s)
---
A Thousand Cube Created! 
TotalAllocatedMemory 378.32 Megabyte
```
## 1.6 查看性能报告

查看性能测试报告

“性能测试报告”窗口显示了各个测试运行的详细分类。这可以用来评估每个测试的稳定性。它提供了记录在样品组中的每个单个样品的可视化以及所选测试的摘要统计信息。您可以通过转到Windows>Analysis>Peformance Test Report来打开窗口。
性能测试报告分为两个视图：测试视图和样本组视图。

测试视图：提供所有测试的列表。可以单击每个列对视图进行排序。列值显示偏差最大的样品组。

*名称 -测试名称。
*偏差 -偏差是通过将标准偏差除以样品组的中位数而得出的值。它显示了具有最高“偏差”值的样本组。用于定义测试的稳定性。
*标准偏差 -样本组中样本的标准偏差。它显示了具有最高标准偏差的样品组。
*样品组视图：在“测试视图”中可视化所选测试的样品组。提供
*样品组摘要显示给定样品组的最小值，最大值和中值。
*条形图中显示的样本按时间排序，蓝线表示中位数。
*箱形图显示给定样本组的样本的上四分位数（75％）和下四分位数（25％），最小值，最大值和中位数。

## 1.7 Tips

删除所有QualitySettings，但在项目设置下删除一个。否则，在不同平台上运行时，您可能具有不同的配置。如果每个平台需要不同的设置，请确保已按预期进行设置。
在QualitySettings下禁用VSync。某些平台（例如Android）具有强制VSync，这是不可能的。

禁用硬件报告 PlayerSettings -> Other -> Disable HW Reporting

如果不测量渲染，请删除相机再以协程模式运行

### 1.7.1 使用Unity Test Framework 中PrebuildSet可以用来统一设置环境。示例：

```C#
public class TestsWithPrebuildStep : IPrebuildSetup
{
    public void Setup()
    {
        // this code is executed before entering playmode or the player is executed
// 可以设置相应的scene、质量属性等
    }
}

public class MyAmazingPerformanceTest
{
    [Test, Performance]
    [PrebuildSetup(typeof(TestsWithPrebuildStep))]//添加PrebuildSetup属性
    public void Test()
    {
        ...
    }
}
```

### 1.7.2 其他例子

场景测量
```C#
    [UnityTest, Performance]
    public IEnumerator Rendering_SampleScene()
    {
        using(Measure.Scope(new SampleGroupDefinition("Setup.LoadScene")))
        {
            SceneManager.LoadScene("SampleScene");
        }
        yield return null;

        yield return Measure.Frames().Run();
    }
```

自定义度量以捕获分配的总内存和保留的内存

```C#
    [Test, Performance, Version("1")]
    public void Measure_Empty()
    {
        var allocated = new SampleGroupDefinition("TotalAllocatedMemory", SampleUnit.Megabyte);
        var reserved = new SampleGroupDefinition("TotalReservedMemory", SampleUnit.Megabyte);
        Measure.Custom(allocated, Profiler.GetTotalAllocatedMemoryLong() / 1048576f);
        Measure.Custom(reserved, Profiler.GetTotalReservedMemoryLong() / 1048576f);
    }
```

# 2 Profiler Script
TODO
# 3 Unity Performance Benchmark
TODO
