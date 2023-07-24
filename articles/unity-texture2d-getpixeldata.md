---
title: "Texture2D.GetPixelData()で直接ピクセルデータを書き換えるとアセットも変更されてしまうことがある"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["unity"]
published: true
---

Unity 2020.1から `Texture2D.GetPixelData()`というAPIが追加されています。
このメソッドを使えば、CPU上にあるテクスチャ色情報を直接参照することができるため、`GetPixels()`と比較して、テクスチャ色情報を配列にコピーするコストを下げることができます。

しかし、このAPIを使ってアセットとしてインポートしたTexture2Dを直接書き換えてしまうと、アセットそのものも変更されてしまいます。

## 実例

先日秋葉原に行った時の写真をネガポジ変換するコードを書きました。

![](/images/20230724/01.png)

```cs
using UnityEngine;

namespace Clay.ImageProcessor
{
	public class NegativeConverter : MonoBehaviour
	{
		[SerializeField] private Texture2D inputTexture;

		public void Start()
		{
			var pixels = inputTexture.GetPixelData<Color32>(0);
			for (var i = 0; i < pixels.Length; i++)
			{
				var color = pixels[i];
				color.r = (byte)(255 - color.r);
				color.g = (byte)(255 - color.g);
				color.b = (byte)(255 - color.b);
				pixels[i] = color;
			}
			// 変更を適応する
			inputTexture.Apply();
		}
	}
}
```

このコードを実行すると……

![](/images/20230724/02.png)

なんと、元の画像アセットごとネガポジ変換されてしまいました。

元々このコードを書いたときは、実行中に（複製して渡されると思っていた）Texture2Dをネガポジ変換するだけで、アセット本体を書き換えるつもりはありませんでした。

## 改善方法

1. スクリプト内でTexture2DやRenderTextureとして複製し、それらが持つピクセルデータを書き換える
2. 一度書き換えてしまったテクスチャアセットは、右クリックメニューでReimportすれば元に戻る
	- これで直るのは、Unityが書き換えたのは「元の画像ファイルをエンジン内部で読み取って作ったデータ」であって、大元のJPGファイルは変更されていないからです。

## まとめ

このようにUnityのAPIには、不意にアセットそのものを破壊してしまうモノが多数あります。注意して開発を進めましょう。
