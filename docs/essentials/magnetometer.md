---
title: Xamarin.Essentials： 磁力仪
description: 中 Xamarin.Essentials 的磁力仪类可让你监视设备的磁力仪传感器，指示相对于地球磁场的设备的方向。
ms.assetid: 64DD0D41-03E2-40DD-9EC8-101CA0ED852B
author: jamesmontemagno
ms.author: jamont
ms.date: 05/04/2018
ms.openlocfilehash: 3827b9a57ec2667a8716f5b56bfb4631b979d43a
ms.sourcegitcommit: 51c274f37369d8965b68ff587e1c2d9865f85da7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/30/2018
ms.locfileid: "39353784"
---
# <a name="xamarinessentials-magnetometer"></a>Xamarin.Essentials： 磁力仪

![预发行版 NuGet](~/media/shared/pre-release.png)

**磁力仪**类可让你监视设备的磁力仪传感器指示相对于地球磁场的设备的方向。

## <a name="using-magnetometer"></a>使用磁力仪

在类中添加对 Xamarin.Essentials 的引用：

```csharp
using Xamarin.Essentials;
```

磁力仪功能适用于通过调用`Start`和`Stop`方法来侦听对磁力仪的更改。 任何更改发送回通过`ReadingChanged`事件。 下面是示例用法：

```csharp

public class MagnetometerTest
{
    // Set speed delay for monitoring changes.
    SensorSpeed speed = SensorSpeed.UI;

    public MagnetometerTest()
    {
        // Register for reading changes.
        Magnetometer.ReadingChanged += Magnetometer_ReadingChanged;
    }

    void Magnetometer_ReadingChanged(object sender, MagnetometerChangedEventArgs e)
    {
        var data = e.Reading;
        // Process MagneticField X, Y, and Z
        Console.WriteLine($"Reading: X: {data.MagneticField.X}, Y: {data.MagneticField.Y}, Z: {data.MagneticField.Z}");
    }

    public void ToggleMagnetometer()
    {
        try
        {
            if (Magnetometer.IsMonitoring)
              Magnetometer.Stop();
            else
              Magnetometer.Start(speed);
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

在 microteslas 中返回所有数据。

[!include[](~/essentials/includes/sensor-speed.md)]

## <a name="api"></a>API

- [磁力仪源代码](https://github.com/xamarin/Essentials/tree/master/Xamarin.Essentials/Magnetometer)
- [磁力仪 API 文档](xref:Xamarin.Essentials.Magnetometer)
