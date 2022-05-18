# 2020 DIGIX全球校园AI算法精英大赛

![](https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/cover/2020DIGIX.png)
CTR + 细粒度图像检索赛题，记录一下比赛过程中遇到的问题。

<!--more-->

# 赛道A
## 2020.7.16
　　7.20放赛题数据，先拿kaggle五年前的Click-Through Rate Prediction试水。
### 分块读取全部数据
  ```python
  loop = True
  chunkSize = 1000000
  chunks = []
  index = 0
  while loop:
      try:
          print(index)
          chunk = train_data.get_chunk(chunkSize)
          chunks.append(chunk)
          index += 1
    except StopIteration:
          loop = False
          print("Iteration is stopped.")
  for i in tqdm(chunks):
      train_data = pd.concat(chunks, ignore_index=True)
  ```
### 随机读取一定比例的数据
  ```python
  for chunk in pd.read_csv(train_data, chunksize=chunksize):
      chunks += 1
      train = pd.concat([train, chunk.sample(frac=.05, replace=False, random_state=42)], axis=0) 
      print('Processing Chunk ' + str(chunks))
  ```
### 条件筛选修改
  ```python
  df['column_d'].loc[df['column_c'] == 0] = 0
  # df['column_d'][df['column_c'] == 0] = 0
  ```
### lightgbm 梯度提升决策树
  ```python
 def kfold_lightgbm(train, test, features, target, seed=42, is_shuffle=True):
    train_pred = np.zeros((train.shape[0],))
    test_pred = np.zeros((test.shape[0],))
    n_splits = 5  
    
    fold = KFold(n_splits=n_splits, shuffle=is_shuffle, random_state=seed)
    kf_way = fold.split(train[features])

    params = {
        'learning_rate': 0.003,
        'boosting_type': 'gbdt',
        'objective': 'regression',
        'num_leaves': 36,
        'metric': 'mse',
        'feature_fraction': 0.6,
        'bagging_fraction': 0.7,
        'bagging_freq': 6,
        'seed': 42,
        'bagging_seed': 1,
        'feature_fraction_seed': 7,
        'min_data_in_leaf': 7,
        'nthread': 8,
        'verbose': 1,
    }
    fold_importance_df = pd.DataFrame()
    for n_fold, (train_idx, valid_idx) in enumerate(kf_way, start=1):
        train_x, train_y = train[features].iloc[train_idx], train[target].iloc[train_idx]
        valid_x, valid_y = train[features].iloc[valid_idx], train[target].iloc[valid_idx]

        n_train = lgb.Dataset(train_x, label=train_y)
        n_valid = lgb.Dataset(valid_x, label=valid_y)

        clf = lgb.train(
            params= params,
            train_set= n_train,
            num_boost_round= 10000,
            valid_sets= [n_valid],
            early_stopping_rounds= 150,
            verbose_eval= 100
        )
        train_pred[valid_idx] = clf.predict(valid_x, num_iteration=clf.best_iteration)
        test_pred += clf.predict(test[features], num_iteration=clf.best_iteration) / fold.n_splits

        fold_importance_df["Feature"] = features
        fold_importance_df["importance"] = clf.feature_importance(importance_type='gain')
        fold_importance_df["fold"] = n_splits

    test[TARGET] = test_pred
    return test[['id', TARGET]], fold_importance_df
  ```
  ### 特征编码
  ```python
  if cat_features:
      ce_oe = ce.OrdinalEncoder(cols=cat_features, handle_unknown='impute')
      ce_oe.fit(train)
      train = ce_oe.transform(train)
      test = ce_oe.transform(test)
  ```
## 2020.7.20
　　下午官网放了赛题数据，随机抽了5%的数据放进GBDT跑了一下，目测效果并不是很好，CTR标签分布很不均匀，训练集标签为1的样本大概只占到了3%。
### 结果出现负值
　　GBDT是加法模型，下一轮都是上一轮预测值和实际值的残差作为label继续拟合，将结果相加，最后可能会出现负值，特别是例如CTR场景下大部分标签都为0的场景下更容易出现这种情况。
## 2020.7.21
　　丢了几个缺失比较大的特征，对数据做了简单随机采样之后跑lgb5折交了一发，线上分数能到0.7，比预想中的要好，还有一定的提升空间，下一步打算从模型角度切入。
## 2020.7.22
### error:Only one class present in y_true
　　DeepFM训练过程报错：Only one class present in y_true. ROC AUC score is not defined in that case.  
　　定义的AUROC函数如下：
  ```Python
  def auroc(y_true, y_pred):
  	return tf.compat.v1.py_func(roc_auc_score, (y_true, y_pred), tf.double)
  ```
