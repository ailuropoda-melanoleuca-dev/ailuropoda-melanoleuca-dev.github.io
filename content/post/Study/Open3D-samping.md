+++
draft = false
thumbnail = ""
tags = ["Open3D", "Python", "3D点群処理"]
categories = []
date = "2023-02-23T20:12:26+09:00"
title = "Open3Dのサンプリング"
description = ""
+++

Open3Dを使っていくつかのサンプリング手法を試してみた。  

- Voxel Down Sampling
- Random Sampling
- Uniform Down Sampling
- Farthest Point Sampling
- Poisson Disk Sampling  

使った点群データは[スタンフォードバニー](http://graphics.stanford.edu/data/3Dscanrep/)ちゃん


![フクロウa](/stanford_bunny.png)


{{< img src="images/tn.png" w="600" h="400" >}}


## Voxel Down Sampling

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

## Random Sampling
## Uniform Down Sampling
## Farthest Point Sampling
## Poisson Disk Sampling  



Open3Dめちゃくちゃ便利、C++の方も今度使ってみようかな。

