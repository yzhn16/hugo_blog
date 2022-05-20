# 2022招商银行FinTech精英训练营 数据赛道首日基线


一个简单的方案，lgb五折，无特征工程，仅简单处理了缺失值，CV 0.9504 LB 0.9535。

<!--more-->

```python
import pandas as pd
import lightgbm as lgb
import numpy as np
from sklearn.model_selection import StratifiedKFold
from sklearn.metrics import roc_auc_score


flag = False

if flag:
    train = pd.read_excel(f'data/data142927/train.xlsx')
    test = pd.read_excel(f'data/data142927/test_A榜.xlsx')

    train = train.replace('?', np.nan)
    test = test.replace('?', np.nan)

    train.to_csv(f'train.csv', index=False)
    test.to_csv(f'test.csv', index=False)


train = pd.read_csv(f'train.csv')
test = pd.read_csv(f'test.csv')

train = train.fillna(-1)
test = test.fillna(-1)

def gen_features(df):
    df['MON_12_CUST_CNT_PTY_ID'] = df['MON_12_CUST_CNT_PTY_ID'].map({-1: -1, 'Y': 1}).astype(np.int8)
    df['WTHR_OPN_ONL_ICO'] = df['WTHR_OPN_ONL_ICO'].map({-1: -1, 'A': 1, 'B': 2}).astype(np.int8)
    df['LGP_HLD_CARD_LVL'] = df['LGP_HLD_CARD_LVL'].map({-1: -1, 'A': 1, 'B': 2, 'C': 3, 'D': 4, 'E': 5, 'F': 6}).astype(np.int8)
    df['NB_CTC_HLD_IDV_AIO_CARD_SITU'] = df['NB_CTC_HLD_IDV_AIO_CARD_SITU'].map({-1: -1, 'A': 1, 'B': 2, 'C': 3, 'D': 4, 'E': 5, 'F': 6}).astype(np.int8)

    return df

def reduce_mem_usage(df):
    """ iterate through all the columns of a dataframe and modify the data type
        to reduce memory usage.        
    """
    start_mem = df.memory_usage().sum() / 1024**2
    print('Memory usage of dataframe is {:.2f} MB'.format(start_mem))
    
    for col in df.columns:
        col_type = df[col].dtype
        
        if col_type != object:
            c_min = df[col].min()
            c_max = df[col].max()
            if str(col_type)[:3] == 'int':
                if c_min > np.iinfo(np.int8).min and c_max < np.iinfo(np.int8).max:
                    df[col] = df[col].astype(np.int8)
                elif c_min > np.iinfo(np.int16).min and c_max < np.iinfo(np.int16).max:
                    df[col] = df[col].astype(np.int16)
                elif c_min > np.iinfo(np.int32).min and c_max < np.iinfo(np.int32).max:
                    df[col] = df[col].astype(np.int32)
                elif c_min > np.iinfo(np.int64).min and c_max < np.iinfo(np.int64).max:
                    df[col] = df[col].astype(np.int64)  
            else:
                if c_min > np.finfo(np.float16).min and c_max < np.finfo(np.float16).max:
                    df[col] = df[col].astype(np.float16)
                elif c_min > np.finfo(np.float32).min and c_max < np.finfo(np.float32).max:
                    df[col] = df[col].astype(np.float32)
                else:
                    df[col] = df[col].astype(np.float64)
        else:
            df[col] = df[col].astype('category')

    end_mem = df.memory_usage().sum() / 1024**2
    print('Memory usage after optimization is: {:.2f} MB'.format(end_mem))
    print('Decreased by {:.1f}%'.format(100 * (start_mem - end_mem) / start_mem))
    
    return df

train_data = reduce_mem_usage(gen_features(train))
test_data = reduce_mem_usage(gen_features(test))

drop_features = ['CUST_UID', 'LABEL']
features = [c for c in train_data.columns if c not in drop_features]
target = train_data['LABEL']

nfold = 5

oof_preds = np.zeros((train_data.shape[0]))
oof_label = np.zeros(train_data.shape[0])
test_preds = pd.DataFrame({'CUST_UID': test_data['CUST_UID'], 'LABEL': np.zeros(len(test_data))}, columns=['CUST_UID', 'LABEL'])
feature_importance_df = pd.DataFrame()

folds = StratifiedKFold(n_splits=nfold, shuffle=True, random_state=42)
for i, (trn_idx, val_idx) in enumerate(folds.split(train_data, target)):
    print('---------- fold', i + 1, '----------')
    trn_X, val_X = train_data[features].iloc[trn_idx, :], train_data[features].iloc[val_idx, :]
    trn_y, val_y = target[trn_idx], target[val_idx]

    dtrn = lgb.Dataset(trn_X, label=trn_y)
    dval = lgb.Dataset(val_X, label=val_y)

    parameters = {
        'learning_rate': 0.01,
        'boosting_type': 'gbdt',
        'objective': 'binary',
        'metric': 'auc',
        'num_leaves': 63,
        'feature_fraction': 0.8,
        'bagging_fraction': 0.8,
        'min_data_in_leaf': 32,
        'verbose': -1,
        'nthread': 12
    }

    lgb_model = lgb.train(
        parameters,
        dtrn,
        num_boost_round=2000,
        valid_sets=[dval],
        early_stopping_rounds=100,
        verbose_eval=100,
    )
    oof_preds[val_idx] = lgb_model.predict(val_X[features], num_iteration=lgb_model.best_iteration)
    oof_label[val_idx] = val_y

    test_preds['LABEL'] += lgb_model.predict(test_data[features], num_iteration=lgb_model.best_iteration) / nfold
    
    fold_importance_df = pd.DataFrame()
    fold_importance_df["feature"] = features
    fold_importance_df["importance"] = lgb_model.feature_importance(importance_type='gain')
    fold_importance_df["fold"] = i + 1
    feature_importance_df = pd.concat([feature_importance_df, fold_importance_df], axis=0)

print('----------- over -----------')
print('oof auc:', roc_auc_score(oof_label, oof_preds))

test_preds.to_csv('result.txt', index=None, header=None, sep=' ')
```
