# 智算之道——2020人工智能应用挑战赛

![](https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/cover/csia_ltc_2020.png)
表格类数据挖掘+图像分类赛题，赛程极短。初赛阶段出现了标签leak。个人赛道初赛Rank4，决赛Rank13。
<!--more-->
## 空值填充
　　以众数填充为例。
```Python
for column in list(all_data.columns[all_data.isnull().sum() > 0]):
  mode_val = all_data[column].mode()[0]
  all_data[column].fillna(mode_val, inplace=True)
```
　　注意可能有多个众数，一般取第一个（mode()[0]），否则填充后的DataFrame仍存在空值。

## focal loss ——tensorflow实现
```Python
def focal_loss(logits, labels, gamma=2):
    softmax = tf.reshape(tf.nn.softmax(logits), [-1])
    labels = tf.range(0, tf.shape(logits)[0]) * tf.shape(logits)[1] + labels
    prob = tf.gather(softmax, labels)
    weight = tf.pow(tf.subtract(1., prob), gamma)
    loss = -tf.reduce_mean(tf.multiply(weight, tf.log(prob)))
    return loss
```

## 初赛解决方案
　　原始特征 + 交叉特征 + 整体(train+test)空值填充，lgb 5折 + catboost，Averaging加权0.4 / 0.6，线上auc 0.8581，个人rank4，初赛出现了标签leak，与前排差距较大，达到了2个百。  

## Function call stack: train_function
　　在tensorflow2.x下调用tf.combat1没有关闭eager_execution()，加入如下代码解决：
  ```Python
  tf.compat.v1.disable_eager_execution()
  ```

## fit_generator
　　从tensorflow 2.1.0开始已不推荐使用fit_generator，fit替代之。
```Python
# generator
def generator(batch_size):
    j = 1
    while True:
        x_train = np.zeros((batch_size, 128, 128, 3))
        for i in range(batch_size):
            img_path= './'
            img = cv2.imread(img_path)
            img = cv2.resize(img,(128,128))
            x_train[i] = img
        labels = y_train[(j-1)*batch_size:j*batch_size]
        j=j+1
        yield x_train, labels

# fit_generator
fit_generator(self, 
			generator, 
            steps_per_epoch,
            epochs=1, 
            verbose=1, 
            callbacks=None, 
            validation_data=None, 
            validation_steps=None, 
            class_weight=None, 
            max_q_size=10, 
            workers=1, 
            pickle_safe=False, 
            initial_epoch=0
            )
```
## 决赛解决方案
　　基线模型为EfficientNET-b5，使用20%的数据预热后，冻结除Dense外的所有层并使用全量数据调整网络。[Fixing the train-test resolution discrepancy](http://papers.nips.cc/paper/9035-fixing-the-train-test-resolution-discrepancy)。    
 　　调用模型对测试集分类结果进行推断时，对测试集做了TTA，包括随机裁切、左右翻转等（考虑天气图像的特殊性，未做上下翻转）。[Test-Time Augmentation (TTA) Tutorial](https://github.com/ultralytics/yolov5/issues/303)
   ![](https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/%E6%99%BA%E7%AE%97%E4%B9%8B%E9%81%932020/scheme.png)
