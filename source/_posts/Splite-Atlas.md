---
title: Splite Atlas

date: 2017-05-26 16:21:41

categories: "Unity�ű�"
tags:
       - Unity
description:��NGUI�����ͼ�����·ָ��Сͼ
---


  ֻ��NGUI��ͼ�������Ҫʹ������ĳ��Сͼ��ô�죿������������������������
  NGUIͼ������󴴽��� Ԥ�� ����ͼ�����ʡ��ı����ı��м�¼����Сͼ����ͼ�е�λ����Ϣ��NGUI��ͼ�����Ǹ����ı��е���Ϣȡ��Сͼ�ġ�������Ҫ�õ����е���ͼ���ı���
 
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
	  //��Ϊ�ı�����Ϣ�����ݽṹ
      {
	
        public string name��//����, ��spriteName
        public Rect frame; // ����ͼ�е�λ��

        public bool rotated;//�Ƿ�����ת
        public bool trimmed;//
        public Rect spriteSourceSize;//

        public Vector2 sourceSize; //ԭ�гߴ�
    }
  
   [MenuItem("Splite/splite")]
   
   public static void Splite()
    {
        TextAsset asset = Selection.activeObject as TextAsset;  //��ȡ�ı�
        //�ı�����ͼͬ���� �����ı�·��������ͼ
        string tpath = AssetDatabase.GetAssetPath(asset);
        tpath = Path.GetDirectoryName(tpath) + Path.AltDirectorySeparatorChar + Path.GetFileNameWithoutExtension(tpath) + ".png";
        Texture2D t2d = AssetDatabase.LoadAssetAtPath(tpath, typeof(Texture2D)) as Texture2D;
        //���ı����н�������ȡСͼ����Ϣ
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
        //���ļ�ѡ�񴰿ڣ�ѡ��Сͼ�洢���ļ���
        string path = EditorUtility.OpenFolderPanel("Find the file to save", "", "");
       //����Сͼ����Ϣ������Сͼ���洢��ָ�����ļ���
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