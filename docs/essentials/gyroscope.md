---
title: Xamarin.Essentials： 陀螺仪
description: 在 Xamarin.Essentials 陀螺仪类可让你监视设备的陀螺仪传感器测量设备的三个主要轴的旋转。
ms.assetid: DA4F968A-D988-41F5-8745-1BEE693660A1
author: jamesmontemagno
ms.author: jamont
ms.date: 05/04/2018
ms.openlocfilehash: f1e1199ae32158889ec569eb5f7e9742f37d45d4
ms.sourcegitcommit: 51c274f37369d8965b68ff587e1c2d9865f85da7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/30/2018
ms.locfileid: "39353621"
---
# <a name="xamarinessentials-gyroscope"></a>Xamarin.Essentials： 陀螺仪

![预发行版 NuGet](~/media/shared/pre-release.png)

**陀螺仪**类可让你监视设备的陀螺仪传感器是围绕设备的三个主要轴的旋转角度。

## <a name="using-gyroscope"></a>使用陀螺仪

在类中添加对 Xamarin.Essentials 的引用：

```csharp
using Xamarin.Essentials;
```

陀螺仪功能适用于通过调用`Start`和`Stop`方法来侦听对陀螺仪的更改。 任何更改发送回通过`ReadingChanged`事件。 下面是示例用法：

```csharp

public class GyroscopeTest
{
    // Set speed delay for monitoring changes.
    SensorSpeed speed = SensorSpeed.UI;

    public GyroscopeTest()
    {
        // Register for reading changes.
        Gyroscope.ReadingChanged += Gyroscope_ReadingChanged;
    }

    void Gyroscope_ReadingChanged(object sender, GyroscopeChangedEventArgs e)
    {
        var data = e.Reading;
        // Process Angular Velocity X, Y, and Z
        Console.WriteLine($"Reading: X: {data.AngularVelocity.X}, Y: {data.AngularVelocity.Y}, Z: {data.AngularVelocity.Z}");
    }

    public void ToggleGyroscope()
    {
        try
        {
            if (Gyroscope.IsMonitoring)
              Gyroscope.Stop();
            else
              Gyroscope.Start(speed);
        }
        catch (FeatureNotSupportedException fnsEx)
        {
            // Feature not supported on device
        }
        catch (Exception ex)
        {
            // Other error has occurred.
        }
    }
}
```

[!include[](~/essentials/includes/sensor-speed.md)]

## <a name="api"></a>API

- [陀螺仪源代码](https://github.com/xamarin/Essentials/tree/master/Xamarin.Essentials/Gyroscope)
- [陀螺仪 API 文档](xref:Xamarin.Essentials.Gyroscope)