　　AUC（ROC 曲线下的面积）需要足够数量的任一类才能有意义，而CTR样本中本身就存在着非常严重的正负样本不平衡的问题。
　　目前解决方案如下：
  ```Python
  # AUC for a binary classifier
  def auc(y_true, y_pred):
      ptas = tf.stack([binary_PTA(y_true, y_pred, k) for k in np.linspace(0, 1, 1000)], axis=0)
      pfas = tf.stack([binary_PFA(y_true, y_pred, k) for k in np.linspace(0, 1, 1000)], axis=0)
      pfas = tf.concat([tf.ones((1,)), pfas], axis=0)
      binSizes = -(pfas[1:] - pfas[:-1])
      s = ptas * binSizes
      return K.sum(s, axis=0)

  # -----------------------------------------------------------------------------
  # PFA, prob false alert for binary classifier
  def binary_PFA(y_true, y_pred, threshold=K.variable(value=0.5)):
      y_pred = K.cast(y_pred >= threshold, 'float32')
      # N = total number of negative labels
      N = K.sum(1 - y_true)
      # FP = total number of false alerts, alerts from the negative class labels
      FP = K.sum(y_pred - y_pred * y_true)
      return FP / N

  # -----------------------------------------------------------------------------
  # P_TA prob true alerts for binary classifier
  def binary_PTA(y_true, y_pred, threshold=K.variable(value=0.5)):
      y_pred = K.cast(y_pred >= threshold, 'float32')
      # P = total number of positive labels
      P = K.sum(y_true)
      # TP = total number of correct alerts, alerts from the positive class labels
      TP = K.sum(y_pred * y_true)
      return TP / P
  ```
　　也可在整个训练过程完成后，在vaild_set上计算auc。
  ```Python
  pred_ans_val = model.predict(vaild_model_input, batch_size=512)
  print('val_auc', roc_auc_score(vaild[target].values, pred_ans_val))
  ```
## 2020.7.24
### DeepFM参数调整
　　* **Regression**<br>　　This implementation also supports regression task. To use DeepFM for regression, you can set loss_type as mse. Accordingly, you should use eval_metric for regression, e.g., mse or mae.*  
　　DeepFM中task参数调整为regression后，loss也需随之进行更改。
  ```Python
  model = DeepFM(
  	linear_feature_columns=linear_feature_columns,
  	dnn_feature_columns=dnn_feature_columns,
  	task='regression',
  	l2_reg_embedding=1e-5
    )
    
    model.compile(
    	'adam',
    	'mse',
    	metrics=['accuracy']
        )
  ```
## 2020.7.29
### onehot编码
　　OneHotEncoder 的输入为 2-D array，data[feat] 返回的 Series 为 1-D array。  
  ```Python
  for feat in cat_features:
  	ohe = OneHotEncoder()
  	data[feat] = ohe.fit_transform(data[[feat]])
  ```
　　将data[feat]改为data[[feat]]
## 2020.8.14
### Error:Input contains NaN
　　报错*ValueError: Input contains NaN, infinity or a value too large for dtype('float32').*  
  ```Python
  for index in tr_x.columns:
  	if(df[index].isna().T.any()):
    	print(index,df[index].isna())
  # ------
  print(df[df.isnull().T.any()])
  ```
　　若检查dataframe无空值后仍然报错。可尝试检查dataframe索引是否连续。
  ```Python
  df.reset_index(drop=True)
  ```
## 2020.8.15
### pandas apply设置进度条
　　apply速度较慢，可设置进度条实时显示处理进度。  
  ```Python
  import pandas as pd
  from tqdm import tqdm
  
  tqdm.pandas(desc='pandas bar')
  
  test['B'] = test.progress_apply(lambda x:func(x['A']), axis=1)

  ```

