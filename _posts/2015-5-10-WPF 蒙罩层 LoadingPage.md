---
layout: post
title: "WPF 蒙罩层 LoadingPage"
description: "无论是在PC客户端，移动端，网站，在遇到长时间处理的时候都会需要用到蒙罩层，让用户有更好的体现。今天上网逛了一下各位前辈网友..."
tags: [wpf]
---

### 前言
　　无论是在PC客户端，移动端，网站，在遇到长时间处理的时候都会需要用到蒙罩层，让用户有更好的体现。今天上网逛了一下各位前辈网友的蒙罩层的实现方式，觉得有很多都搞复杂了（利用前台代码+后台代码+数学计算），无疑增加了维护的难度。然而，本人参考了各位前辈的实现以后，自己实现了一个可重用LoadingPage控件，欢迎各位下载使用。

 

### 需求
需求先行是必须的，我的目标是做成怎样一个效果呢？

1. 是一个控件，可以在.NET各环境中得以重用。

2. 可配置，例如颜色，大小，提醒字符串等等的属性，用户可以自定义，以满足用户所在情况的需求。

3. 大小比例自适应，不同大小的窗口载体，能自动改变自身大小比例。

4. 效果全部xaml实现，全部集中于xaml可控制难度不会大，维护起来方便，用户拷贝xaml也方便。

 

### 解决方法
　
1. 新建WPF用户控件库进行开发。

2. 使用依赖项属性，然后前台xaml使用属性绑定来实现。

3. 使用ViewBox控件（该控件能够自动缩放内容）。

4. 在xaml中的写动画代码。

