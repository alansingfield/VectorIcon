# VectorIcon
## WPF vector based icons
![VectorIcon screenshot](/images/screenshot.png)

``` xml
<WrapPanel ItemHeight="60">
  <local:VectorIcon Style="{StaticResource CopyIcon}" Foreground="Green"/>
  <local:VectorIcon Style="{StaticResource CarIcon}" Foreground="Blue"/>
  <local:VectorIcon Style="{StaticResource BugleIcon}" Foreground="Red"/>
  <local:VectorIcon Style="{StaticResource CarIcon}" Foreground="YellowGreen"/>
</WrapPanel>
```

This project demonstrates how to style vector-based icons in WPF. It's far harder than it should be, the documentation is particularly unhelpful when explaining this.

I've been using the icons from https://materialdesignicons.com/. They give you the vector XAML for the icons, however, in the form supplied you would have to repeat the full block of text each time you want to use it, and the size is fixed.

``` xml
<Canvas xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml" 
        Width="24" 
        Height="24">
  <Path Data="M18,18H6V6H18M18,4H6A2,2 0 0,0 4,6V18A2,2 0 0,0 6,20H18A2,2 0 0,0 20,18V6C20,4.89 19.1,4 18,4Z" />
</Canvas>
```

I surrounded the Canvas with a Viewbox and created a UserControl so I could specify the colour:

``` xml
<UserControl ... DataContext="{Binding RelativeSource={RelativeSource Self}}" >
  <Canvas>
    <Path Data="M18,18H6V6H18M18,4H6A2,2 0 0,0 4,6V18A2,2 0 0,0 6,20H18A2,2 0 0,0 20,18V6C20,4.89 19.1,4 18,4Z" 
          Fill="{Binding Foreground}"/>
```

Easy.

So I thought it would be straightforward to add a String DependencyProperty and bind to the Path.Data property.
To keep things simple I started with a static string resource.

``` xml
<UserControl xmlns:sys="clr-namespace:System;assembly=mscorlib">
    <UserControl.Resources>
      <sys:String x:Key="MyPath">M18,18H6V6H18M18,4H6A2,2 0 0,0 4,6V18A2,2 0 0,0 6,20H18A2,2 0 0,0 20,18V6C20,4.89 19.1,4 18,4Z</sys:String>
    </UserControl.Resources>
  ...
<Path Data="{StaticResource MyPath}" Fill="{Binding Foreground}" />
```

And it all came crashing down.

```
An object of the type "System.String" cannot be applied to a property that expects the type "System.Windows.Media.Geometry".
```

The Data property is not a string. Although you can assign a string to it in the XAML, there is a lot of magic going on behind the scenes at XAML parse time to convert this string into a Geometry.

What we need to do instead is declare a &lt;PathGeometry/&gt; with the Figures property.

``` xml
<UserControl xmlns:sys="clr-namespace:System;assembly=mscorlib">
    <UserControl.Resources>
      <PathGeometry x:Key="MyPath" Figures="M18,18H6V6H18M18,4H6A2,2 0 0,0 4,6V18A2,2 0 0,0 6,20H18A2,2 0 0,0 20,18V6C20,4.89 19.1,4 18,4Z" />
    </UserControl.Resources>
  ...
<Path Data="{StaticResource MyPath}" Fill="{Binding Foreground}" />
```

Once we've done this we can extract out into a DependencyProperty and style it.

``` cs
    public partial class VectorIcon : UserControl
    {
        public Geometry Geometry
        {
            get { return (Geometry)GetValue(GeometryProperty); }
            set { SetValue(GeometryProperty, value); }
        }

        public static readonly DependencyProperty GeometryProperty =
            DependencyProperty.Register(
                name: nameof(Geometry),
                propertyType: typeof(Geometry),
                ownerType: typeof(VectorIcon),
                typeMetadata: new PropertyMetadata(null));
```



``` xml
<UserControl x:Class="Didsbury.VectorIconExample.VectorIcon">
  <Viewbox>
    <Canvas>
      <Path Data="{Binding Geometry}" 
            Fill="{Binding Foreground}"/>
    </Canvas>
  </Viewbox>
</UserControl>

  <Style x:Key="CarIcon" 
         TargetType="local:VectorIcon">
    <Style.Setters>
      <Setter Property="Geometry">
        <Setter.Value>
          <PathGeometry Figures="M19, ..., L16.91,13Z" />
        </Setter.Value>
      </Setter>
    </Style.Setters>
  </Style>

...
<local:VectorIcon Style="{StaticResource CarIcon}" Foreground="Green"/>
<local:VectorIcon Style="{StaticResource CarIcon}" Foreground="Blue"/>
```

## Just one more thing...

In the examples above, I've used the DataContext="{Binding RelativeSource={RelativeSource Self}}" trick. This is simpler for a quick explanation but it fails when you try use a DataTrigger, the real DataContext gets overwritten by the UserControl object. The proper design looks like:

``` xml
<UserControl ...
             x:Name="Root">
  <Viewbox>
    <Canvas Width=  "{Binding PathWidth,  ElementName=Root}" 
            Height= "{Binding PathHeight, ElementName=Root}">
      <Path Data=   "{Binding Geometry,   ElementName=Root}" 
            Fill=   "{Binding Foreground, ElementName=Root}"/>
    </Canvas>
  </Viewbox>
</UserControl>

```
