# CCF-BDCI 视频拷贝检测

## 当前思路
1. 提取视频关键帧；
2. 通过resnet18提取关键帧特征；
3. 对特征进行~~PCA降维~~（失败中）和L2正则化；
4. 所有视频两两计算得相似度矩阵；
5. 对于相似度top-K视频对，进行帧级匹配。

## TODO

1. 细粒度抽帧（当前1s抽一帧，感觉已经足够了）；
2. 代码重构（还差video_retrieval）；
3. 继续case analysis（不同视频，相同位置、角度与表情的大妈和男生的相似度竟然有85%，特征提取要继续研究）

## 数据说明

训练数据集分为3个部分：

* query文件夹，其中包括3000个视频，为侵权视频训练集，格式为mp4，文件名为视频id，例如：b394c1e0-afd9-11e9-a9d1-fa163ee49799.mp4,其中b394c1e0-afd9-11e9-a9d1-fa163ee49799为视频id，与文件train.csv中字段对应

* refer文件夹，其中包括200个视频，为版权长视频视频集，格式为mp4，文件名为视频id，例如，2528707200.mp4，2528707200表示视频id，与文件train.csv中字段对应

* train.csv文件，记录侵权视频和版权长视频对应的关系及具体匹配时间，其中每列有8个空格分隔，具体字段说明参见下表：

| 列名             | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| query_id         | 侵权视频id，对应query文件夹中视频名称                        |
| query_time_range | 侵权视频的具体侵权时间段，起止时间用逗号分隔，时间单位毫秒   |
| refer_id         | 侵权视频所侵权的版权视频id，对应refer文件夹中视频名称        |
| refer_time_range | 侵权视频所侵权的版权视频的具体时间段，起止时间用逗号分隔，时间单位毫秒 |

测试数据集分为2个部分：

• query文件夹，其中包括1500个视频，为侵权视频测试集，由训练集refer文件夹长视频生成，格式为mp4，文件名为视频id，例如：b394c1e0-afd9-11e9-a9d1-fa163ee49799.mp4,其中b394c1e0-afd9-11e9-a9d1-fa163ee49799为视频id，与文件evaluation_public.csv中字段对应

• evaluation_public.csv文件，测试集，没有标注结果，用于提交模型竞赛结果，具体字段说明如下：

| 列名     | 说明                                        |
| -------- | ------------------------------------------- |
| query_id | 可能侵权视频id，id对应query文件夹中视频名称 |

## 文件路径

video_copy_detection/  
├── code  
│   ├── CBVR  
│   ├── feature_extract.ipynb  
│   ├── key_frame_extract.ipynb  
│   ├── README.md  
│   └── video_retrieval.ipynb  
├── download  
│   ├── submit_example.csv  
│   ├── test_query.tar.gz  
│   ├── train.csv  
│   ├── train_query.tar.gz  
│   └── train_refer.tar.gz  
├── test  
│   ├── query  
│   ├── query_frame  
│   └── submit_example.csv  
├── train  
│   ├── query  
│   ├── query_frame  
│   ├── refer  
│   ├── refer_frame  
│   └── train.csv  
└── var  
    ├── refer_features.txt  
    └── test_query_features.txt  

其中`query`和`refer`为训练数据文件夹，而`query_frame`和`refer_frame`分别为提取的关键帧的存放文件夹。

## 评测标准

本模型依据提交的结果文件，采用F1-score进行评价。执行时间及特征索引大小将在复赛进行考察，初赛不进行相应限制和评分。

（1）针对每个待检测侵权视频，如果正确匹配侵权长视频ID，并且起止时间段匹配误差在5秒以内，认定为预测结果正确，用TP表示；错误匹配长视频ID或者起止时间段误差超过5秒，认定为预测结果错误，用FP表示；未进行预测数据及预测错误数据，用FN表示。

（2）通过第一步的统计值计算precision和recall，计算公式如下：

$$
precision= \frac{TP}{TP+FP}
$$	
 
$$
recall= \frac{TP}{TP+FN}
$$	

（3）通过第二步计算结果计算每个类别下的F1-score，计算方式如下：

$$
f1= \frac{2\*precision\*recall}{precision+recall}
$$
