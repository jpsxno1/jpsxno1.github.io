---
title: Splite Atlas

date: 2017-05-26 16:21:41

categories: "Unity脚本"
tags:
       - Unity
description:将NGUI打包的图集重新分割成小图
---


  只有NGUI的图集，如果要使用里面某个小图怎么办？下面就这个问题给出解决方法。
  NGUI图集打包后创建了 预设 、贴图、材质、文本。文本中记录的是小图在贴图中的位置信息。NGUI的图集就是根据文本中的信息取得小图的。我们需要用到其中的贴图和文本。
 
```bash
using UnityEngine;
using UnityEditor;
using System.Collections;
using System.IO;
using System.Collections.Generic;

public class AtlasSplite {

    static List<Sprite> sprites = new List<Sprite>();

   [System.Serializable]

    public class Sprite
	  //此为文本中信息的数据结构
      {
	
        public string name；//名称, 即spriteName
        public Rect frame; // 在贴图中的位置

        public bool rotated;//是否有旋转
        public bool trimmed;//
        public Rect spriteSourceSize;//

        public Vector2 sourceSize; //原有尺寸
    }
  
   [MenuItem("Splite/splite")]
   
   public static void Splite()
    {
        TextAsset asset = Selection.activeObject as TextAsset;  //获取文本
        //文本和贴图同名， 根据文本路径加载贴图
        string tpath = AssetDatabase.GetAssetPath(asset);
        tpath = Path.GetDirectoryName(tpath) + Path.AltDirectorySeparatorChar + Path.GetFileNameWithoutExtension(tpath) + ".png";
        Texture2D t2d = AssetDatabase.LoadAssetAtPath(tpath, typeof(Texture2D)) as Texture2D;
        //对文本进行解析，获取小图的信息
        try
        {
            string text = asset.text;
            IDictionary<string, object> dict = MiniJSONs.Json.Deserialize(text) as IDictionary<string, object>;
            dict = dict["frames"] as IDictionary<string, object>;
            Sprite sprite;
            foreach(var pair in dict)
            {
                sprite = new Sprite();
                sprite.name = pair.Key;
                IDictionary<string, object> map = pair.Value as IDictionary<string, object>;
                sprite.frame = ParseString.ParseToRect(map["frame"] as IDictionary<string, object>);
                sprite.rotated = (bool)map["rotated"];
                sprite.trimmed = System.Convert.ToBoolean(map["trimmed"]);
                sprite.spriteSourceSize = ParseString.ParseToRect(map["spriteSourceSize"] as IDictionary<string, object>);
                sprite.sourceSize = ParseString.ParseToVector2(map["sourceSize"] as IDictionary<string, object>);
                sprites.Add(sprite);
            }
        }
        catch(System.Exception e)
        {
            Debug.LogError("Get Info From text error." + e.Message);
        }
        //打开文件选择窗口，选择小图存储的文件夹
        string path = EditorUtility.OpenFolderPanel("Find the file to save", "", "");
       //根据小图的信息，创建小图并存储到指定的文件夹
        string ppath;
        float i = 0;
        foreach (var sprite in sprites)
        {  
            ppath = path + Path.AltDirectorySeparatorChar + sprite.name;
            i += 1;
            EditorUtility.DisplayCancelableProgressBar(path, ppath, i / sprites.Count);

            Texture2D tt = new Texture2D((int)sprite.frame.width, (int)sprite.frame.height, t2d.format, true);
            Debug.Log(sprite.frame);
            Color[] clrs = t2d.GetPixels((int)sprite.frame.x, t2d.height - (int)sprite.frame.y - (int)sprite.frame.height, (int)sprite.frame.width, (int)sprite.frame.height);

            tt.SetPixels(clrs);
            File.WriteAllBytes(ppath, tt.EncodeToPNG());
        }
        EditorUtility.ClearProgressBar();
    }

}


public static class ParseString
{
    public static Rect ParseToRect(IDictionary<string, object> str)
    {
        Rect rect;
         try
         {
             rect = new Rect((long)str["x"], (long)str["y"], (long)str["w"], (long)str["h"]);
         }
        catch(System.Exception e)
         {
            rect = new Rect();
        }
        finally{

         }
         return rect;
    }

    public static Vector2 ParseToVector2(IDictionary<string, object> str)
    {
        Vector2 vect;
        try
        {
            vect = new Vector2((long)str["w"], (long)str["h"]);

        }
        catch (System.Exception e)
        {
            vect = Vector2.zero;
        }
        finally
        {

        }
        return vect;
    }
}

```