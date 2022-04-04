---
layout:     post
title:      "C#-Delegate and event"
date:       2022-4-04
author:     "hmboy"
header-img: "img/post-bg-2015.jpg"
tags:
    - C#
---



# C# 委托

## Delegate

### 为什么需要委托

- 比如有个发消息的业务代码

  ```c#
  void SendMessage(string mes,SendWayEnum option)
  {
      switch(option)
      {
          case SendWayEnum.Email:
              sendByEmail(mes);
              break;
              ...
           
      }
  }
  void sendByEmail(string mes)
  {
      
  }
  void sendBySMS(string mes)
  {
      
  }
  ```

  - 随着发送的方式越来越多，导致不得不修改枚举和`SendMessage`函数

### 委托定义

-  是存有对某个方法的引用的一种引用类型变量，类似c中函数指针

  - ```c#
    //定义委托 （可以代表方法的类型）
    delegate void SendMesDelegate(SendWayEnum option)
    ```

    - 委托在编译的时候确实会编译成类。因为`delegate`是一个类，所以在任何可以声明类的地方都可以声明委托

  - ```c#
    SendMesDelegate delegate1;
    int a;
    ```

    - 此时`SendMesDelegate `好比`int `，都可以直接声明变量

  - ```c#
    void SendMessage(string mes,SendMesDelegate del)
    {
        del(mes);
    }
    ```

    ```c#
       SendMesDelegate delegate1;
        delegate1 = sendByEmail; // 先给委托类型的变量赋值
        delegate1 += sendBySMS;   // 给此委托变量再绑定一个方法
    
         // 将先后调用 sendByEmail 与 sendBySMS 方法
        SendMessage("Hello World!", delegate1);
    	//也可以直接调用
    	delegate1("Hello World!");
    
    ```

    - 委托不同于`int`的一个特性：可以将多个方法赋给同一个委托，或者叫将多个方法绑定到同一个委托，当调用这个委托的时候，将依次调用其所绑定的方法

- 以后添加新的发送方式的话 `SendMessage`也不会再修改

## event

- 由于委托第一个方法注册用`=`，是赋值语法，因为要进行实例化，第二个方法注册则用的是`+=`。但是，不管是赋值还是注册，都是将方法绑定到委托上，除了调用时先后顺序不同，再没有任何的分别。

  - delegate可以使用`=`对所有已经订阅的取消，只保留`=`后新的订阅

- 由此对此进行了封装，引出了事件，event只能使用`+=``-=`不能直接使用`=`

  - ```c#
    //类似属性封装
    private name = "HM";
    public Name{get{ return name}; set{ name = value};}
    
    private delegate void SendMesDelegate(SendWayEnum option);
    public event SendMesDelegate sendMesEvent;
    ```

- 事件类似于声明一个进行了封装的委托类型的变量

  - ```c#
    //定义委托 （可以代表方法的类型）
    delegate void SendMesDelegate(SendWayEnum option);
    event SendMesDelegate sendMesEvent;
    void main()
    {
        sendMesEvent sendEvent;
        sendEvent += sendByEmail; // 先给委托类型的变量赋值
        sendEvent += sendBySMS;   // 给此委托变量再绑定一个方法
    
         // 将先后调用 sendByEmail 与 sendBySMS 方法
        SendMessage("Hello World!", sendEvent);
    	//也可以直接调用
    	sendEvent("Hello World!");
    }
    ```

### Observer设计模式

- 假设我们有个高档的热水器，我们给它通上电，当水温超过95度的时候：
  - 1、扬声器会开始发出语音，告诉你水的温度；
  - 2、液晶屏也会改变水温的显示，来提示水已经快烧开了。

- 现在我们需要写个程序来模拟这个烧水的过程，我们将定义一个类来代表热水器，我们管它叫：`Heater`，它有代表水温的字段，叫做`temperature`；当然，还有必不可少的给水加热方法`BoilWater()`，一个发出语音警报的方法`MakeAlert()`，一个显示水温的方法，`ShowMsg()`。

- ```c#
  namespace Delegate {
      
      class Heater
      {
          private int temperature; // 水温
          // 烧水
          public void BoilWater() 
          {
              for (int i = 0; i <= 100; i++) {
                 temperature = i;
                 if (temperature > 95) {undefined
                     MakeAlert(temperature);
                     ShowMsg(temperature);
                  }
              }
          }
  
          // 发出语音警报
          private void MakeAlert(int param) {undefined
             Console.WriteLine("Alarm：嘀嘀嘀，水已经 {0} 度了：" , param);
          }
  
          // 显示水温
          private void ShowMsg(int param)
          {
             Console.WriteLine("Display：水快开了，当前温度：{0}度。" , param);
          }
  	}
  
      class Program 
      {
          static void Main() 
          {
             Heater ht = new Heater();
             ht.BoilWater();
          }
      }
  }
  ```

#### Observer设计模式简介

上面的例子显然能完成我们之前描述的工作，但是却并不够好。现在假设热水器由三部分组成：热水器、警报器、显示器，它们来自于不同厂商并进行了组装。那么，应该是**热水器**仅仅负责烧水，它不能发出警报也不能显示水温；在水烧开时由**警报器**发出警报、**显示器**显示提示和水温。

