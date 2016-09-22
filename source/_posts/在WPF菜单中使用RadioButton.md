title: 在WPF菜单中使用RadioButton
date: 2016-09-22 14:00:51
tags: [WPF]
---
有时候我们需要在一组菜单中只能选中一个菜单,类似于RadioButtonGroup的功能.
<!--more-->
#####**第一种方式**
有一种简单的方法可以实现，保存一组Checkable的MenuItem然后在代码中判断哪个一个菜单选中，并设置其它菜单为末选中。
```xml
<Menu>
    <MenuItem Header="Radio">
         <MenuItem Header="Radio1" IsCheckable="True" Click="Menu_Click" />
         <MenuItem Header="Radio2" IsCheckable="True" Click="Menu_Click" />
         <MenuItem Header="Radio3" IsCheckable="True" Click="Menu_Click" />
    </MenuItem>
</Menu>
```
C#代码操作菜单.
```C#
List<MenuItem> Radios = new List<MenuItem>();//存储同一组RadioMenu的集合
public void Menu_Click(object sender, RoutedEventArgs e)
{
    //取得当前引发事件的菜单
    MenuItem menu = sender as MenuItem;
    //循环设置选中属性
    foreach( var m in Radios)
    {
         if(m != menu) m.IsChecked = false;
    }
}
```

这样就可以达到模拟RadioButtonGroup的功能。但是这种方式看起来一点也不简洁。下面使用另一种通用的方法来达到目的。

#####**扩展菜单类方法**

既然RadioButton是采用分组的方式来设置，那么我们是不是可以在MenuItem中简单的使用GroupName属性来实现分组？
例如：
```xml
<MenuItem IsCheckable="True" GroupName="Group1" />
<MenuItem IsCheckable="True" GroupName="Group2" />
<MenuItem IsCheckable="True" GroupName="Group3" />
```
上面这种形式是可以的，就是使用一个额外的类属性来设置GroupName。

首先我们需要一个类来包含GroupName这个属性，而我们这个属性同时还要设置为可绑定，就需要继承自DependencyObject。类中还需要一个名为GroupName的属性。那么这个类大概就是下面这样子：
```C#
namespace RadioMenu.Ext
{
    class MenuItemExtensions: DependencyObject
    {
        //包含一个名为GroupName的属性
        public static readonly DependencyProperty GroupNameProperty =
            DependencyProperty.RegisterAttached("GroupName",
                                         typeof(String),
                                         typeof(MenuItemExtensions),
                                         new PropertyMetadata(String.Empty, OnGroupNameChanged));
        //GroupName的Setter属性
        public static void SetGroupName(MenuItem element, String value)
        {
            element.SetValue(GroupNameProperty, value);
        }

        //getter属性
        public static String GetGroupName(MenuItem element)
        {
            return element.GetValue(GroupNameProperty).ToString();
        }
    }
}
```
我们还需要一个字典来存储菜单和它所在的分组：
```C#
public static Dictionary<MenuItem, String> ElementToGroupNames = new Dictionary<MenuItem, String>();//Key为菜单，value为分组名称
```
在实现GroupName属性时我们有一个当属性变化调用的函数没有实现，现在来实现它：
```C#
private static void OnGroupNameChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
        {
            //取得引起属性变化的菜单项
            var menuItem = d as MenuItem;
            if (menuItem != null)
            {
                String newGroupName = e.NewValue.ToString();
                String oldGroupName = e.OldValue.ToString();
                if (String.IsNullOrEmpty(newGroupName))
                {
                    //如果新的分组为Null或空，当前menuItem已不在任何分组中，则移除他
                    RemoveCheckboxFromGrouping(menuItem);
                }
                else
                {
                    //如果新的分组名和旧的分组名不一样，则把当前menuItem移动到新的分组中
                    if (newGroupName != oldGroupName)
                    {
                        if (!String.IsNullOrEmpty(oldGroupName))
                        {
                            //从旧的分组删除
                            RemoveCheckboxFromGrouping(menuItem);
                        }
                        //加入新的分组
                        ElementToGroupNames.Add(menuItem, e.NewValue.ToString());
                        //增加菜单事件
                        menuItem.Checked += MenuItemChecked;
                    }
                }
            }
        }
```
下一步实现移除分组和菜单事件，菜单事件中需要判断哪一个菜单项是选中的，同时设置同一组其它菜单项为不选中。
```C#
 private static void RemoveCheckboxFromGrouping(MenuItem checkBox)
        {
            ElementToGroupNames.Remove(checkBox);
            checkBox.Checked -= MenuItemChecked;
        }

        static void MenuItemChecked(object sender, RoutedEventArgs e)
        {
            var menuItem = e.OriginalSource as MenuItem;
            foreach (var item in ElementToGroupNames)
            {
                if (item.Key != menuItem && item.Value == GetGroupName(menuItem))
                {
                //设置其它菜单项都为不选中
                    item.Key.IsChecked = false;
                }
            }
        }
```
最后在使用中只需要在xaml文件中设置相关的属性即可。
```xml
<Window 
   ...
   xmlns:local="clr-namespace:RadioMenu.Ext">
   ...
   <MenuItem IsCheckable="True" local:MenuItemExtensions.GroupName="Group1" />
   <MenuItem IsCheckable="True" local:MenuItemExtensions.GroupName="Group1" />
   <MenuItem IsCheckable="True" local:MenuItemExtensions.GroupName="Group1" />
   ...
</Window>
```
这样就实现了菜单项的RadioMenu，而且更加通用。