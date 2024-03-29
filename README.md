# LuaCreator -- EamUnityEditorTool
*Quickly creating lua files in unity with a templete file.*

CSDN博客地址：[点击这里](https://blog.csdn.net/qq_33337811/article/details/78379057)


----------


Unity里能创建 c#脚本模板，但是如果我想创建Lua脚本模板怎么办呢？拓展一下编辑器吧。

先准备一个lua脚本模版文件，位置例如在：Assets/Editor/Lua/Template/lua.lua

我们先写上一行代码：print("#NAME#")，下面我们会使用正则表达式用新建的lua脚本文件名替换NAME字段。

C#代码：

    using UnityEngine;
	using UnityEditor;
	using System;
	using System.IO;
	using System.Text;
	using UnityEditor.ProjectWindowCallback;
	using System.Text.RegularExpressions;
	 
	public class Test
	{
    [MenuItem("Assets/Create/Lua Script", false, 80)]
    public static void CreatNewLua()
    {
        ProjectWindowUtil.StartNameEditingIfProjectWindowExists(0,
        ScriptableObject.CreateInstance<MyDoCreateScriptAsset>(),
        GetSelectedPathOrFallback() + "/New Lua.lua",
        null,
       "Assets/Editor/Lua/Template/lua.lua");
    }
 
 
 
    public static string GetSelectedPathOrFallback()
    {
        string path = "Assets";
        foreach (UnityEngine.Object obj in Selection.GetFiltered(typeof(UnityEngine.Object), SelectionMode.Assets))
        {
            path = AssetDatabase.GetAssetPath(obj);
            if (!string.IsNullOrEmpty(path) && File.Exists(path))
            {
                path = Path.GetDirectoryName(path);
                break;
            }
        }
        return path;
    }
	}    
 
 
	class MyDoCreateScriptAsset : EndNameEditAction
	{
 
 
    public override void Action(int instanceId, string pathName, string resourceFile)
    {
        UnityEngine.Object o = CreateScriptAssetFromTemplate(pathName, resourceFile);
        ProjectWindowUtil.ShowCreatedAsset(o);
    }
 
    internal static UnityEngine.Object CreateScriptAssetFromTemplate(string pathName, string resourceFile)
    {
        string fullPath = Path.GetFullPath(pathName);
        StreamReader streamReader = new StreamReader(resourceFile);
        string text = streamReader.ReadToEnd();
        streamReader.Close();
        string fileNameWithoutExtension = Path.GetFileNameWithoutExtension(pathName);
        text = Regex.Replace(text, "#NAME#", fileNameWithoutExtension);
        //string text2 = Regex.Replace(fileNameWithoutExtension, " ", string.Empty);
        //text = Regex.Replace(text, "#SCRIPTNAME#", text2);
        //if (char.IsUpper(text2, 0))
        //{
        //    text2 = char.ToLower(text2[0]) + text2.Substring(1);
        //    text = Regex.Replace(text, "#SCRIPTNAME_LOWER#", text2);
        //}
        //else
        //{
        //    text2 = "my" + char.ToUpper(text2[0]) + text2.Substring(1);
        //    text = Regex.Replace(text, "#SCRIPTNAME_LOWER#", text2);
        //}
        bool encoderShouldEmitUTF8Identifier = true;
        bool throwOnInvalidBytes = false;
        UTF8Encoding encoding = new UTF8Encoding(encoderShouldEmitUTF8Identifier, throwOnInvalidBytes);
        bool append = false;
        StreamWriter streamWriter = new StreamWriter(fullPath, append, encoding);
        streamWriter.Write(text);
        streamWriter.Close();
        AssetDatabase.ImportAsset(pathName);
        return AssetDatabase.LoadAssetAtPath(pathName, typeof(UnityEngine.Object));
    }
 	}    



因为是模板，就可以添加一些预制代码在上面， 当填写完类名后可以来替换操作，  例如unity自带的 创建C#文件， 外面写了类名里面也跟着变。

在Project面板右键新建：

![](https://i.imgur.com/7FyROuG.png)

然后可以输入Lua脚本的类名。

然后就根据自己创建Lua的文件名 正则替换模板里的东西了：

![](https://i.imgur.com/agN6cX6.png)

C#里的代码可以自行扩展，多写几个菜单，使用不同的模版创建不同用户的lua脚本，如某界面的UI脚本，某一块的数据管理脚本。

----------


**扩展：**

1/ 如也可以在63行左右位置加上text = Regex.Replace(text, "#TIME#", System.DateTime.Today.ToShortDateString());把当前年月日也替换到Lua代码里，也可以根据自己的项目的习惯，谢一些模版内容，按需要替换哈。

2/ 可以把ProjectWindowUtil.StartNameEditingIfProjectWindowExists单独封装为一个方法，传入string参数作模版路径，然后根据需要谢不同的MenuItem菜单去调用使用不同的模版创建文件。