# 中移集成（雄安产业研究院）首届OneCity编程大赛

NLP赛题，政务表格文件分类。初赛Rank13，复赛Rank33（20+）。
首次接触NLP，本题上分重在数据的正确提取与预处理。
<!--more-->
## 赛题任务
　　选手需要建立模型，针对政务表格文件实现自动化分类。允许使用一些常见的开源预训练模型，如bert。  
　　数据智能分类按照行业领域，将政务数据分为以下20个基础大类，分别是：生态环境、资源能源、信息产业、医疗卫生、文化休闲、财税金融、经济管理、教育科技、交通运输、工业、农业畜牧业、政法监察、城乡建设、商业贸易、旅游服务、气象水文测绘地震地理、外交外事、文秘行政、民政社区、劳动人事。


## 初赛方案
　　参赛时赛事日程已过半，提交次数较少，仅考虑了一些简单的方案。  
　　初赛阶段，文件标题基本完整，首先仅使用标题进行训练，预训练模型使用了RoBERTa-wwm-ext，使用五折CV。  
　　读取文件content后，针对content单独训练bert，最后将title和content的raw_output按0.7、0.3加权平均，线上accuracy 0.985，Rank 13。

## 决赛方案
　　舍弃初赛方案——Bert训练标题+TextCNN训练Content，直接使用TextCNN训练标题+Content拼接后的内容，并且最终acc指标受文本长度影响较大。  
　　训练前做了一定量的数据预处理工作，包括关键字提取、地理位置提取等，其中省市提取使用了[cpca](https://github.com/DQinYuan/chinese_province_city_area_mapper)。  
　　由于花了大量的时间用于文件读取上，导致预处理不够深入，而预处理也是本题的一个关键上分点，此外还可以对网络增加例如文件长度等特征输入。复赛线上rank33，后补位至20+。

## 完整代码
　　仅供参考。
```Python
#!/usr/bin/env python
# coding: utf-8

# In[1]:


import warnings
warnings.simplefilter('ignore')
get_ipython().run_line_magic('matplotlib', 'inline')
import matplotlib.pyplot as plt
#!pip install seaborn
#!pip install --user pandarallel
#!pip install --user toolkit4nlp
import seaborn as sns

import numpy as np
import pandas as pd
from pandarallel import pandarallel
pandarallel.initialize(progress_bar=True)


# In[2]:


import glob
from xlrd import XLRDError
import os, sys
from tqdm import tqdm

import pandas as pd
import numpy as np

from toolkit4nlp.utils import DataGenerator, pad_sequences
from toolkit4nlp.models import build_transformer_model, Model
from toolkit4nlp.layers import *
from toolkit4nlp.tokenizers import Tokenizer
from toolkit4nlp.optimizers import *
from sklearn.metrics import accuracy_score

train_paths = glob.glob('中移编程大赛-初赛数据-开放/train/*')
#test_paths = glob.glob('/home/featurize/data/中移编程大赛-复赛数据-开放/test2/*')
example_sub = pd.read_csv('中移编程大赛-复赛数据-开放/submit_example_test2.csv')
test_paths = example_sub['filename'].tolist()
df_label = pd.read_csv('中移编程大赛-初赛数据-开放/answer_train.csv')
file2label = {}

for i, item in df_label.iterrows():
    file2label[os.path.split(item['filename'])[-1]] = item['label']

all_labels = set(file2label.values())
id2label = {i: label for i, label in enumerate(all_labels)}
label2id = {label: i for i, label in enumerate(all_labels)}

def read_file(path):
    df = None

    columns = []
    content = []
    # get label
    fname = os.path.split(path)[-1]
    label = file2label.get(fname, None)
    try:
        df = pd.read_excel(path)
    except:
        try:
            df = pd.read_csv(path, error_bad_lines=False, encoding='utf8')
        except:
            pass

    if df is not None and len(df) > 0:
        df = df.astype(str)
        columns = list(df.columns)
        #content = df.to_numpy()[0].tolist()
        for i in range(len(df.to_numpy())):
            content = content + df.to_numpy()[i].tolist()# + df.to_numpy()[1].tolist() + df.to_numpy()[2].tolist()
            if(i==10):
                break
        #print(content)
    return [label, fname, columns, content, fname]


def load_train_data(paths):
    data = []
    for path in tqdm(paths):
        data.append(read_file(path))
    return data

def load_test_data(paths):
    data = []
    for path in tqdm(paths):
        data.append(read_file('中移编程大赛-复赛数据-开放/' + path))
    return data
train_data = load_train_data(train_paths)
test_data = load_test_data(test_paths)


# In[3]:


only_title = [t for t in train_data if not t[2]]
has_content = [t for t in train_data if t[2]]
np.random.shuffle(has_content)
half = int(len(has_content) * 0.5)

np.random.shuffle(has_content)
half = int(len(has_content) * 0.5)

remove_title = [[t[0], None] + t[2:] for t in has_content[:half]]
new_has_content = remove_title + has_content[half:]
train_data = only_title + new_has_content



def clean_title(x):
    try:
        temp = x.replace('train/', '').replace('test1/', '').replace('.xls', '').replace('.csv', '').replace('.xlsx','').replace('_', ' ').replace('test2/', '')   
        return temp
    except:
        return ""




import copy
#temp = copy.deepcopy(train_data)
for i in tqdm(train_data):
    i[1] = clean_title(i[1])
    i[2] = ' '.join(i[2])
    i[3] = ' '.join(i[3])
    i[2]= ''.join(re.findall(r"[\u4e00-\u9fa5\s]",i[2]))
    i[3]= ''.join(re.findall(r"[\u4e00-\u9fa5\s]",i[3]))


# In[6]:


for i in tqdm(test_data):
    i[1] = clean_title(i[1])
    i[2] = ' '.join(i[2])
    i[3] = ' '.join(i[3])
    i[1]= ''.join(re.findall(r"[\u4e00-\u9fa5\s]",i[1]))
    i[2]= ''.join(re.findall(r"[\u4e00-\u9fa5\s]",i[2]))
    i[3]= ''.join(re.findall(r"[\u4e00-\u9fa5\s]",i[3]))


# In[ ]:





# In[7]:


train = pd.DataFrame(train_data)
train.columns = ['label','title','columns','content','filename']


# In[8]:


train['all'] = (train['title'] + train['columns'] + train['content'])
#train = train[(train['all'] != '' )] 



test=pd.DataFrame(test_data)
test.columns = ['label','title','columns','content','filename']


# In[11]:


test['all'] = (test['title'] + test['columns'] + test['content'])



import cpca
def remove_loc(info):
    #info ='铁岭市清河区市广东省名牌产品工业类称号信息'
    info_ = [info]
    df = cpca.transform(info_)
    if(df['省'][0] != None):
        info = info.replace(df['省'][0],'')
    if(df['市'][0] != None):
        info = info.replace(df['市'][0],'')
    if(df['区'][0] != None):
        info = info.replace(df['区'][0],'')
    return info
tqdm.pandas(desc='pandas bar')
train['all'] = train.progress_apply(lambda x:remove_loc(x['all']),axis=1)
test['all'] = test.progress_apply(lambda x:remove_loc(x['all']),axis=1)

train['all'] = train.progress_apply(lambda x:remove_loc(x['all']),axis=1)
test['all'] = test.progress_apply(lambda x:remove_loc(x['all']),axis=1)


# In[14]:


error_read = test[test['all'] == '']
error_read_filename = error_read.filename.tolist()
error_read


# In[15]:


def read_file_error(path):
    df = None
    columns = []
    content = []
    # get label
    fname = os.path.split(path)[-1]
    label = file2label.get(fname, None)
    try:
        df = pd.read_excel(path,sheet_name=1)
    except:
        try:
            #df = pd.read_excel(path)
            df = pd.read_csv(path, error_bad_lines=True, encoding='utf8')
        except:
            pass

    if df is not None and len(df) > 0:
        df = df.astype(str)
        columns = list(df.columns)
        #content = df.to_numpy()[0].tolist()
        for i in range(len(df.to_numpy())):
            content = content + df.to_numpy()[i].tolist()# + df.to_numpy()[1].tolist() + df.to_numpy()[2].tolist()
            if(i==10):
                break
    return [label, fname, columns, content, fname]

def load_test_data_error(paths):
    data = []
    for path in tqdm(paths):
        data.append(read_file_error('中移编程大赛-复赛数据-开放/test2/' + path))
    return data
test_error_data = load_test_data_error(error_read_filename)


for i in tqdm(test_error_data):
    i[1] = clean_title(i[1])
    i[2] = ' '.join(i[2])
    i[3] = ' '.join(i[3])
    i[1]= ''.join(re.findall(r"[\u4e00-\u9fa5\s]",i[1]))
    i[2]= ''.join(re.findall(r"[\u4e00-\u9fa5\s]",i[2]))
    i[3]= ''.join(re.findall(r"[\u4e00-\u9fa5\s]",i[3]))


test_error = pd.DataFrame(test_error_data)


test_error.columns = ['label','title','columns','content','filename']
test_error['all'] = (test_error['title'] + test_error['columns'] + test_error['content'])



test_no_error = test[~test['filename'].isin(test_error['filename'].tolist())]
#test = pd.merge(test,test_error,how='left',on='filename')
#test




test = pd.concat([test_error,test_no_error])



#test['all_y'][test['all_x'] == '']
#test['all_y'].str.cat(test['all_x'])# + test['all_y']
#test[test['all_x'] == '']
#test['all_y'] + test['all_x']




test.columns = ['label','filename','title','columns','content','all']



# In[27]:


train['all'] = train.apply(lambda x:x['all'].replace('年', '').replace('月', '').replace('日', ''),axis=1)
test['all'] = test.apply(lambda x:x['all'].replace('年', '').replace('月', '').replace('日', ''),axis=1)



# In[30]:


def remove_other(x):
    x = x.replace('男','').replace('女','').replace('市','').replace('区','')
    return x
train['all'] = train.progress_apply(lambda x:remove_other(x['all']),axis=1)
test['all'] = test.progress_apply(lambda x:remove_other(x['all']),axis=1)



# In[33]:


del train_data
del test_data
import gc
gc.collect()


# In[34]:


import pandas as pd
from sklearn.preprocessing import LabelEncoder


lb = LabelEncoder()
train['label'] = lb.fit_transform(train['label'])
test.drop('label', axis=1, inplace=True)
df = pd.concat([train, test], axis=0)
#df['file'] = df['filename'].apply(lambda x: x.split('.')[0][6:].replace('_', ''))


# In[35]:


import jieba
from keras.preprocessing.text import Tokenizer
from keras.preprocessing.sequence import pad_sequences
from sklearn.model_selection import train_test_split
import pkuseg
seg = pkuseg.pkuseg() 


cw = lambda x: list(seg.cut(x))
df['words'] = df['all'].apply(cw)
tokenizer=Tokenizer() 
tokenizer.fit_on_texts(df['words'])
vocab=tokenizer.word_index


# In[36]:


X = df[~df['label'].isnull()]
y_train = X['label']
X_train = X['words']
X_test = df[df['label'].isnull()]['words']


# In[38]:


from keras.preprocessing.sequence import pad_sequences
from keras.preprocessing.text import Tokenizer
from keras.layers.merge import concatenate
from keras.models import Sequential, Model
from keras.layers import Dense, Embedding, Activation, merge, Input, Lambda, Reshape, BatchNormalization
from keras.layers import Convolution1D, Flatten, Dropout, MaxPool1D, GlobalAveragePooling1D, Conv1D, MaxPooling1D
from keras.layers import LSTM, GRU, TimeDistributed, Bidirectional
from keras.utils.np_utils import to_categorical
from keras import initializers
from keras import backend as K
from keras.engine.topology import Layer
from sklearn.naive_bayes import MultinomialNB
from sklearn.linear_model import SGDClassifier
from sklearn.feature_extraction.text import TfidfVectorizer
import keras
from sklearn import metrics
from sklearn.model_selection import StratifiedKFold
import numpy as np
import tensorflow as tf

# 设置随机数种子
seed = 42
np.random.seed(seed)
tf.set_random_seed(seed)
MAX_LENGTH = 128
def NN():
    model = Sequential()
    model.add(Embedding(len(vocab) + 1, 300, input_length=MAX_LENGTH)) #使用Embeeding层将每个词编码转换为词向量
    model.add(Conv1D(256, 5, padding='same'))
    model.add(MaxPooling1D(3, 3, padding='same'))
    model.add(Conv1D(128, 5, padding='same'))
    model.add(MaxPooling1D(3, 3, padding='same'))
    model.add(Conv1D(64, 3, padding='same'))
    model.add(Flatten())
    model.add(Dropout(0.1))
    model.add(BatchNormalization())  
    model.add(Dense(256, activation='relu'))
    model.add(Dropout(0.1))
    model.add(Dense(20, activation='softmax'))   
    return model



# In[39]:


oof = np.zeros(len(X_train))
predictions = np.zeros((len(X_test), 20))
KF = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
for fold_, (trn_idx, val_idx) in enumerate(KF.split(X_train.values, y_train.values)):
    print("fold n°{}".format(fold_))
    print('trn_idx:',trn_idx)
    print('val_idx:',val_idx)
    X_tr = X_train.iloc[trn_idx]
    X_val = X_train.iloc[val_idx]
    X_train_word_ids = tokenizer.texts_to_sequences(X_tr)
    X_valid_word_ids = tokenizer.texts_to_sequences(X_val)
    X_test_word_ids = tokenizer.texts_to_sequences(X_test)
    X_train_padded_seqs=pad_sequences(X_train_word_ids,maxlen=MAX_LENGTH)
    X_valid_padded_seqs=pad_sequences(X_valid_word_ids,maxlen=MAX_LENGTH)
    X_test_padded_seqs=pad_sequences(X_test_word_ids, maxlen=MAX_LENGTH)
    y_tr = y_train.iloc[trn_idx]
    y_tr = keras.utils.to_categorical(y_tr, num_classes=20)
    y_val = y_train.iloc[val_idx]
    y_val = keras.utils.to_categorical(y_val, num_classes=20)
    model = NN()
    model.compile(loss='categorical_crossentropy',optimizer='adam',metrics=['accuracy'])      
    model.fit(X_train_padded_seqs, y_tr, epochs=10, batch_size=1024, validation_data=(X_valid_padded_seqs, y_val))
    oof[val_idx] = model.predict_classes(X_valid_padded_seqs)
    temp = model.predict_proba(X_test_padded_seqs)
    predictions[:] += temp


# In[40]:


test_label = lb.inverse_transform([np.argmax(i) for i in predictions / np.array(5)])
test['label'] = test_label
#test.to_csv('submission_test2_all.csv', index=False)


# In[41]:


test['filename'] = 'test2/' + test['filename'] 


# In[42]:


test[['filename','label']]


# In[43]:


test[['filename','label']].to_csv('submission_test2_all.csv', index=False)
```