这时候，上面的例子就应该变成这个样子：  

```c#
// 热水器
public class Heater {
  private int temperature;
    
  // 烧水
  private void BoilWater()
  {
    for (int i = 0; i <= 100; i++)
    {
      temperature = i;
    }
  }
}

// 警报器
public class Alarm
{
  private void MakeAlert(int param) {undefined
    Console.WriteLine("Alarm：嘀嘀嘀，水已经 {0} 度了：" , param);
  }
}

// 显示器
public class Display{
  private void ShowMsg(int param) 
  {
    Console.WriteLine("Display：水已烧开，当前温度：{0}度。" , param);
  }
}
```

这里就出现了一个问题：如何在水烧开的时候通知报警器和显示器？在继续进行之前，我们先了解一下Observer设计模式，Observer设计模式中主要包括如下两类对象：

1. Subject：监视对象，它往往包含着其他对象所感兴趣的内容。在本范例中，热水器就是一个监视对象，它包含的其他对象所感兴趣的内容，就是temprature字段，当这个字段的值快到100时，会不断把数据发给监视它的对象。
2. Observer：监视者，它监视Subject，当Subject中的某件事发生的时候，会告知Observer，而Observer则会采取相应的行动。在本范例中，Observer有警报器和显示器，它们采取的行动分别是发出警报和显示水温。

在本例中，事情发生的顺序应该是这样的：

1. 警报器和显示器告诉热水器，它对它的温度比较感兴趣(注册)。
2. 热水器知道后保留对警报器和显示器的引用。
3. 热水器进行烧水这一动作，当水温超过95度时，通过对警报器和显示器的引用，自动调用警报器的`MakeAlert()`方法、显示器的`ShowMsg()`方法。

类似这样的例子是很多的，GOF对它进行了抽象，称为Observer设计模式：**Observer设计模式是为了定义对象间的一种一对多的依赖关系，以便于当一个对象的状态改变时，其他依赖于它的对象会被自动告知并更新。Observer模式是一种松耦合的设计模式。**

#### 实现范例的Observer设计模式

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Delegate 
{
  // 热水器
  public class Heater 
  {
    private int temperature;
    public delegate void BoilHandler(int param);  //声明委托
    public event BoilHandler BoilEvent;    //声明事件
    // 烧水
    public void BoilWater() 
    {
      for (int i = 0; i <= 100; i++) 
      {
           temperature = i;
           if (temperature > 95)
           {
             if (BoilEvent != null) //如果有对象注册
             { 
               BoilEvent(temperature); //调用所有注册对象的方法
             }
           }
      }
   }
  }

  // 警报器
  public class Alarm
  {
      
    public void MakeAlert(int param) 
    {
      Console.WriteLine("Alarm：嘀嘀嘀，水已经 {0} 度了：", param);
    }
  }

  // 显示器
  public class Display 
  {
    public static void ShowMsg(int param) //静态方法
    { 
      Console.WriteLine("Display：水快烧开了，当前温度：{0}度。", param);
    }
  }
  
  class Program 
  {
    static void Main() 
    {
      Heater heater = new Heater();
      Alarm alarm = new Alarm();

      heater.BoilEvent += alarm.MakeAlert;  //注册方法
      heater.BoilEvent += (new Alarm()).MakeAlert;  //给匿名对象注册方法
      heater.BoilEvent += Display.ShowMsg;    //注册静态方法

      heater.BoilWater();  //烧水，会自动调用注册过对象的方法
    }
  }
}
```

## .Net Framework中的委托与事件

尽管上面的范例很好地完成了我们想要完成的工作，但是我们不仅疑惑：为什么.Net Framework 中的事件模型和上面的不同？为什么有很多的`EventArgs`参数? 比如说，如果我们不光想获得热水器的温度，还想在Observer端(警报器或者显示器)方法中获得它的生产日期、型号、价格，那么委托和方法的声明都会变得很麻烦，而如果我们将热水器的引用传给警报器的方法，就可以在方法中直接访问热水器了。

现在我们改写之前的范例，让它符合 .Net Framework 的规范：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Delegate 
{
  // 热水器
  public class Heater
  {
    private int temperature;
    public string type = "RealFire 001";    // 添加型号作为演示
    public string area = "China Xian";     // 添加产地作为演示
    //声明委托
    public delegate void BoiledEventHandler(Object sender, BoiledEventArgs e);
    public event BoiledEventHandler Boiled; //声明事件

    // 定义BoiledEventArgs类，传递给Observer所感兴趣的信息
    public class BoiledEventArgs : EventArgs 
    {
     	  public readonly int temperature;
          public BoiledEventArgs(int temperature) 
          {
           this.temperature = temperature;
          }
    }

    // 可以供继承自 Heater 的类重写，以便继承类拒绝其他对象对它的监视
    protected virtual void OnBoiled(BoiledEventArgs e) 
    {
      if (Boiled != null)// 如果有对象注册
      { 
       	Boiled(this, e); // 调用所有注册对象的方法
      }
    }
   
    // 烧水。
    public void BoilWater()
    {
      for (int i = 0; i <= 100; i++) 
      {
           temperature = i;
           if (temperature > 95)
           {
             //建立BoiledEventArgs 对象。
             BoiledEventArgs e = new BoiledEventArgs(temperature);
             OnBoiled(e); // 调用 OnBolied方法
       		}
      }
    }
  }

  // 警报器
  public class Alarm {
    public void MakeAlert(Object sender, Heater.BoiledEventArgs e) 
    {
      Heater heater = (Heater)sender;   //这里是不是很熟悉呢？
      //访问 sender 中的公共字段
      Console.WriteLine("Alarm：{0} - {1}: ", heater.area, heater.type);
      Console.WriteLine("Alarm: 嘀嘀嘀，水已经 {0} 度了：", e.temperature);
      Console.WriteLine();
    }
  }

  // 显示器
  public class Display 
  {
    public static void ShowMsg(Object sender, Heater.BoiledEventArgs e)
    {  
      Heater heater = (Heater)sender;
      Console.WriteLine("Display：{0} - {1}: ", heater.area, heater.type);
      Console.WriteLine("Display：水快烧开了，当前温度：{0}度。", e.temperature);
      Console.WriteLine();
    }
  }

  class Program 
  {
    static void Main()
    {
      Heater heater = new Heater();
      Alarm alarm = new Alarm();

      heater.Boiled += alarm.MakeAlert;  //注册方法
      heater.Boiled += (new Alarm()).MakeAlert;   //给匿名对象注册方法
      heater.Boiled += new Heater.BoiledEventHandler(alarm.MakeAlert);  //也可以这么注册
      heater.Boiled += Display.ShowMsg;    //注册静态方法

      heater.BoilWater();  //烧水，会自动调用注册过对象的方法
    }
  }
}
```

