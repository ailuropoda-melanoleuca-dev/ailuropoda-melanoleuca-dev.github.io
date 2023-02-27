+++
draft = false
thumbnail = ""
tags = ["Open3D", "Python", "3D点群処理"]
categories = ["Study"]
date = "2023-02-27T11:47:53+09:00"
title = "Open3Dを用いた法線推定"
description = ""
+++

　今回もOpen3Dを使ったお遊び試行として、スタンフォードバニーの法線推定をしてみた。  

　法線は点群の位置合わせや、画像認識時の特徴量として重要なので、Open3Dにはもちろん標準実装されている。

# 1. 点群データの法線  
　点に法線とはこれ如何に？という感があるが、一般に点群データを作成するときは、物体表面の点群が取得される(測光が物体の中に入り込まない)。  

　なので、得られる点群は面(物体表面)に位置する。これなら法線を新たに定義できそう。具体的には、ある点から一定距離以内にある点群を近傍点群とし、その３次元座標の主成分分析をおこなう。結果の固有値が最小となる固有ベクトルが法線となる。 

　結局は３次元座標での分散が一番小さいものが法線でしょ、的な感じ。

# 2. メッシュデータの法線
　メッシュデータの方では、メッシュを構成する辺を2本選び、そのベクトル積を求めれば、それはメッシュに対する法線である。


### 点群データ

　もちろん点群データはうさぴょん。
{{< img src="images/point_bunny.png" w="600" h="400" >}}

　kD-Treeで近傍点群を求めたのち、それを表面だと思い、PCAから最小固有ベクトルを算出。法線の大きさが全て一定になっているのは、正規化しているため。

　うさぴょんに針が刺さっているのがわかると思う。これが法線
{{< img src="images/point_normal_bunny.png" w="600" h="400" >}}

(えっ...コワー...)
### 法線データ

入力のメッシュデータなうさぴょん。
{{< img src="images/mesh_bunny.png" w="600" h="400" >}}

法線情報をメッシュデータから計算し、メッシュデータに付与することで、法線に合わせたライティングが追加されたビューができる。

{{< img src="images/mesh_bunny_lighting.png" w="600" h="400" >}}

(これはこれで生々しくて気持ち悪い)

```python
import sys
import open3d as o3d 
import numpy as np 

filename = sys.argv[1] 

# メッシュデータとして読み込み
mesh_bunny = o3d.io.read_triangle_mesh(filename)
o3d.visualization.draw_geometries([mesh_bunny])

# メッシュデータの法線推定
mesh_bunny.compute_vertex_normals()

# 法線情報が付与されているので、それに合わせたライティングがされる
o3d.visualization.draw_geometries([mesh_bunny])

# ポイントクラウドデータ
pcd_bunny =  o3d.geometry.PointCloud()
pcd_bunny.points = mesh_bunny.vertices
o3d.visualization.draw_geometries([pcd_bunny])

# 近傍点群はkD-Treeを用いて探索
# radius:探索半径
# max_nn:探索個数 
pcd_bunny.estimate_normals(search_param = o3d.geometry.KDTreeSearchParamHybrid(radius=10.0,\
                                                                               max_nn=10))

# 法線情報が追加されているので、描画フラグをtrueにする
o3d.visualization.draw_geometries([pcd_bunny], point_show_normal= True)

```

