`图片尚未上传`

# 一些前提说明
本文档使用markdown语法, 查看时可以使用相应的编辑器来凸显层次。  
本文档先写的是我自己用到的, 如果有官网有但这里没有的, 那只是我没用到, 后续会继续跟进。

## Unity GUI系统的分类
首先它可以分成两个独立的部分：非Editor类与Editor类.    
其中每部分又可以分成两个独立的部分：GUI类和GUILayout类.  
也就是一共四部分: GUI、GUILayout、EditorGUI、EditorGUILayout. 这四个部分，每个拿出来都能相对完整个完成一个UI制做.  

其中的GUI系统下的组件的名字基本上只用"GUI"开头，而GUILayout系统下的组件的名字基本上是"GUILayout"开头的。  

以下内容主要以`EditorGUILayout`来进行叙述.

## Unity Editor的参考文档
[Editor类中`UIElements`和`IMGUI`用法的区别与该类的基本方法和参数](https://docs.unity3d.com/ScriptReference/Editor.html)  
[官方文档-Custom Editors](https://docs.unity3d.com/Manual/editor-CustomEditors.html)  
[2019.1中UIElements的新增功能](https://blogs.unity3d.com/2019/04/23/whats-new-with-uielements-in-2019-1/)  

## 关于GUI的深层说明
Unity的GUI系统说直白点就是将Html和Css进行了封装处理, GUI系统的四部分均是不同实现的标签封装(大体没差距), 而后提供了两套编写方法: `IMGUI`和`UIElements`.    
`UIElements`在编写时使用的是`UXml`和`Css`, 其实就类似于写`html + css`.  
`IMGUI`在编写时使用的是C#代码，类似于`FLutter语言`.  

以下内容主要以`IMGUI`的写法来进行叙述.

# PropertyAttribute说明(题外)
[PropertyAttribute](https://docs.unity3d.com/ScriptReference/PropertyAttribute.html)可以和[PropertyDrawer](https://docs.unity3d.com/ScriptReference/PropertyDrawer.html)一起使用，来定义脚本中的变量在`Inspector`中如何展示

## [Multiline()]
让string类型参数在Inspector界面中的展示由text变为textarea
```
[Multiline()]
public string a;
```

## [Range()]
让数字型参数在Inspector界面中的展示由text变为滑条,且限制数值范围
```
[Range(0, 100)]
public int a;
```

## [Header("xx")]
在Inspector界面中等同于将`xx`变为`docx文档`中标题一样的存在，也等同于`md文档`中的`# xx`

其后面必须跟有属性定义, 不能在空类中直接编写.
```
[Header("xx")]
public int a;
```

## [Tooltip("xx")]
给指定的属性添加描述, 在Inspector界面中, 将鼠标放在`a`属性上将会展示内容

```
[Tooltip("xx")]
public int a;
```

# Editor类与EditorGUILayout类
## Editor
[官方文档-Editor](https://docs.unity3d.com/ScriptReference/Editor.html)  
`Editor`类继承自`ScriptableObject`, 位于`UnityEditor`命名空间内, 在官方文档中有些方法的描述可以对应理解.

一个`IMGUI`的实现案例如下, 后续将基于这个案例叙述.  
```
using UnityEditor;

public Class Player : MonoBehaviour {

}

[CustomEditor(typeof(Player))]
public class PlayerEditor : Editor {
    SerializedProperty damageProp;

    private void Awake () {
    }

    void OnEnable() {
        Player player = (Player) target;
        Object[] targetsList = targets;
        damageProp = serializedObject.FindProperty("damage");

        GameObject[] gameObjects = Selection.gameObjects;
    }

    public override void OnInspectorGUI() {
        // 这行代码表示Inspector将以Player脚本在Unity中默认的原有形式展现内容
        // 如果想要自己自定义就把这行注释/删掉
        base.OnInspectorGUI();

        // 以下是声明自定义的方式, 一旦声明且中间没有内容，Inspector中将展示为空的
        serializedObject.Update();
        // ... 这里填写你自定义的展示
        serializedObject.ApplyModifiedProperties();
    }
}
```

**CustomEditor(typeof(xxx))**  
代表当前的Editor是针对哪个类来定义的, 在后续使用该类时将显示为当前Editor规定的样子

**target**  
在上述代码场景中，挂有`Player脚本`的一个GameObject，这里获取的就是你选中的那个。

**targets**  
如果不开启`[CanEditMultipleObjects()]`，`targets`和`target`效果将会相同，不过类型是数组罢了。  
如果开启`[CanEditMultipleObjects()]`, 通过`按住shift`选择的所有带有`Player脚本`的GameObject都将被获取, 我没搞懂获取到的先后顺序是怎么安排的。

**serializedObject**  
`serializedObject`和`target`表示的差不多，区别在于前者支持`[CanEditMultipleObjects()]`，而`target`不支持。  

支持`[CanEditMultipleObjects()]`意味着能够多对象操作，按住shift选择多个带有Player脚本的gameobject，然后修改参数。  
具体的`target`和`serializedObject`在使用上的区别可以在[文档](https://docs.unity3d.com/ScriptReference/Editor.html)中轻松找到。

**OnInspectorGUI()**  
在其中编写将要展示到`Inspector`上的内容, 当鼠标放到`Inspector`区域内后, 移动时就会不断的触发该方法。

**Awake()、OnEnable()、OnDestroy()、OnDisable()**  
这些方法继承自`ScriptableObject`, 见详情各自的使用方式.  
[ScriptableObject.Awake()使用方式](https://docs.unity3d.com/ScriptReference/ScriptableObject.Awake.html)  
[ScriptableObject.OnEnable()使用方式](https://docs.unity3d.com/ScriptReference/ScriptableObject.OnEnable.html)  
[ScriptableObject.OnDestroy()使用方式](https://docs.unity3d.com/ScriptReference/ScriptableObject.OnDestroy.html)  
[ScriptableObject.OnDisable()使用方式](https://docs.unity3d.com/ScriptReference/ScriptableObject.OnDisable.html)  
[更多相关讨论](https://forum.unity.com/threads/scriptableobject-behaviour-discussion-how-scriptable-objects-work.541212/)

**Selection接口**  
通过该类可以调用一些关于目前选中的GameObject集合的事物,具体方法自己试吧。

## EditorGUILayout
[官方文档-EditorGUILayout](https://docs.unity3d.com/ScriptReference/EditorGUILayout.html)  
很多方法都是配对的, 就如同声明if代码的开始和结束位置一样.  

### BeginBuildTargetSelectionGrouping & EndBuildTargetSelectionGrouping
将当前系统上已有的编辑平台以可切换到导航的形式展示, 通过选中具体的编辑平台来获取到选中的平台名称.  
[BeginBuildTargetSelectionGrouping官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.BeginBuildTargetSelectionGrouping.html)  

示例代码:
```
BuildTargetGroup selectedBuildTargetGroup = EditorGUILayout.BeginBuildTargetSelectionGrouping ();
if (selectedBuildTargetGroup == BuildTargetGroup.Android) {
    EditorGUILayout.LabelField ("Android specific things");
}

if (selectedBuildTargetGroup == BuildTargetGroup.Standalone) {
    EditorGUILayout.LabelField ("Standalone specific things");
}

EditorGUILayout.EndBuildTargetSelectionGrouping ();
```
![示例代码效果图](https://docs.unity3d.com/StaticFiles/ScriptRefImages/BuildTargetGroupExampleExtended.png)

### BeginFadeGroup & EndFadeGroup
将特定字符串作为折叠帘的标题, 通过配合`Toggle`来`选中`折叠帘的标题来进行带有渐入渐出效果的折叠/展示, 点击后返回参数如下.
- 0: 折叠
- 1: 展开

返回参数并不是直接由0到1, 而是有一个过渡.  
[BeginFadeGroup官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.BeginFadeGroup.html)  

官方示例代码(改): 
```
public class PlayerEditor : Editor {
    UnityEditor.AnimatedValues.AnimBool m_ShowExtraFields;

    private void OnEnable () {
        // 设置初始化
        m_ShowExtraFields = new UnityEditor.AnimatedValues.AnimBool (true);
        // 添加监听值变化事件 (像极了js、Vue的checkbox标签监听...)
        m_ShowExtraFields.valueChanged.AddListener (Repaint);
    }

    public override void OnInspectorGUI () {
        // 添加父级触发器(勾选后显示子内容、取消勾选后隐藏子内容)
        m_ShowExtraFields.target = EditorGUILayout.ToggleLeft ("Show extra fields", m_ShowExtraFields.target);
        
        if (EditorGUILayout.BeginFadeGroup (m_ShowExtraFields.faded)) {
            EditorGUILayout.TextField (new GUIContent ("test"), "");
        }
        EditorGUILayout.EndFadeGroup ();
    }
}
```
![BeginFadeGroup示例]()

### Toggle
构成一个类似Html的`lable + checkbox`组合, 且`label在左 checkbox 在右`.  
[Toggle官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.Toggle.html)

使用示例请按上诉`BeginFadeGroup方法`为例(只需要把`ToggleLeft`换成`Toggle`即可). 

![Toogle示例]()

### ToggleLeft
构成一个类似Html的`lable + checkbox`组合, 且`label在右 checkbox 在左`.  
[ToggleLeft官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.ToggleLeft.html)

### BeginFoldoutHeaderGroup & EndFoldoutHeaderGroup
将特定字符串作为折叠帘的标题, 通过点击折叠帘的标题来选择折叠/展示, 点击后返回参数如下. 
- false: 折叠
- true: 展开 

[BeginFoldoutHeaderGroup官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.BeginFoldoutHeaderGroup.html)

示例代码
```
public class PlayerEditor : Editor {
    bool showPosition = true;

    public override void OnInspectorGUI () {
        showPosition = EditorGUILayout.BeginFoldoutHeaderGroup (showPosition, "Test");

        if (showPosition)
            EditorGUILayout.TextField (new GUIContent ("test"), "");

        EditorGUILayout.EndFoldoutHeaderGroup ();
    }
}
```
![BeginFoldoutHeaderGroup示例效果]()


### Foldout
将特定字符串作为折叠帘的标题, 通过点击折叠帘的标题来选择折叠/展示, 点击后返回参数如下.  
- false: 折叠
- true: 展开

到目前一共遇到的有关折叠的已有3种,分别是:  
- BeginFadeGroup  
配合Toggle可实现淡入淡出效果, 上层元素是一个Checkbox。
- BeginFoldoutHeaderGroup  
上层元素是一个鼠标移上去会聚焦的Header。
- Foldout
上层元素是一个普通label。

`BeginFoldoutHeaderGroup`和`Foldout`差距不大.  

[Foldout官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.Foldout.html)

示例代码: 
```
public class PlayerEditor : Editor {
    bool showPosition = true;

    public override void OnInspectorGUI () {
        showPosition = EditorGUILayout.Foldout (showPosition, "Test");
        if (showPosition)
            EditorGUILayout.Vector3Field ("Position", Selection.activeTransform.position);

        showPosition = EditorGUILayout.BeginFoldoutHeaderGroup(showPosition, "Test1");
        if (showPosition)
            EditorGUILayout.Vector3Field ("Position", Selection.activeTransform.position);
    }
}
```

### BeginHorizontal & EndHorizontal
在Begin和End之间添加的Layout将以水平方式自动排列(非常像Bootstrap的一个概念).   
[BeginHorizontal官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.BeginHorizontal.html)

示例代码:
```
public class PlayerEditor : Editor {
    public override void OnInspectorGUI () {
        EditorGUILayout.BeginHorizontal("Button");
        GUILayout.Button("button1");
        GUILayout.Button("button2");
        GUILayout.Button("button3");
        EditorGUILayout.EndHorizontal();
    }
}
```
![BeginHorizontal示例效果]()

### BeginVertical & EndVertical
在Begin和End之间添加的Layout将以竖直方式自动排列(非常像Bootstrap的一个概念).   
[BeginVertical官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.BeginVertical.html)

### BoundsField & BoundsIntField
两个方法效果相同, 均为编辑一个边界类型的布局, 其中包含两个3D坐标参数(中心位置和轴向大小), 用于去标记一个范围/立方体形状.   
区别在于传递的参数是整型的还是浮点型的而已.  
更多扩展使用方式需要查看`Bounds`和`BoundsInt`类提供的方法.  

[BoundsField官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.BoundsField.html)  
[BoundsIntField官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.BoundsIntField.html)  

示例代码:
```
public class PlayerEditor : Editor {
    Bounds bounds;
    BoundsInt boundsInt;

    private void OnEnable() {
        bounds = new Bounds();
        boundsInt = new BoundsInt();
    }

    public override void OnInspectorGUI() {
        bounds = EditorGUILayout.BoundsField("test", bounds);
        boundsInt = EditorGUILayout.BoundsIntField("test1", boundsInt);
    }
}
```

### BeginScrollView & EndScrollView
构成类似于浏览器的滚动条功能, 横向和竖向都可以设置是否存在滚动条.   
[BeginScrollView官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.BeginScrollView.html)

示例代码:
```
public class PlayerEditor : Editor {
    Vector2 scrollPos;
    string t = "This is a string inside a Scroll view!";

    public override void OnInspectorGUI () {
        scrollPos = EditorGUILayout.BeginScrollView (scrollPos, GUILayout.Width (300), GUILayout.Height (60));
        GUILayout.Label (t);
        GUILayout.Label (t);
        GUILayout.Label (t);
        GUILayout.Label (t);
        GUILayout.Label (t);
        EditorGUILayout.EndScrollView ();
    }
}
```
![BeginScrollView效果]()

### BeginToggleGroup 和 EndToggleGroup
将一组`Toggle`组合起来, 通过勾选/取消勾选父级来判断子集能否被修改.  
[BeginToggleGroup官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.BeginToggleGroup.html)

示例代码: 
```
public class PlayerEditor : Editor {
    bool[] pos = new bool[3] { true, true, true };

    bool posGroupEnabled = true;

    public override void OnInspectorGUI () {
        posGroupEnabled = EditorGUILayout.BeginToggleGroup ("Align position", posGroupEnabled);
        pos[0] = EditorGUILayout.Toggle ("x", pos[0]);
        pos[1] = EditorGUILayout.Toggle ("y", pos[1]);
        pos[2] = EditorGUILayout.Toggle ("z", pos[2]);
        EditorGUILayout.EndToggleGroup ();
    }
}
```
![BeginToggleGroup效果]()

### ColorField
创建会创建一个颜色选择器.   
[ColorField官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.ColorField.html)  

示例代码:
```
public class PlayerEditor : Editor {
    Color matColor = Color.white;

    public override void OnInspectorGUI () {
        matColor = EditorGUILayout.ColorField ("New Color", matColor);
    }
}
```

### GradientField
创建一个颜色渐变器.   
[GradientField官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.GradientField.html)  

示例代码: 
```
public class PlayerEditor : Editor {
    Gradient gradient = new Gradient ();

    public override void OnInspectorGUI () {
        gradient = EditorGUILayout.GradientField ("Gradient", gradient);
    }
}
```

### CurveField
创建一个曲线编辑器.   
[CurveField官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.CurveField.html)  

示例代码:
```
public class PlayerEditor : Editor {
    AnimationCurve curveX = AnimationCurve.Linear(0, 0, 10, 10);

    public override void OnInspectorGUI() {
        curveX = EditorGUILayout.CurveField("Animation on X", curveX);
    }
}
```

### DoubleField & FloatField & IntField & TextField & DelayedDoubleField & DelayedFloatField & DelayedIntField & DelayedTextField
这些方法的效果都类似与html中的input输入框，只不过是数据类型不同.

Delayed方法与非Delayed方法的区别在于前者在修改完input框内的数值后需要按下回车键或者鼠标点击其他地方才会生效, 而后者在修改后即使生效.  

### DropdownButton
可以通过和`GenericMenu`配合来制定一个下拉按钮, 通过点击下拉按钮弹出下拉按钮窗口, 在窗口中可以选择绑定有回调函数的具体属性.  
点击下拉按钮后返回true.   

[DropdownButton官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.DropdownButton.html)

示例代码:
```
public class PlayerEditor : Editor {
    public override void OnInspectorGUI () {
        EditorGUILayout.BeginHorizontal();
        EditorGUILayout.LabelField(new GUIContent("test"));

        if (EditorGUILayout.DropdownButton (new GUIContent("Nothing"), FocusType.Keyboard)) {
            var alls = new string[4] {"Nothing", "A", "B", "C", "D" };
            GenericMenu menu = new GenericMenu ();
            foreach (var item in alls) {
                menu.AddItem (new GUIContent (item), false, null, item);
            }
            menu.ShowAsContext ();
        }
        EditorGUILayout.EndHorizontal();
    }
}
```

### GetControlRect
可以通过`GUILayout`配合来获取一块指定的矩形空间(可以用来充当空格)

[GetControlRect官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.GetControlRect.html)

示例代码: 
```
public class PlayerEditor : Editor {
    public override void OnInspectorGUI () {
        EditorGUILayout.GetControlRect (GUILayout.Height (200));
        EditorGUILayout.ColorField(Color.white);
    }
}
```

### Space
添加一行空行.  
[Space官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.Space.html)

示例代码: 
```
EditorGUILayout.Space();
```

### EnumFlagsField
将枚举类中的枚举以下拉按钮的形式展示, 通过选中下拉按钮中的内容返回对应的枚举(非单选也非多选, 我说不清这是什么, 自己试试看).  

可选的结果有
- 枚举类中定义的枚举
- Nothing
- Everything

[EnumFlagsField官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.EnumFlagsField.html)

示例代码:
```
public class PlayerEditor : Editor {
    enum ExampleFlagsEnum {
        None = 0, // Custom name for "Nothing" option
        A = 1 << 0,
    }

    ExampleFlagsEnum m_Flags;

    public override void OnInspectorGUI () {
        m_Flags = (ExampleFlagsEnum) EditorGUILayout.EnumFlagsField (m_Flags);
    }
}
```

### EnumPopup
将枚举类中的枚举以下拉按钮的形式展示, 通过选中下拉按钮中的内容返回对应的枚举(单选).    

可选的结果有
- 枚举类中定义的枚举

[EnumPopup官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.EnumPopup.html)

示例代码:
```
public class PlayerEditor : Editor {
    enum ExampleFlagsEnum {
        None = 0, // Custom name for "Nothing" option
        A = 1 << 0,
    }

    ExampleFlagsEnum m_Flags;

    public override void OnInspectorGUI () {
        m_Flags = (ExampleFlagsEnum) EditorGUILayout.EnumPopup (m_Flags);
    }
}
```

### Popup
将数组以下拉按钮的形式展示, 通过选中下拉按钮中的内容返回对应的数组下标(单选).  

可选的结果有
- 数组中所有参数

[Popup官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.Popup.html)

示例代码: 
```
public class PlayerEditor : Editor {
    public int index = 0;

    public string[] options = new string[] { "Cube", "Sphere", "Plane" };
    
    public override void OnInspectorGUI () {
        EditorGUILayout.BeginHorizontal();
        EditorGUILayout.PrefixLabel("popup");
        index = EditorGUILayout.Popup (index, options);
        EditorGUILayout.EndHorizontal();
    }
}
```

### HelpBox
该方法最终展示样式为信息提示框, 没特别含义.  
[HelpBox官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.HelpBox.html)

示例代码:
```
public class PlayerEditor : Editor {
    public override void OnInspectorGUI () {
        EditorGUILayout.HelpBox("this is a message example", MessageType.Info);
    }
}
```

### InspectorTitlebar
展示样式为嵌套式的Inspector.  
[InspectorTitlebar官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.InspectorTitlebar.html)

示例代码:
```
public class PlayerEditor : Editor {
    bool fold = true;
    Transform selectedTransform;

    public override void OnInspectorGUI () {
        fold = EditorGUILayout.InspectorTitlebar (fold, selectedTransform);
        if (fold) {
            selectedTransform.position = EditorGUILayout.Vector3Field ("Position", selectedTransform.position);
        }
    }
}
```

### ObjectField
展示样式与Html中的File标签相似, 选中一个Unity中的对象.    
[ObjectField官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.ObjectField.html)

示例代码: 
```
public class PlayerEditor : Editor {
    public Object source;

    public override void OnInspectorGUI () {
        source = EditorGUILayout.ObjectField(source, typeof(Object), true);
    }
}
```

### PrefixLabel
比`Lablefield`好使用点。  

`Lablefield`可以单独使用, 在`Inspector`中展示为一个不可选的字符串.   
`PrefixLabel`需要与其他Layout协同工作, 在`Inspector`中展示为一个可选的标题字符串, 选中后自动聚焦到后面的输入框.  

[PrefixLabel官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.PrefixLabel.html)

示例代码: 
```
public class PlayerEditor : Editor {
    static int ammo = 0;

    public override void OnInspectorGUI () {
        EditorGUILayout.BeginHorizontal();
        EditorGUILayout.PrefixLabel("Ammo");
        ammo = EditorGUILayout.IntField(ammo);
        EditorGUILayout.EndHorizontal();
    }
}
```

### SelectableLabel
可以被选中的标题，能够方便标题名称的复制.  
[SelectableLabel官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.SelectableLabel.html)

示例代码:
```
EditorGUILayout.SelectableLabel("popup");
```

### PropertyField
将代理的目标类中的属性字段以自定义的方式展示到`Inspector`中。

例如如下例子, Player脚本中存在int类型参数b, 默认清空下在`Inspector`中会显示label为b的输入框。   
但通过如下方式后, 可以自定义b参数的展示方式, 还能用于条件判断。  

[PropertyField官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.PropertyField.html)

示例代码: 
```
public class PlayerEditor : Editor {
    SerializedProperty b;

    void OnEnable () {
        b = serializedObject.FindProperty ("b");
    }

    public override void OnInspectorGUI () {
        serializedObject.Update();
        
        EditorGUILayout.PropertyField (b, new GUIContent ("Int Field"));

        serializedObject.ApplyModifiedProperties();
    }
}
```

### Slider & IntSlider
`Slider`是将float类型的参数使用滑块的方式来进行修改。
`IntSlider`是将int类型的参数使用滑块的方式来进行修改。  
[Slider官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.Slider.html)
[IntSlider官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.IntSlider.html)

示例代码:
```
public class PlayerEditor : Editor {
    int v;
    public override void OnInspectorGUI () {
        v = EditorGUILayout.IntSlider (v, 0, 10);
    }
}
```

### MinMaxSlider
可以在一个float数值范围内, 以滑块的方式选中一个`连续`的数值区间.  
[MinMaxSlider](https://docs.unity3d.com/ScriptReference/EditorGUILayout.MinMaxSlider.html)

示例代码: 
```
public class PlayerEditor : Editor {
    float minVal   = -10;
    float minLimit = -20;
    float maxVal   =  10;
    float maxLimit =  20;

    public override void OnInspectorGUI () {
        EditorGUILayout.MinMaxSlider(ref minVal, ref maxVal, minLimit, maxLimit);
    }
}
```


### RectField & RectIntField
用以在`Inspector`界面上指定`Rect`的属性集合(一共4个属性x、y、w、h)  

Rect属性说明:
以屏幕为标准, 以左上角作为原点.
x: 矩形距离左侧屏幕的距离.
y: 矩形距离顶部屏幕的距离.
w: 矩形的宽度
h: 矩形的高度

`RectField`和`RectIntField`的区别在于前者参数类型为float后者为int.  

[RectField官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.RectField.html)  
[RectIntField官方文档](https://docs.unity3d.com/ScriptReference/EditorGUILayout.RectIntField.html)

示例代码:
```
public class PlayerEditor : Editor {
    static Rect pos;

    public override void OnInspectorGUI () {
        pos =  EditorGUILayout.RectField("Internal input:", pos);
    }
}
```

### 剩下没举例的
- LabelField
- LayerField
- MaskField
- TagField
- Vector2Field
- Vector2IntField
- Vector3Field
- Vector3IntField
- Vector4Field
- PasswordField
- TextField
- TextArea

这些方法在有以上的寄出后看名字就能理解, 差不多都是左边是个label,右边要么是输入框,要么是下拉按钮