## EventHandler

- 内部采用委托来实现
- **EventHandler相当于可以带很多参数的无返回值的委托**

- ```c#
  public event EventHandler<StudentClass> showStudentInfo;
  StudentClass student = new StudentClass("12","lp","man");
  
  Test(showStudentInfo);
  private void Test(EventHandler eventHandler)
  {
      showStudentInfo(this ,student);
  }
  //其中 写类对参数列表类进行继承
  public class StudentClass : EventArgs
  {
      public string age;
      public string name;
      public string sex;
  
      public StudentClass(string _age,string _name,string _sex)
      {
          age = _age;
          name = _name;
          sex = _sex;
      }
  }
  ```

## Action

- Action 内部是由delegate实现的

- **Action相当于可以带很多参数的无返回值的委托**

  - ```C#
    public Action<string, int> action;
    public void Action1(string str,int a)
    {
        print(str + a.ToString()); 
    }
    action = Action1;
    ```

- 当普通的delegate定义的参数与Action个数、类型一致时，两者实现的功能是一样的。只是Action的方式更加简洁

  ```c#
  public delegate void DoDelegate(object parm);
  public DoDelegate DoMethod;
  
  public Action<object> doAction4OneParm;
  public Action<object, object> doAction4TwoParm;
  
  private void Form1_Load(object sender, EventArgs e)
  {
      DoMethod += DoTestMetohd;  //普通委托(由于委托定义时给定一个参数，故此处匹配一个参数的方法)
      doAction4OneParm += DoTestMetohd;  //Action委托(此处匹配一个参数的方法)
      doAction4TwoParm += DoTestMetohd;  //Action委托(此处匹配两个参数的方法)
  }
  
  private void DoTestMetohd(object parm)
  {
      MessageBox.Show(Convert.ToString(parm));
  }
  
  private void DoTestMetohd(object parm1, object parm2)
  {
      MessageBox.Show(Convert.ToString(parm1 + " " + parm2));
  }
  ```

- 而Action与delegate更重要的一个区别在于泛型，即Action的内部使用了泛型+委托，且泛型的方法的参数个数可扩展到16个

- ```c#
  1 namespace System
   2 {
   3     [System.Diagnostics.CodeAnalysis.SuppressMessage("Microsoft.Design", "CA1005:AvoidExcessiveParametersOnGenericTypes")]
   4     public delegate void Action<in T1, in T2, in T3, in T4, in T5, in T6, in T7, in T8, in T9>(T1 arg1, T2 arg2, T3 arg3, T4 arg4, T5 arg5, T6 arg6, T7 arg7, T8 arg8, T9 arg9);
   5 
  ```

## Func

- **Func可以带很多参数，但是必须带返回值的委托**

- Func<T1,T2,T3,T4,TResult> func；

- ```c#
  public Func<string , bool> func;
  public bool Func1(string str)
  {
      print(str);
  
      return true;
  }
  func = Func1;
  ```





----

以上摘自

- [(6条消息) C# 委托(delegate)和事件(event)详解__Adwore的博客-CSDN博客_c# delegate event](https://blog.csdn.net/life_is_crazy/article/details/78206166)
- [C#:关于Action和Func以及Eventhandler的区别 - 怪力~乱神 - 博客园 (cnblogs.com)](https://www.cnblogs.com/zbyglls/p/11798808.html)



