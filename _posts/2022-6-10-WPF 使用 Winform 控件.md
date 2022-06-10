---
layout:     post
title:      "C#-WPF 使用 winform 控件"
date:       2022-6-10
author:     "hmboy"
header-img: "img/post-bg-2015.jpg"
tags:
    - C#
---

## Wpf 使用 winform 控件

1. 使用WindowsFormsHost
2. 添加System.Windows.Forms dll
3. `using System.Windows.Forms.Integration;`

## 问题

### Winform控件代码加入不显示

- ex:

  ```c#
   WindowsFormsHost windowsFormsHost = new WindowsFormsHost();
  grid.Child = windowsFormsHost;
  ```

- method:

  - 对应windows xmal中加入`AllowTransparency = false`

  - 但会失去⽆边框效果。
    需要牺牲⾃由放⼤缩⼩的功能的解决办法就是加上`ResizeMode=“NoResize”`，这样就能实现⽆边框的同时显⽰`WindowsFormsHost`，也可以⽤`ResizeMode=“CanMinimize”`，这个保留最⼤最⼩化功能。

### Winform控件始终置顶("空域 Airspace")

- WPF原生的控件无法对其遮盖，更别说透明了, 调整Panel.ZIndex 无效

- method:

  - 在WPF项目中添加`Microsoft.DwayneNeed.dll`和`Microsoft.DwayneNeed.Win32.dll`引用([Github地址](https://github.com/MahApps/Microsoft.DwayneNeed))

  - ```xaml
    <xmlns:interop=clr-namespace:Microsoft.DwayneNeed.Interop;assembly=Microsoft.DwayneNeed>
    </xmlns>
    ```

  - ```xaml
    <airspace:AirspaceDecorator AirspaceMode="Redirect" IsInputRedirectionEnabled="True" IsOutputRedirectionEnabled="True">
        <WindowsFormsHost Name="FormsHost">
            ...
        </WindowsFormsHost>
    </airspace:AirspaceDecorator>
    ```

  - 但也会出现在airspace:AirspaceDecorator AirspaceMode="Redirect" 属性这样设置时就会出现控件显示不完整的问题；

    在设置为AirspaceMode="Clip" 或AirspaceMode="None"时就没问题，但此时不能控制...




---

***Tip: WPF尽量不要使用WindowsFormsHost !***

