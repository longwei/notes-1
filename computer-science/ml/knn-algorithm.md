# kNN algorithm


wp_id: 478
Status: publish
Date: 2017-05-29 15:22:00
Modified: 2017-05-29 15:22:00


kNN是一个有监督的分类算法，目的是把给定的数据的类型确定出来。具体做法是通过把给定数据与已知训练集中的数据之间计算距离（欧氏距离，曼哈顿距离。。。）来找出距离给定数据 X 最近的 k 个数据，然后取这 k 个数据中出现最多的分类作为 X 的分类，如果给定数据的维度是两维的话，在平面笛卡尔坐标系里表示出来就是这样的