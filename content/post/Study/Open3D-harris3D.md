+++
draft = false
thumbnail = ""
tags = ["Open3D" , "3D点群処理", "Python"]
categories = ["Study"]
date = "2023-03-02T15:55:42+09:00"
title = "Open3DでHarris特徴量を計算する"
description = ""
+++

# Harris-3D

二次元画像処理ではHarris特徴量の計算は割とよくやる。
### Harrisのコーナー検出
輝度値や明度値にした画像にフィルターを縦横に移動させながら、その値の変化を見る。
- 縦横にも変化が大きいのはコーナー
- 縦、あるいは横のみに変化が大きいのはエッジ
- 変化が大きくないのは特徴的ではないところ

ちなみに以下がわかりやすい
- [Harrisのコーナー検出を魂で理解する](https://campkougaku.com/2021/09/22/harris)
- A Combined Corner and Edge Detector(原論文)

フィルターは長方形でなくてもよく、円形だったりする場合もある(使ったことないけど)。

### Harris-3D
ここでは、点群の法線の変化をみることで、特徴点を判定している。

変化がある一定以上であれば、とりあえず全てメモしておき、最後に、周りの特徴点をチェックし、その中で一番、基準値$R$

$$R = \frac{det(M)}{tr(M)} $$

が大きいものを選ぶことで、同じ特徴点付近にいくつもの点が選ばれるのを防ぐ(Non Maximimum Supuressionというらしい)。

### やってみた

黄色いぴょんに赤い点が付いているのが今回計算したハリス特徴量

鼻とか、足周りの曲がってるところとか、耳元とか、いろいろ赤くなっているのがわかると思う

![代替テキスト](/images/harris.png)

```python
import sys 
import open3d as o3d
import numpy as np 

# 指定した点の色を変更し、強調する
def keypoints_to_spheres(keypoints, radius):
    spheres = o3d.geometry.TriangleMesh()
    for keypoint in keypoints.points:
        sphere = o3d.geometry.TriangleMesh.create_sphere(radius=radius)
        sphere.translate(keypoint)
        spheres += sphere
    spheres.paint_uniform_color([1.0, 0.0, 0.0])
    return spheres

def nms(pcd , harris, is_active, inds, max_nn):
    # Kd-Treeで近傍点探索
    pcd_tree = o3d.geometry.KDTreeFlann(pcd)

    # 特徴点の周りmax_nn個の頂点を確認し、一番評価が高いもののみ残す
    for i in range(len(np.asarray(pcd.points))):
        if is_active[i]:
            [num_nn, inds, _] = pcd_tree.search_knn_vector_3d(pcd.points[i], max_nn)
            inds.pop(harris[inds].argmax() )
            is_active[inds] = False

    return is_active

# harris3d - harrisのコーナー検出の3D版
def harris3d(pcd , radius = 0.01, max_nn = 100, threshold = 0.001):
    # Kd-Treeで法線計算
    pcd.estimate_normals(search_param = o3d.geometry.KDTreeSearchParamHybrid(radius = radius, max_nn = max_nn))
    # Kd-Treeで近傍点探索
    pcd_tree = o3d.geometry.KDTreeFlann(pcd)

    # 特徴点の候補
    harris = np.zeros(len(np.asarray(pcd.points)) )
    # NMSの結果を保存
    is_active = np.zeros(len(np.asarray(pcd.points)), dtype = bool)

    for i in range(len(np.asarray(pcd.points))):
        # 関数search_knn_vector_3dは、アンカーポイントの最近傍点のインデックスのリストを返す
        # num_nn は近傍点の個数, idxsはインデックス
        [num_nn, inds, _] = pcd_tree.search_knn_vector_3d(pcd.points[i], max_nn)
        # indsだけ抜き出し
        pcd_normals = pcd.select_by_index(inds)

        pcd_normals.points = pcd_normals.normals

        # 平均と共分散行列
        [_,covar] = pcd_normals.compute_mean_and_covariance()

        # det/trace
        harris[i] = np.linalg.det(covar)/np.trace(covar)

        # det/traceが基準を超えているか？
        if (harris[i] > threshold):
            is_active[i] = True 

    # Non maximum Suppression
    is_active = nms(pcd ,harris, is_active, inds, max_nn)

    # 最終的な特徴点をselect by indexで生成
    keypoints = pcd.select_by_index(np.where(is_active)[0])
    
    return keypoints


filename = sys.argv[1]
pcd = o3d.io.read_point_cloud(filename)

keypoints = harris3d(pcd, 1000, 100, 0.0001) 

pcd.paint_uniform_color([1.0,1.0,0])
radius = 0.001

o3d.visualization.draw_geometries([keypoints_to_spheres(keypoints, radius) , pcd])
```