### 结果展示
　　
![](http://upload-images.jianshu.io/upload_images/1952818-9ac315efcd93a4b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 制作过程
　
1. 新建一个"WPF用户控件库"工程，新建一个WPF控件。（这步不解释）

2. 为了实现蒙罩效果，我们把控件的背景弄成黑色背景，并且透明度为0.2。
```
        <UserControl.Background>
            <SolidColorBrush Color="Black" Opacity="0.2" ></SolidColorBrush>
        </UserControl.Background>
```
3. 然后就是先利用Canvas作为背景，在其上画一个由小圆圈构成的大圈，使用控件Ellipse。
```
        <Canvas RenderTransformOrigin="0.5,0.5" 
                    HorizontalAlignment="Center"  x:Name="loadCancas"
                    VerticalAlignment="Center" Width="120"  
                    Height="120" >
                <Canvas.Resources>
                    <Style TargetType="Ellipse">
                        <Setter Property="Width" Value="10" ></Setter>
                        <Setter Property="Height" Value="10" ></Setter>
                        <Setter Property="Canvas.Left" Value="30"></Setter>
                        <Setter Property="Canvas.Top" Value="30"></Setter>
                        <Setter Property="Stretch" Value="Fill"></Setter>
                        <Setter Property="Fill" Value="Blue"></Setter>
                        <Setter Property="RenderTransformOrigin" Value="3,3"></Setter>
                    </Style>
                </Canvas.Resources>
                <Ellipse >
                </Ellipse>
                <Ellipse Opacity="0.9">
                    <Ellipse.RenderTransform>
                        <TransformGroup>
                            <RotateTransform Angle="36"/>
                        </TransformGroup>
                    </Ellipse.RenderTransform>
                </Ellipse>
                <Ellipse Opacity="0.8">
                    <Ellipse.RenderTransform>
                        <TransformGroup>
                            <RotateTransform Angle="72"/>
                        </TransformGroup>
                    </Ellipse.RenderTransform>
                </Ellipse>
                <Ellipse Opacity="0.7">
                    <Ellipse.RenderTransform>
                        <TransformGroup>
                            <RotateTransform Angle="108"/>
                        </TransformGroup>
                    </Ellipse.RenderTransform>
                </Ellipse>
                <Ellipse Opacity="0.6">
                    <Ellipse.RenderTransform>
                        <TransformGroup>
                            <RotateTransform Angle="144"/>
                        </TransformGroup>
                    </Ellipse.RenderTransform>
                </Ellipse>
                <Ellipse Opacity="0.5">
                    <Ellipse.RenderTransform>
                        <TransformGroup>
                            <RotateTransform Angle="180"/>
                        </TransformGroup>
                    </Ellipse.RenderTransform>
                </Ellipse>
                <Ellipse Opacity="0.4">
                    <Ellipse.RenderTransform>
                        <TransformGroup>
                            <RotateTransform Angle="216"/>
                        </TransformGroup>
                    </Ellipse.RenderTransform>
                </Ellipse>
                <Ellipse Opacity="0.3">
                    <Ellipse.RenderTransform>
                        <TransformGroup>
                            <RotateTransform Angle="252"/>
                        </TransformGroup>
                    </Ellipse.RenderTransform>
                </Ellipse>
                <Ellipse Opacity="0.2">
                    <Ellipse.RenderTransform>
                        <TransformGroup>
                            <RotateTransform Angle="288"/>
                        </TransformGroup>
                    </Ellipse.RenderTransform>
                </Ellipse>
                <Ellipse Opacity="0.1" >
                    <Ellipse.RenderTransform>
                        <TransformGroup>
                            <RotateTransform Angle="324"/>
                        </TransformGroup>
                    </Ellipse.RenderTransform>
                </Ellipse>
                <Canvas.RenderTransform>
                    <TransformGroup>
                        <RotateTransform x:Name="SpinnerRotate"  
                         Angle="0" />
                    </TransformGroup>
                </Canvas.RenderTransform>
            </Canvas>
        </Grid>
```
这样就形成了一个圈，然后为了实现目标3(内容能自动改变大小)，使用一个viewBox作为容器，包住这个Canvas。（不贴代码了）

4. 旋转动画编写。因为我在Canvas画了一个圈，然而我只需无限旋转Canvas便可实现像小圆圈在动一样。下面看一下Canvas的触发器，在触发器中实现动画的编写。
```
        <Canvas.Triggers>
        　　<EventTrigger RoutedEvent="Loaded">
        　　　　<BeginStoryboard Name="loadAnimation">
        　　　　　　<Storyboard>
        　　　　　　　　<DoubleAnimation Storyboard.TargetName="loadCancas" Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[0].(RotateTransform.Angle)" From="360" To="0" RepeatBehavior="Forever" Duration="0:0:3"></DoubleAnimation>
                  </Storyboard>
              </BeginStoryboard>
            </EventTrigger>
            <EventTrigger RoutedEvent="Unloaded">
        　　　　<StopStoryboard BeginStoryboardName="loadAnimation"></StopStoryboard>
            </EventTrigger>
        </Canvas.Triggers>
```

5. 属性可配置，使用依赖属性，并且在xaml中写绑定，下面先看后台代码中的依赖项属性的定义，然后前台绑定就补贴了，统一在附件中可以看到。
```
        public partial class LoadingPage : UserControl
        {
            public LoadingPage()
            {
                InitializeComponent();
            }

            #region 加载圆圈的margin
            [DescriptionAttribute("加载圆圈的margin"), CategoryAttribute("扩展"), DefaultValueAttribute(0)]
            public string LoadCirclesMargin
            {
                get { return (string)GetValue(LoadCirclesMarginProperty); }
                set { SetValue(LoadCirclesMarginProperty, value); }
            }


            public static readonly DependencyProperty LoadCirclesMarginProperty =
                DependencyProperty.Register("LoadCirclesMargin", typeof(string), typeof(LoadingPage),
                new FrameworkPropertyMetadata("50"));
            #endregion

            #region 加载中的提示
            [DescriptionAttribute("加载中的提示"), CategoryAttribute("扩展"), DefaultValueAttribute(0)]
            public string LoadingText
            {
                get { return (string)GetValue(LoadingTextProperty); }
                set { SetValue(LoadingTextProperty, value); }
            }


            public static readonly DependencyProperty LoadingTextProperty =
                DependencyProperty.Register("LoadingText", typeof(string), typeof(LoadingPage),
                new FrameworkPropertyMetadata("加载中"));
            #endregion

            #region 加载中的提示的字体大小
            [DescriptionAttribute("加载中的提示的字体大小"), CategoryAttribute("扩展"), DefaultValueAttribute(0)]
            public int LoadingTextFontSize
            {
                get { return (int)GetValue(LoadingTextFontSizeProperty); }
                set { SetValue(LoadingTextFontSizeProperty, value); }
            }


            public static readonly DependencyProperty LoadingTextFontSizeProperty =
                DependencyProperty.Register("LoadingTextFontSize", typeof(int), typeof(LoadingPage),
                new FrameworkPropertyMetadata(12));
            #endregion

            #region 圆圈的颜色
            [DescriptionAttribute("圆圈的颜色"), CategoryAttribute("扩展"), DefaultValueAttribute(0)]
            public Brush CirclesBrush
            {
                get { return (Brush)GetValue(CirclesBrushProperty); }
                set { SetValue(CirclesBrushProperty, value); }
            }


            public static readonly DependencyProperty CirclesBrushProperty =
                DependencyProperty.Register("CirclesBrush", typeof(Brush), typeof(LoadingPage),
                new FrameworkPropertyMetadata(Brushes.Black));
            #endregion

            #region 加载中的提示的字体颜色
            [DescriptionAttribute("加载中的提示的字体颜色"), CategoryAttribute("扩展"), DefaultValueAttribute(0)]
            public Brush LoadingTextForeground
            {
                get { return (Brush)GetValue(LoadingTextForegroundProperty); }
                set { SetValue(LoadingTextForegroundProperty, value); }
            }


            public static readonly DependencyProperty LoadingTextForegroundProperty =
                DependencyProperty.Register("LoadingTextForeground", typeof(Brush), typeof(LoadingPage),
                new FrameworkPropertyMetadata(Brushes.DarkSlateGray));
            #endregion
        }
```
 
　　

大功告成！！！！上面的代码都是为了展示原理而分拆出来的零碎代码，如果想使用该控件，可以点下面的

[下载Demo下载](http://files.cnblogs.com/files/Jarvin/Loading.rar)