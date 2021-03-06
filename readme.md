# 解题思路

## 特征提取算法:
### LDA

	照抄neuronblack同学的算法,使用LDA降维的确效果很不错 https://github.com/neuronblack/yiguan
	但是该同学仅仅依赖于app的安装与否来进行的LDA判断,我对他从使用角度进行了扩展,比如APP的点击数,APP的使用时长.

### word2vec

	使用gensim计算每个app的向量, 参数用这个向量训练CNN和LSTM,效果不好,放弃了.但是在后面的APP分类中使用效果不错
	
### KNN/KMeans

	主办方给的数据2/3没有APP分类. 
	最初我是用Kmeans,忽略主办方给的分类信息,对APP进行分类,发现不管分多少类,用XGB来训练都没什么提高.
	后来用KNN来补上APP分类信息的缺失,基于之前的word2vec来计算距离.


### TFIDF
	
	在3个维度计算了TFIDF, 一个是 APP, APP分类##1, APP分类##2, 每个维度分别在2个方向计算TFIDF: app打开的次数, APP使用时间.TFIDF维度依旧比较高,后面会继续对齐进行降维.

### SVD
	
	之前使用了word2vec来对APP做降维处理, 但是word2vec是依赖于app出现的上下文顺序,这个似乎和我们当前的场景不太合适.所以同时选择了用SVD来对上面计算所得的TFIDF做降维.


## 简单特征:
	
### 24小时分片统计APP
	
	一天24小时,每个APP使用占对应Device使用的百分比, 分别对点击次数和时长计算百分比
	
### 按照星期来统计APP
    
    一周7天,每天的APP使用分布情况
    工作日和周末APP使用情况分布及对比	

### 删除低频APP
	
	删除低频APP后做统计, 这个有点微弱的效果



## 模型
### XGB

- 单独对年龄,性别预测. 然后作为DNN模型的输入来参与预测. 
- 不能用年龄和性别做概率运算,这样得到的结果特别差.有人分析过,应该是在这些数据中年龄和性别并不是独立的2个维度.
- 这个模型做为基模型对性别年龄来预测,效果是比较好的. 

### LGB

作为一个树模型的补充,不知道为什么一直调优总是比xgb差一些. 和DNN的结果比较接近

### DNN

单独使用手工提取出来的维度,参与Training也能得到不错的成绩. 但是把树模型Training的结果合并在一起作为输入,分数有比较大的提高.

### CNN/LSTM

都尝试过,效果不太好放弃了

### 模型融合

做的比较简单,就是把之前模型中每一个跳出比较好的一个,使用DNN来融合.


## 文件结构
 
    code_felix/
    ├── merge
    │   ├── dnn_merge.py  :使用DNN对各个模型输出的h5文件做融合
    │   ├── file_merge.py :直接对输出结果做简单的加权融合 
    │   └── utils.py
    ├── model
    │   ├── dnn.py
    │   ├── lgb_raw.py
    │   ├── search
    │   │   ├── lgb.py : 对LGB模型参数做随机搜索
    │   │   └── xgb.py :对XGB模型参数做随机搜索
    │   ├── sex
    │   │   ├── dnn_sex.py :使用DNN模型对sex 和age很不预测
    │   │   ├── lgb_age.py :使用LGB 对age做单独预测
    │   │   ├── lgb_sex.py :使用LGB 对sex做单独预测
    │   │   ├── xgb_age.py :使用XGB 对age做单独预测
    │   │   └── xgb_sex.py :使用XGB 对age做单独预测
    │   └── xgb.py
    ├── tiny
    │   ├── feature_filter.py :随机的过滤一些不重要的维度
    │   ├── group_label.py    :对APP分组做各种统计
    │   ├── knn.py            :使用KNN算法补齐APP缺失的分组  
    │   ├── lda.py            :使用LDA算法对APP和分组做降维
    │   ├── package.py        :对APP的信息做各种汇总
    │   ├── tfidf.py          :对APP和分组计算TFIDF
    │   ├── usage.py          :对APP的使用做汇总
    │   ├── util.py
    │   └── word2vec.py       :针对APP的使用信息, 对APP向量化
    └── utils_
        ├── util_cache_file.py
        ├── util_date.py
        ├── util_log.py
        └── util_pandas.py
