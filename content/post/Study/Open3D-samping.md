+++
draft = false
thumbnail = ""
tags = ["Open3D", "Python", "3D点群処理"]
categories = ["Study"]
date = "2023-02-23T20:12:26+09:00"
title = "Open3Dのサンプリング"
description = ""
+++

Open3Dを使っていくつかのサンプリング手法を試してみた。  

- Voxel Down Sampling
- Random Sampling
- Farthest Point Sampling
- Poisson Disk Sampling  

使った点群データは[スタンフォードバニー](http://graphics.stanford.edu/data/3Dscanrep/)ちゃん

{{< img src="images/stanford_bunny.png" w="600" h="400" >}}


## Voxel Down Sampling

ボクセル化は一辺がある定数の立方体の中に、一点だけ点があるように点群の再構築するサンプリング手法。  
Open3Dにはボクセル化用の組み込み関数があるので、それを使うだけ。  

内部で３次元のそれぞれの軸でmin,maxを計算した後、空間を切るので、点群が整列する。
ボクセル化は最終的にいくつの点で再構成されるのかを指定できないのが微妙なところ。
```python
import sys 
import open3d as o3d

# コマンドライン引数でplyとボクセルの１辺の長さを指定
filename = sys.argv[1] 
sampling_rate = float(sys.argv[2])

pcd = o3d.io.read_point_cloud(filename)

# ボクセル化の実行
downpcd = pcd.voxel_down_sample(voxel_size = sampling_rate) 

o3d.visualization.draw_geometries([pcd])
print(pcd)

o3d.visualization.draw_geometries([downpcd])
print(downpcd)
```

{{< img src="images/stanford_voxel_bunny.png" w="600" h="400" >}}

スタンフォードバニーを一辺の長さ0.003でボクセル化した。  
ポイント数は7834点、実行時間は0.004[ms]

## Random Sampling

ランダムサンプリング、これは比較のために自分で適当に実装した。  
listの中に既に選ばれた点があるかどうかを確認するのに、サンプリング数を$N$として、$\mathcal{O}(N)$かかるので、全体計算量$\mathcal{O}(N^2)$になる。遅い。  
もちろん不均一
```python
import sys
import open3d as o3d
import numpy as np

def random_sampling(pcd, sampling_count):
    now_count = 0 
    points = np.asarray(pcd.points)
    indices = []
    while now_count != sampling_count:
        idx = np.random.randint(len(points))
        if idx not in indices:
            indices.append(idx)
            now_count += 1
        
    down_pcd = pcd.select_by_index(indices)
    return down_pcd


filename = sys.argv[1] 
sampling_count = int(sys.argv[2])

pcd = o3d.io.read_point_cloud(filename)

downpcd = random_sampling(pcd , sampling_count)

o3d.visualization.draw_geometries([pcd])
print(pcd)

o3d.visualization.draw_geometries([downpcd])
print(downpcd)
```

{{< img src="images/stanford_random_bunny.png" w="600" h="400" >}}

ポイント数は7800点、実行時間0.5630021095275879[ms]


## Farthest Point Sampling
ユークリッド距離を距離関数としてFPSを実装した。  
計算量は全点群数を$M$、取得点群数を$N$として、$\mathcal{O}(NM)$ぐらい。ランダムサンプリングより遅い。  
ランダムサンプリングと比べて点群が等間隔になっているのがわかると思う。  
計算量、速度の改善はダイクストラやKd-treeを使えば改善される。
```python
import sys
import open3d as o3d 
import numpy as np 
 
def l2_norm(a,b):
    return ((a-b)**2).sum(axis = 1)

def fps(pcd, k , metrics=l2_norm):
    # 選ばれた点の集合
    indices = np.zeros(k,dtype=np.int32)

    points = np.asarray(pcd.points)

    # 初期点を乱数で生成
    indices[0] = np.random.randint(len(points))

    distances = np.zeros(points.shape[0], dtype=np.float32)

    farthest_point = points[indices[0]]
    # 入力されたメトリクスに従って距離計算
    min_distances = metrics(farthest_point,points)
    distances[:] = min_distances 

    for i in range(1,k):
        indices[i] = np.argmax(min_distances)
        farthest_point = points[indices[i]]
        distances[:] = metrics(farthest_point,points)
        min_distances = np.minimum(min_distances,distances[:])

    pcd = pcd.select_by_index(indices)
    return pcd 


filename = sys.argv[1]
sample_count = int(sys.argv[2])
pcd = o3d.io.read_point_cloud(filename)
downpcd = fps(pcd ,sample_count)
o3d.visualization.draw_geometries([downpcd])

```
{{< img src="images/stanford_fps_bunny.png" w="600" h="400" >}}
ポイント数は7800点、実行時間は7.579[ms]


## Poisson Disk Sampling  

FPSと比べて速い。当たり前だけどランダムサンプリングよりも一様に点を取得できている。  
PDSの処理内容はランダムサンプリングとあまり変わらないが、サンプリングされた点同士がある一定距離以下である場合に棄却するので、空間的に一様に近づく。
```python
import sys 
import open3d as o3d


filename = sys.argv[1]
sample_count = int(sys.argv[2])

mesh = o3d.io.read_triangle_mesh(filename)
downpcd = mesh.sample_points_poisson_disk(number_of_points = sample_count)

o3d.visualization.draw_geometries([mesh])
o3d.visualization.draw_geometries([downpcd])
```

{{< img src="images/stanford_pds_bunny.png" w="600" h="400" >}}
ポイント数は7800点、実行時間は2.397[ms]




---

Open3Dめちゃくちゃ便利、ほとんどコーディングなしで遊べる。  
C++の方も今度使ってみようかな。

