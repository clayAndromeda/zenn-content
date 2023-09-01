---
title: "UnityのSplineパッケージを使ってプロシージャルな壁を作る方法"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["unity"]
published: true
---

Unity公式が提供しているSplineパッケージを使って、壁を生成するシステムを作ってみました。

![](/images/20230829/thumb.gif)

## UnityのSplineパッケージについて

Unity2022で、公式がスプラインツールを公開しました。
本パッケージには、スプライン作成に加えて、スプライン上にGameObjectを生成する `SplineInstantiate` や、チューブ上のメッシュを生成する `SplineExtrude` といったコンポーネントが含まれています。

@[card](https://light11.hatenadiary.com/entry/2022/11/17/194205)

@[card](https://youtu.be/5IrKqVnvP6M?si=37nk6MK27IFc_Hx_)

今回は `SplineExtrude` コンポーネントを応用して、スプラインの形状に沿った壁メッシュを生成する `SplineWall` コンポーネントを作成しました。

## SplineWallコンポーネントの実装

### 実現方法

今回は、スプライン曲線を一定の間隔で分割し得られた点と、それらの点を上方向に平行移動した点を使って四角形のポリゴンを作ることで壁を生成することにしました。

`Evaluate()`メソッドを利用すると、Spline上の任意の点の座標が得られるので、このメソッドを利用すれば良さそうです。

![](/images/20230829/02.png)

### ポリゴンの構築方法

壁は、上述の手法で作った細かい四角形を、それぞれ三角形ポリゴン2枚で構築するようにしています。

頂点インデックスは画像の緑文字・矢印で示す順番で指定していきます。

![](/images/20230829/01.png)

### SplineWallのコード全体

この方針でスプラインに沿って壁を生成するコンポーネント `SplineWall` を実装しました。

今回は、参考にした `SplineExtrude` 通り、Advanced Mesh APIを使ってメッシュを生成しています（といっても複雑なことはしていませんが……）。

Advanced Mesh APIについては、以下の記事などで詳しく知ることができます。

@[card](https://qiita.com/aa_debdeb/items/486456e8352070055fac)

```cs
using System;
using System.Runtime.InteropServices;
using Unity.Mathematics;
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Splines;

[StructLayout(LayoutKind.Sequential)]
struct VertexData
{
	public Vector3 Position { get; set; }
}

[RequireComponent(typeof(SplineContainer), typeof(MeshFilter))]
public class SplineWall : MonoBehaviour
{
	[SerializeField, Range(2, 100)] private int divided = 10;
	[SerializeField] private float height = 5.0f;

	private SplineContainer splineContainer;
	private Mesh mesh;
	private MeshFilter meshFilter;

	private void Reset()
	{
		PrepareComponents();
		Rebuild();
	}

	private void PrepareComponents()
	{
		TryGetComponent(out splineContainer);
		TryGetComponent(out meshFilter);
		mesh = new Mesh();
		meshFilter.sharedMesh = mesh;
	}

	public void Rebuild()
	{
		var spline = splineContainer.Spline;
		if (spline == null) return;

		mesh.Clear();
		var meshDataArray = Mesh.AllocateWritableMeshData(1);
		var meshData = meshDataArray[0];
		meshData.subMeshCount = 1;

		// 頂点数とインデックス数を計算する
		var vertexCount = 2 * (divided + 1);
		var indexCount = 6 * divided;

		// インデックスと頂点のフォーマットを指定する
		var indexFormat = IndexFormat.UInt32;
		meshData.SetIndexBufferParams(indexCount, indexFormat);
		meshData.SetVertexBufferParams(vertexCount, new VertexAttributeDescriptor[]
		{
			new(VertexAttribute.Position),
		});

		var vertices = meshData.GetVertexData<VertexData>();
		var indices = meshData.GetIndexData<UInt32>();

		for (int i = 0; i <= divided; ++i)
		{
			// 頂点座標を計算する
			spline.Evaluate((float)i / divided, out var position, out var direction, out var splineUp);
			var p0 = vertices[2 * i];
			var p1 = vertices[2 * i + 1];
			p0.Position = position;
			p1.Position = position + new float3(0, height, 0);
			vertices[2 * i] = p0;
			vertices[2 * i + 1] = p1;
		}

		for (int i = 0; i < divided; ++i)
		{
			indices[6 * i + 0] = (UInt32)(2 * i + 0);
			indices[6 * i + 1] = (UInt32)(2 * i + 1);
			indices[6 * i + 2] = (UInt32)(2 * i + 2);
			indices[6 * i + 3] = (UInt32)(2 * i + 1);
			indices[6 * i + 4] = (UInt32)(2 * i + 3);
			indices[6 * i + 5] = (UInt32)(2 * i + 2);
		}

		meshData.SetSubMesh(0, new SubMeshDescriptor(0, indexCount));

		Mesh.ApplyAndDisposeWritableMeshData(meshDataArray, mesh);
		mesh.RecalculateBounds();
	}
}
```

## SplineWallコンポーネントを使いやすくするCustomEditorを用意する

このままの実装では、 `SplineWall.Reset()` が呼ばれるタイミングでしか壁メッシュが作られません。

`SplineExtrude` の実装を参考に、 `SplineWall` コンポーネントに対応するCustomEditorクラスである `SplineWallEditor` を用意して、スプラインに変更があれば壁メッシュを作り直せるようにしてみました。

```cs
using UnityEditor;
using UnityEditor.Splines;
using UnityEngine.Splines;

[CustomEditor(typeof(SplineWall))]
public class SplineWallEditor : Editor
{
	void OnEnable()
	{
		Spline.Changed += OnSplineChanged;
		EditorSplineUtility.AfterSplineWasModified += OnSplineModified;
		SplineContainer.SplineAdded += OnContainerSplineSetModified;
		SplineContainer.SplineRemoved += OnContainerSplineSetModified;
	}

	void OnDisable()
	{
		Spline.Changed -= OnSplineChanged;
		EditorSplineUtility.AfterSplineWasModified -= OnSplineModified;
		SplineContainer.SplineAdded -= OnContainerSplineSetModified;
		SplineContainer.SplineRemoved -= OnContainerSplineSetModified;
	}

	public override void OnInspectorGUI()
	{
		EditorGUI.BeginChangeCheck();
		base.OnInspectorGUI();
		if (EditorGUI.EndChangeCheck())
		{
			if (target is SplineWall)
			{
				(target as SplineWall)?.Rebuild();
			}
		}
	}

	private void OnSplineChanged(Spline spline, int knotIndex, SplineModification modificationType)
	{
		OnSplineModified();
	}

	private void OnSplineModified(Spline spline)
	{
		OnSplineModified();
	}

	private void OnContainerSplineSetModified(SplineContainer container, int spline)
	{
		OnSplineModified();
	}

	// ReSharper disable Unity.PerformanceAnalysis
	private void OnSplineModified()
	{
		if (EditorApplication.isPlayingOrWillChangePlaymode)
		{
			// プレイモード中なら何もしない
			return;
		}

		// 本来は対象のSplineが編集されたときだけメッシュを再計算する方がいい
		if (target is SplineWall component)
		{
			component.Rebuild();
		}
	}
}
```

このCustomEditorを使ったエディタ上の変更を検知するテクニックは、以前別記事で紹介しておりますので、是非参考にしてみてください。

@[card](https://zenn.dev/clay_andromeda/articles/unity-custom-editor-change-check)

これで、冒頭で示したような、壁メッシュ作成機能が作れました！


![](/images/20230829/thumb.gif)

## まとめ

この記事で紹介した実装では、テクスチャをはることができず、また、壁に厚みもないので使いにくいと思います。ですが、Splineパッケージを拡張して何かしたいという方の参考になれば幸いです。