# 赛道B
## 2020.7.28
### Resnet50预训练权重文件
　　.h5文件已上传至百度网盘，链接放在此处。  
   　　[resnet50_weights_tf_dim_ordering_tf_kernels.h5](https://pan.baidu.com/s/1jTn1lI101BZfOoFys9tlOA)，提取码: pdcg  
      　　放在C://users//(yourusername)//.keras//models文件下。  
         　　另外，可以通过[该网站](https://d.serctl.com/)下载Github上的release内容。
### plt.imshow与cv2.imshow显示色差
　　使用plt.imshow和cv2.imshow对同一幅图显示时，可能会出现色差，这是由于opencv的接口为BGR，而matplotlib.pyplot接口使用的是RGB。
  ```Python
  img = cv2.cvtColor(cv2.imread(img_path), cv2.COLOR_BGR2RGB)
  
  plt.imshow(img)
  plt.show()
  ```
　　或通过以下方法也可实现：
  ```
  b,g,r = cv2.split(cv2.imread(img_path))
  img = cv2.merge([r,g,b])
  ```
## 2020.8.7
### 余弦相似度
　　余弦相似性通过测量两个向量的夹角的余弦值来度量它们之间的相似性。0度角的余弦值是1，而其他任何角度的余弦值都不大于1；并且其最小值是-1。从而两个向量之间的角度的余弦值确定两个向量是否大致指向相同的方向。两个向量有相同的指向时，余弦相似度的值为1；两个向量夹角为90°时，余弦相似度的值为0；两个向量指向完全相反的方向时，余弦相似度的值为-1。该结果仅与向量方向相关。余弦相似度通常用于正空间，因此给出的值为-1到1之间。  
　　![](https://wx2.sbimg.cn/2020/08/08/oJscK.png)
　　给定两个属性向量，A和B，其余弦相似性θ由点积和向量长度给出：
　　![](https://wx2.sbimg.cn/2020/08/08/oJLVT.png)
　　对于两个向量的**余弦距离**（余弦距离 = 1 - 余弦相似度）的基本计算，Python代码如下：
  ```Python
  def cosin_distance(vec_1, vec_2):
    dot_product = 0.0
    normA = 0.0
    normB = 0.0
    for a, b in zip(vec_1, vec_2):
        dot_product += a * b
        normA += a ** 2
        normB += b ** 2
    if normA == 0.0 or normB == 0.0:
        return None
    else:
        return dot_product / ((normA * normB) ** 0.5)
  ```
## 2020.8.8
### 大规模数据下使用faiss计算余弦相似度(待完善)
  ```Python
  d = 2048                           # dimension

  nb = gallery_features.shape[0]        # database size
  nq = query_features.shape[0]      # nb of queries

  xb = gallery_features.astype('float32')
  xq = query_features.astype('float32')


  nlist = 1000                      #聚类中心的个数
  k = 10      # topk搜索
  quantizer = faiss.IndexFlatL2(d)  # the other index
  index = faiss.IndexIVFFlat(quantizer, d, nlist, faiss.METRIC_L2)
       # here we specify METRIC_L2, by default it performs inner-product search
  assert not index.is_trained
  index.train(xb)
  assert index.is_trained
 
  index.add(xb)                  # add may be a bit slower as well
  D, I = index.search(xq, k)     # actual search
  index.nprobe = 10              # default nprobe is 1, try a few more
  D, I = index.search(xq, k)

  ```
　　此处参考[官方样例](https://github.com/facebookresearch/faiss/wiki/Getting-started)。
## 2020.8.11
### Keras添加网络结构报错
  ```Python
  model = Sequential()
  model.add(load_model('/mnt/resnet.model').get_output_at(0))
  ```
　　*TypeError: The added layer must be an instance of class Layer.*   
　　可能是混合使用了keras.Sequential()和tf.keras.Sequential()；Keras的layer中有input和output属性，错误地使用该部分的成员函数时也可能导致该问题。  
　　修改如下：
  ```Python
  model = Sequential()
  model.add(load_model('/mnt/resnet.model').get_layer(index=0))
  ```
