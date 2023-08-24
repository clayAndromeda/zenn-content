---
title: "Unityのカスタムエディタを使って[SerializeField]の値が変更されたことを検知する方法"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["unity"]
published: true
---

UnityのMonoBehaviour継承コンポーネントを作っていたとき、`[SerializeField]`の値が変更された時に特定の処理を通したいということがありました。

通常この手の処理を組むときはMonoBehaviour.OnValidate()を使うと思います。しかし、その時組んでいた処理は、何らかの理由でOnValudate()中で呼び出せませんでした。

またOnValidate()は、ややこしい挙動をするため使いこなすのは難しいです。

https://someiyoshino.info/entry/2022/08/12/201247

そこで今回はカスタムエディタ機能を使ってOnValidate()でやりたかった「値が変更されたことを検知して処理を通す」方法を実装してみました。

## カスタムエディタとは

カスタムエディタとは、コンポーネントのインスペクター上での見た目を変えることができる機能です。

https://docs.unity3d.com/ja/2022.3/Manual/editor-CustomEditors.html

## 実装

ここでは、以下に示すコンポーネントの値が変更されたら特定の処理を通すことを目標に実装します。

```cs
using UnityEngine;

public class SampleComponent : MonoBehaviour
{
    [SerializeField] private int value;

    // valueが変更された時にこのメソッドを呼び出したい
    public void Rebuild()
    {
        Debug.Log("Rebuild");
    }
}
```

カスタムエディタは以下のように実装します。

```cs
using UnityEditor;
using UnityEngine;

[CustomEditor(typeof(SampleComponent))]
public class SampleComponentEditor : Editor
{

    public override void OnInspectorGUI()
    {
        EditorGUI.BeginChangeCheck();

        // 通常通りインスペクタ描画する
        base.OnInspectorGUI();

        if (EditorGUI.EndChangeCheck())
        {
            // インスペクタで値が変更された
            if (target is SampleComponent component)
            {
                component.Rebuild();
            }
        }
    }
}
```

## 解説

カスタムエディタでは、base.OnInspectorGUI()を呼び出すことで、通常通りインスペクタ描画することができます。

この前後にEditorGUI.BeginChangeCheck()とEditorGUI.EndChangeCheck()を挟むと、インスペクタで値が変更されたかを調べることができます。

```cs
    public override void OnInspectorGUI()
    {
        EditorGUI.BeginChangeCheck();

        // 通常通りインスペクタ描画する
        base.OnInspectorGUI();

        if (EditorGUI.EndChangeCheck())
        {
            // インスペクタで値が変更された
            if (target is SampleComponent component)
            {
                component.Rebuild();
            }
        }
    }
```

## おまけ

もしインスペクタ上で値が変更された時呼び出す処理をPlay Mode中は無視したい時は、 `EditorApplication.isPlayingOrWillChangePlayMode`が使えます。

```cs
    public override void OnInspectorGUI()
    {
        EditorGUI.BeginChangeCheck();

        // 通常通りインスペクタ描画する
        base.OnInspectorGUI();

        if (EditorGUI.EndChangeCheck())
        {
            if (EditorApplication.isPlayingOrWillChangePlayMode)
            {
                return;
            }

            // インスペクタで値が変更された
            if (target is SampleComponent component)
            {
                component.Rebuild();
            }
        }
    }
```

## まとめ

カスタムエディタ機能を使ってインスペクタ上での値の変更を検知する方法を紹介しました。

OnValidate()では不足、あるいは挙動がわかりにくいので別の方法を使いたいという時はこちらを参考にしてください。

