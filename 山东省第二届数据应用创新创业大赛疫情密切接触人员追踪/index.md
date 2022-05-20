# 山东省第二届数据应用创新创业大赛——疫情密切接触人员追踪


性能优化赛题，k-d树方案。初赛Rank28，复赛Rank16。

<!--more-->

## 赛题介绍
### 赛题任务
　　通过筛选个人基本信息、个人防疫信息、亮码位置、亮码时间等数据，判定直接密接人员，间接密接人员，判定疫情传播风险等级，辅助决策疫情防控力度。  
### 评分标准
　　准确率分数使用**macro F1**计算。

　　直接密接人员：与确诊患者亮码时间差的绝对值在5分钟内，距离在10米以内。  
　　间接密接人员：与直接密接人员的亮码时间差的绝对值在5分钟内，距离在10米以内。  
　　如果既是直接密接人员又是间接密接人员，统一归类为直接密接人员。

　　距离计算参考函数如下：
```Python
from math import radians, cos, sin, asin, sqrt
def geodistance(lng1, lat1, lng2, lat2):
	lng1, lat1, lng2, lat2 = map(radians, [float(lng1), float(lat1), float(lng2), float(lat2)])
    dlon = lng2-lng1
    dlat = lat2-lat1
    a = sin(dlat/2)**2 + cos(lat1) * cos(lat2) * sin(dlon/2)**2
    distance = 2 * asin(sqrt(a)) * 6371.393 * 1000
    distance = round(distance, 5)
    return distance
```
### 数据集描述
　　**个人轨迹数据（df_travel.csv）**：  
　　id：String，人员唯一ID  
　　usetime：Date，亮码时间  
　　lng：Float，亮码位置经度（小数点5位，GCJ02坐标系）  
　　lat：Float，亮码位置纬度（小数点5位，GCJ02坐标系）  

　　**确诊患者亮码记录（confirm.csv）**：   
　　亮码时间：Date，亮码时间；  
　　lng：Float，亮码位置经度（小数点5位，GCJ02坐标系）  
　　lat：Float，亮码位置纬度（小数点5位，GCJ02坐标系）  
　　备注：String，确诊患者的具体行为  
　　*只有一名确诊患者*
## 解决思路
　　整体上来看，方案比较常规，决赛还未开始，对于这题有没有一些特殊的方法，还需要等待答辩结束。  位于Rank5的江离重楼大佬在复赛结束后就分享了自己的代码和思路  
　　为每条轨迹记录添加唯一索引，先根据确诊人员信息和条件判断出直接密接的记录，根据这些记录筛选每个id在接触时间以后的所有记录，最后再根据这些记录和条件判断间接密接人员，**先初筛，后细筛**。
### 直接密接人员
　　轨迹数据的规模为200万+条，确诊人员记录约为50条，规模并不大，所以直接考虑计算所有点对之间的Euclidean Distance / CityBlock Distance，筛选出亮码时间差在300秒以内以及经纬度距离在0.00025内的点对，再根据经纬度计算精确距离差，筛选出直接密接人员。  
　　在复赛阶段对这个方案又做了一些小优化，在线下环境成绩有所提升。由于确诊人员记录数量极少，所以根据这些点的经纬度及时间可以先大幅缩小搜索范围，即先使用集合运算挑选出处在可能范围内的人员轨迹记录，再做如上操作，代码如下。  
```Python
def gen_idx_direct(candidate, target):
    set_target_time = set([j for i in target.time for j in range(i - 300,i + 301)]) 
    set_target_lng = set(np.around([j for i in target.lng for j in np.arange(i - 0.00025,i + 0.00025,0.00001)], 5))
    set_target_lat = set(np.around([j for i in target.lat for j in np.arange(i - 0.00025,i + 0.00025,0.00001)], 5))

    candidate_ = candidate[(
                    candidate.time.isin(list(set_target_time))) & (
                    np.round(candidate.lng, 5).isin(list(set_target_lng))) & (
                    np.round(candidate.lat, 5).isin(list(set_target_lat)))].reset_index(drop=True)

    time_diff = np.where(
                    abs(cdist(candidate_[['time','temp']],
                        target[['time','temp']], metric='cityblock')) <= 300, 1, 0)
    distance = np.where(
                    abs(cdist(candidate_[['lng', 'lat']],
                        target[['lng', 'lat']])) <= 0.00025, 1, 0)
    result_mat = time_diff * distance

    c_idx = np.where(result_mat == 1)[0]
    t_idx = np.where(result_mat == 1)[1]

    distance = geodistance(candidate_.iloc[c_idx, 3],
                           candidate_.iloc[c_idx, 2],
                           target.iloc[t_idx, 2],
                           target.iloc[t_idx, 1])

    c_result = c_idx[np.where(distance <= 10)[0]]
    # t_result = t_idx[np.where(distance <= 10)[0]]
   
    return candidate_.loc[list(set(c_result)), 'idx'].tolist()
```
　　**对距离计算函数进行了修改**，使用numpy，改为矩阵并行计算。
```Python
import numexpr as ne
def geodistance(lng1, lat1, lng2, lat2):
    lng1 = np.radians(np.array(lng1))
    lat1 = np.radians(np.array(lat1))
    lng2 = np.radians(np.array(lng2))
    lat2 = np.radians(np.array(lat2))
    return np.round(ne.evaluate("2 * arcsin(sqrt(sin((lat2 - lat1) / 2) ** 2 + cos(lat1) * cos(lat2) * sin((lng2 - lng1) / 2) ** 2)) * 6371.393 * 1000"), 5)
```
### 间接密接人员
　　关于间接密接人员的搜索，尝试了两种方案，一种是使用KD树，另一种是将经纬度以一定的尺度网格化（0.0001），对于直接密接人员的记录，搜索其附近的九个格子。  
　　我所使用的是KD树的方案，然而比赛结束后看到Rank5 江离.重楼大佬在群里的分享，也是使用的网格化搜索的方案，不过语言方面使用的是C++，这才意识到针对计算密集型程序Python和C++巨大的效率差距，初赛阶段看到以Python作为入口语言，就默认用Python实现了，复赛阶段也一直没向这个方向进行优化，最后只排到了Rank16，这也是做这次比赛的遗憾之处吧~
#### K-D Tree
　　Scipy中提供了KDTree的接口，scipy.spatial.cKDTree，其底层使用C语言实现，效率更高。
```Python
def gen_idx_indirect_kd(candidate, target):
    candidate.reset_index(inplace=True, drop=True)
    target.reset_index(inplace=True, drop=True)
    
    kd_time_diff = cKDTree(candidate[['time']])
    time_diff_idx = kd_time_diff.query_ball_point(target[['time']].values.tolist(), r = 300, p = 1)

    kd_distance = cKDTree(candidate[['lng','lat']])
    distance_idx = kd_distance.query_ball_point(target[['lng','lat']].values.tolist(), r = 0.00025, p = 2)

    intersection = []
    for i in range(len(distance_idx)):
        intersection.append(list(set(distance_idx[i]).intersection(set(time_diff_idx[i]))))

    c_idx = []
    t_idx = []
    for i in range(len(intersection)):
        c_idx.extend(intersection[i])
        t_idx.extend([i] * len(intersection[i]))

    distance = geodistance(candidate.iloc[c_idx, 3],
                           candidate.iloc[c_idx, 2],
                           target.iloc[t_idx, 3],
                           target.iloc[t_idx, 2])

    c_result = [c_idx[i] for i in (np.where((distance <= 10))[0]).tolist()]

    return candidate.iloc[list(set(c_result)), 0].tolist()
```
#### 网格化搜索
　　网格化搜索实现起来相对比较简单，其思想类似于GeoHash，即将二位的经纬度坐标点映射到一维，但GeoHash在这里并不适合，一方面是效率过低，另一方面，和直接将经纬度点映射到0,1,...,n的效果相同。  
  ![GeoHash](https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/ShanDongBD_covid/2095550-d86dc182d102451b.png)
## 关于效率
　　复赛结束时，线上成绩约为30s，并不算优秀。  
　　除计算外，I/O方面的优化也是本题的上分点之一。  
　　关于效率的计算，比较奇怪的是，线上评测成绩的抖动幅度似乎不小，我个人第一次和第二次提交的代码几乎相同，但线上成绩提升了近30%，除此之外，主办方所的线上环境只开放了CPU的一个核心，但经过实验，多进程对该题是work的，能带来大概25%的提升。但如果时间效率足够高的话，多进程显然就不太合适了，进程创建和切换所带来的开销已经不能忽略不计了。  
## 关于准确率
　　初赛使用该方案的macro F1为1.0，但是对于复赛阶段的数据为0.999179，针对复赛的数据如何达到1.0还需要等决赛答辩后学习思路。  
　　补充：经验证，将密接的亮码时间判定从300秒改为310秒即可达到1.0的准确率，逻辑比较迷。  
　　不过这场比赛在密接的判定上确实比较模糊，有些复杂情况难以判定，主办方在一些问题上也没有给出明确的答复，需要多次尝试来进行验证。
## 个人实现完整代码
　　**由纯Python实现**，和C/C++等语言的实现相比，时间效率并不算高，仅供参考。
```Python
import pandas as pd
import numpy as np
from datetime import datetime
from scipy.spatial.distance import cdist
from scipy.spatial import cKDTree
import numexpr as ne
import os
import multiprocessing
import sys

def geodistance(lng1, lat1, lng2, lat2):
    lng1 = np.radians(np.array(lng1))
    lat1 = np.radians(np.array(lat1))
    lng2 = np.radians(np.array(lng2))
    lat2 = np.radians(np.array(lat2))
    return np.round(ne.evaluate("2 * arcsin(sqrt(sin((lat2 - lat1) / 2) ** 2 + cos(lat1) * cos(lat2) * sin((lng2 - lng1) / 2) ** 2)) * 6371.393 * 1000"), 5)

def gen_idx_direct(candidate, target):
    set_target_time = set([j for i in target.time for j in range(i - 300,i + 301)]) 
    set_target_lng = set(np.around([j for i in target.lng for j in np.arange(i - 0.00025,i + 0.00025,0.00001)], 5))
    set_target_lat = set(np.around([j for i in target.lat for j in np.arange(i - 0.00025,i + 0.00025,0.00001)], 5))

    candidate_ = candidate[(
                    candidate.time.isin(list(set_target_time))) & (
                    np.round(candidate.lng, 5).isin(list(set_target_lng))) & (
                    np.round(candidate.lat, 5).isin(list(set_target_lat)))].reset_index(drop=True)

    time_diff = np.where(
                    abs(cdist(candidate_[['time','temp']],
                        target[['time','temp']], metric='cityblock')) <= 300, 1, 0)
    distance = np.where(
                    abs(cdist(candidate_[['lng', 'lat']],
                        target[['lng', 'lat']])) <= 0.00025, 1, 0)
    result_mat = time_diff * distance

    c_idx = np.where(result_mat == 1)[0]
    t_idx = np.where(result_mat == 1)[1]

    distance = geodistance(candidate_.iloc[c_idx, 3],
                           candidate_.iloc[c_idx, 2],
                           target.iloc[t_idx, 2],
                           target.iloc[t_idx, 1])

    c_result = c_idx[np.where(distance <= 10)[0]]
    t_result = t_idx[np.where(distance <= 10)[0]]

    return candidate_.iloc[list(set(c_result)), 4].tolist(), pd.concat([candidate_.iloc[c_result,[0,6]].reset_index(drop=True), (target.iloc[t_result,6]).reset_index(drop=True)],axis=1,ignore_index=True)

def gen_idx_indirect_kd(candidate, target):
    candidate.reset_index(inplace=True, drop=True)
    target.reset_index(inplace=True, drop=True)
    
    kd_time_diff = cKDTree(candidate[['time']])
    time_diff_idx = kd_time_diff.query_ball_point(target[['time']].values.tolist(), r = 300, p = 1)

    kd_distance = cKDTree(candidate[['lng','lat']])
    distance_idx = kd_distance.query_ball_point(target[['lng','lat']].values.tolist(), r = 0.00025, p = 1)

    intersection = []
    for i in range(len(distance_idx)):
        intersection.append(list(set(distance_idx[i]).intersection(set(time_diff_idx[i]))))

    c_idx = []
    t_idx = []
    for i in range(len(intersection)):
        c_idx.extend(intersection[i])
        t_idx.extend([i] * len(intersection[i]))

    distance = geodistance(candidate.iloc[c_idx, 3],
                           candidate.iloc[c_idx, 2],
                           target.iloc[t_idx, 3],
                           target.iloc[t_idx, 2])

    c_result = [c_idx[i] for i in (np.where((distance <= 10))[0]).tolist()]
    
    return candidate.iloc[list(set(c_result)), 0].tolist()

def main(dirname, savePath):
    confirmPath = os.path.join(dirname, "confirm.csv")
    travelPath = os.path.join(dirname, "df_travel.csv")
    
    confirmed = pd.read_csv(confirmPath)
    df_travel = pd.read_csv(travelPath)

    confirmed['亮码时间'] = pd.to_datetime(confirmed['亮码时间'],format='%Y-%m-%d %H:%M:%S')
    df_travel['usetime'] = pd.to_datetime(df_travel['usetime'],format='%Y-%m-%d %H:%M:%S')
    
    df_travel_length = len(df_travel)

    confirmed['idx'] = range(len(confirmed))
    df_travel['idx'] = range(df_travel_length)

    confirmed['temp'] = 0
    df_travel['temp'] = 0

    confirmed['time'] = (pd.to_timedelta(confirmed['亮码时间'] - datetime(2020, 10, 30)).dt.total_seconds()).astype(int)
    df_travel['time'] = (pd.to_timedelta(df_travel['usetime'] - datetime(2020, 10, 30)).dt.total_seconds()).astype(int)
    
    direct_idx, df_direct_min_time_ = gen_idx_direct(df_travel, confirmed)
    df_direct_min_time_.columns = ['id','ctime_direct', 'ctime_confirm']
    
    df_direct = df_travel[df_travel['idx'].isin(direct_idx)]

    df_travel = pd.merge(df_travel, df_direct_min_time_.groupby('id').agg({'ctime_direct':min}), on='id', how='left')
    df_travel.ctime_direct.fillna(-1,inplace=True)

    df_travel['direct'] = 0
    df_travel.loc[(df_travel['time'] >= df_travel['ctime_direct']) & (df_travel['ctime_direct'] >= 0),'direct'] = 1

    df_direct = df_travel[df_travel['direct'] == 1]

    #df_indirect_min_time = gen_idx_indirect_kd(df_travel[df_travel['direct'] == 0], df_direct)
    #indirect_id = df_indirect_min_time.id_i.tolist()

    p_i = multiprocessing.Pool(3)
    indirect_id_0 = p_i.apply_async(gen_idx_indirect_kd, args=(df_travel.iloc[np.arange(0, df_travel_length, 3)], df_direct))
    indirect_id_1 = p_i.apply_async(gen_idx_indirect_kd, args=(df_travel.iloc[np.arange(1, df_travel_length, 3)], df_direct))
    indirect_id_2 = p_i.apply_async(gen_idx_indirect_kd, args=(df_travel.iloc[np.arange(2, df_travel_length, 3)], df_direct))
    p_i.close()
    p_i.join()

    submission = pd.DataFrame({'id':list(set(df_travel['id'].tolist())),'label':0})
    submission.loc[submission['id'].isin(list(set([i for l in [indirect_id_0.get(),indirect_id_1.get(),indirect_id_2.get()] for i in l]))),'label'] = 2
    submission.loc[submission['id'].isin(df_direct['id'].tolist()),'label'] = 1
    submission.to_csv(savePath,index=False)

if __name__ == "__main__":
    dirname = sys.argv[1]  # 所需预测的文件夹
    savePath = sys.argv[2]  # 预测结果保存文件
    main(dirname, savePath)
```
## Rank5实现（江离.重楼）
　　转自群聊。
```C++
#include <cstdio>
#include <cmath>
#include <algorithm>
#include <sys/mman.h>
#include <unistd.h>
#include <fcntl.h>
#include <cstring>

#define MAX 15000005

using namespace std;

double radians(double a) {
    return a / 180.0 * 3.141592653589793;
}

double sqr(double a) {
    return a * a;
}

int touchCnt = 0;

struct MyHashSet {
    struct Node {
        long long value;
        int nxt;
    } myHash[MAX];
    const int MOD = (1 << 19) - 1;
    int myHashCnt = MOD + 1;

    void insert(long long value) {
        int now = (value & MOD);
        while (myHash[now].nxt) {
            if (myHash[now].value == value) {
                return;
            }
            now = myHash[now].nxt;
        }
        myHash[now].value = value;
        myHash[now].nxt = myHashCnt++;
    }
    bool has(long long value) {
        int now = (value & MOD);
        while (myHash[now].nxt) {
            if (myHash[now].value == value) {
                return true;
            }
            now = myHash[now].nxt;
        }
        return false;
    }
} idHashSet;

struct MyHashMap {
    struct Node {
        long long value;
        int realValue;
        int nxt;
    } myHash[MAX];
    const int MOD = (1 << 16) - 1;
    int myHashCnt = MOD + 1;

    void insert(long long value, int realValue) {
        int now = (value & MOD);
        while (myHash[now].nxt) {
            if (myHash[now].value == value) {
                myHash[now].realValue = realValue;
                return;
            }
            now = myHash[now].nxt;
        }
        myHash[now].realValue = realValue;
        myHash[now].value = value;
        myHash[now].nxt = myHashCnt++;
    }
    int find(long long value) {
        int now = (value & MOD);
        while (myHash[now].nxt) {
            if (myHash[now].value == value) {
                return myHash[now].realValue;
            }
            now = myHash[now].nxt;
        }
        return -1;
    }
} startTime, level;

struct Record {
    char *pos;
    long long idHash = 0;
    int time;
    int lat, lng;
    bool isxg = false;

    bool operator < (const Record &b) {
        return time < b.time;
    }

    bool IsTouch(const Record &b) {
        touchCnt++;

        if (abs(lng - b.lng) > 50 || fabs(lat - b.lat) > 50) {
            return false;
        }
        double lng1 = radians((double)lng / 100000);
        double lat1 = radians((double)lat / 100000);
        double lng2 = radians((double)b.lng / 100000);
        double lat2 = radians((double)b.lat / 100000);
        double dlon = lng2 - lng1;
        double dlat = lat2 - lat1;
        double a = sqr(sin(dlat / 2))  + cos(lat1) * cos(lat2) * sqr(sin(dlon / 2));
        double distance = 2 * asin(sqrt(a)) * 6371.393 * 1000;
        return distance < 10 + 1e-9;
    }

    double getDist(const Record &b) {
        double lng1 = radians((double)lng / 100000);
        double lat1 = radians((double)lat / 100000);
        double lng2 = radians((double)b.lng / 100000);
        double lat2 = radians((double)b.lat / 100000);
        double dlon = lng2 - lng1;
        double dlat = lat2 - lat1;
        double a = sqr(sin(dlat / 2))  + cos(lat1) * cos(lat2) * sqr(sin(dlon / 2));
        double distance = 2 * asin(sqrt(a)) * 6371.393 * 1000;
        return distance;
    }
};

struct MyBucket {
    struct Node {
        int x, y;
    };
    Node nodes[MAX];
    int cnt = 2000000;

    void insert(int key, int id) {
        cnt++;
        nodes[cnt].x = id;
        if (nodes[key].x == 0) {
            nodes[key].x = nodes[key].y = cnt;
        } else {
            nodes[nodes[key].y].y = cnt;
            nodes[key].y = cnt;
        }
    }

    void pop(int key) {
        nodes[key].x = nodes[nodes[key].x].y;
    }
} /*latids, lngids*/ ids;

Record temp[MAX], records[MAX];
int recordSize = 0;

const int MAXY = 3000;
int days[MAXY][15];
void InitDays() {
    int cur = 0;
    for (int y = 2000; y < MAXY; y++) {
        for (int m = 1; m <= 12; m++) {
            days[y][m] = cur;
            if (m == 1 || m == 3 || m == 5 || m == 7 || m == 8 || m == 10 || m == 12) {
                cur += 31;
            } else if (m == 4 || m == 6 || m == 9 || m == 11) {
                cur += 30;
            } else {
                if ((y % 100 != 0 && y % 4 == 0) || (y % 400 == 0)) {
                    cur += 29;
                } else {
                    cur += 28;
                }
            }
        }
    }
}

int gettime(struct tm &tm) {
    return (((days[tm.tm_year][tm.tm_mon] + tm.tm_mday) * 24 + tm.tm_hour) * 60 + tm.tm_min) * 60 + tm.tm_sec;
}

char *buffer;

void ReadRecords(Record* records, Record *result, char *recordFile, char *travelFile) {
    auto sst = clock();
    //records.clear();

    /*Record cur;
    FILE *fi = fopen("../data/record", "r");
    while (cur.ReadRecord(fi)) {
        records.push_back(cur);
    }
    fclose(fi);
    fi = fopen("../data/travel", "r");
    while (cur.ReadTravel(fi)) {
        records.push_back(cur);
        level[cur.id] = 0;
    }
    fclose(fi);*/

    /*FILE *fi = fopen(recordFile, "rb");
    fseek(fi, 0L, SEEK_END);
    int len = ftell(fi);
    fseek(fi, 0L, SEEK_SET);*/
    //buffer = new char[len];
    //int ret = fread(buffer, sizeof(char), len, fi);
    int fd = open(recordFile, O_RDONLY);
    int len = lseek(fd, 0, SEEK_END);
    buffer = (char *) mmap( NULL,  len ,PROT_READ, MAP_PRIVATE, fd, 0 );
    char *pos = buffer;
    char *end = buffer + len;
    while (*pos != '\n') {
        pos++;
    }
    pos++;
    int minTime = (1 << 30);
    int maxTime = 0;
    while (pos < end) {
        auto &record = records[recordSize++];
        record.isxg = true;
        record.time = (((days[(pos[0] - '0') * 1000 + (pos[1] - '0') * 100 + (pos[2] - '0') * 10 + pos[3] - '0'][(pos[5] - '0') * 10 + pos[6] - '0']
                         + ((pos[8] - '0') * 10 + pos[9] - '0')) * 24 +
                ((pos[11] - '0') * 10 + pos[12] - '0')) * 60 + ((pos[14] - '0') * 10 + pos[15] - '0')) * 60 + ((pos[17] - '0') * 10 + pos[18] - '0');
        minTime = min(minTime, record.time);
        maxTime = max(maxTime, record.time);
        pos += 20;
        while (*pos != '.') {
            (record.lat *= 10) += ((*pos++) - '0');
        }
        pos++;
        int fz = 0, fm = 1;
        while (*pos != ',' && fm < 100000) {
            (fz *= 10) += ((*pos++) - '0');
            fm *= 10;
        }
        if (*pos != ',') {
            if ((*pos) >= '5') {
                fz++;
            }
            while (*pos != ',') {
                pos++;
            }
        }
        (record.lat *= 100000) += (fz * (100000 / fm));
        pos++;
        while (*pos != '.') {
            (record.lng *= 10) += ((*pos++) - '0');
        }
        pos++;
        fz = 0, fm = 1;
        while (*pos != ',' && fm < 100000) {
            (fz *= 10) += ((*pos++) - '0');
            fm *= 10;
        }
        if (*pos != ',') {
            if ((*pos) >= '5') {
                fz++;
            }
            while (*pos != ',') {
                pos++;
            }
        }
        (record.lng *= 100000) += (fz * (100000 / fm));
        pos++;
        while (*pos != '\n' && pos < end) {
            pos++;
        }
        pos++;
    }
    //delete[] buffer;
    /*fclose(fi);

    fi = fopen(travelFile, "rb");
    fseek(fi, 0L, SEEK_END);
    len = ftell(fi);
    fseek(fi, 0L, SEEK_SET);*/
    //buffer = new char[len];
    //ret = fread(buffer, sizeof(char), len, fi);
    fd = open(travelFile, O_RDONLY);
    len = lseek(fd, 0, SEEK_END);
    posix_fadvise(fd, 0, len, POSIX_FADV_SEQUENTIAL);
    buffer = (char *) mmap( NULL,  len ,PROT_READ, MAP_PRIVATE, fd, 0 );
    pos = buffer;
    end = buffer + len;
    while (*pos != '\n') {
        pos++;
    }
    pos++;

    printf("read spend %f s.\n", (double)(clock() - sst) / CLOCKS_PER_SEC);
    while (pos < end) {
        auto &record = records[recordSize++];
        record.isxg = false;
        record.pos = pos;
        while (*pos != ',') {
            (record.idHash *= 13131) += *(pos++);
        }
        pos++;

        record.time = (((days[(pos[0] - '0') * 1000 + (pos[1] - '0') * 100 + (pos[2] - '0') * 10 + pos[3] - '0'][(pos[5] - '0') * 10 + pos[6] - '0']
                         + ((pos[8] - '0') * 10 + pos[9] - '0')) * 24 +
                        ((pos[11] - '0') * 10 + pos[12] - '0')) * 60 + ((pos[14] - '0') * 10 + pos[15] - '0')) * 60 + ((pos[17] - '0') * 10 + pos[18] - '0');
        minTime = min(minTime, record.time);
        maxTime = max(maxTime, record.time);
        pos += 20;
        while (*pos != '.') {
            (record.lat *= 10) += ((*pos++) - '0');
        }
        pos++;
        int fz = 0, fm = 1;
        while (*pos != ',' && fm < 100000) {
            (fz *= 10) += ((*pos++) - '0');
            fm *= 10;
        }
        if (*pos != ',') {
            if ((*pos) >= '5') {
                fz++;
            }
            while (*pos != ',') {
                pos++;
            }
        }

        (record.lat *= 100000) += (fz * (100000 / fm));
        pos++;
        while (*pos != '.') {
            (record.lng *= 10) += ((*pos++) - '0');
        }
        pos++;
        fz = 0, fm = 1;
        while (*pos != '\n' && pos < end && fm < 100000) {
            (fz *= 10) += ((*pos++) - '0');
            fm *= 10;
        }
        if (*pos != ',') {
            if ((*pos) >= '5') {
                fz++;
            }
            while (*pos != '\n' && pos < end) {
                pos++;
            }
        }
        (record.lng *= 100000) += (fz * (100000 / fm));
        pos++;
    }
    //fclose(fi);

    printf("read all spend %f s.\n", (double)(clock() - sst) / CLOCKS_PER_SEC);
    /*maxTime -= minTime;
    int *sum = new int[maxTime + 1];
    memset(sum, 0, sizeof(int) * (maxTime + 1));
    for (int i = 0; i < recordSize; i++) {
        sum[records[i].time - minTime]++;
    }
    for (int i = 1; i <= maxTime; i++) {
        sum[i] += sum[i - 1];
    }
    for (int i = 0; i < recordSize; i++) {
        result[--sum[records[i].time - minTime]] = records[i];
    }*/
    sort(records, records + recordSize);
    printf("finish all spend %f s.\n", (double)(clock() - sst) / CLOCKS_PER_SEC);
}

extern "C" {
void work(char *recordFile, char *travelFile, char *outputFile) {
    auto st = clock();
    InitDays();
    ReadRecords(records, temp, recordFile, travelFile);
    printf("finish read at %f s.\n", (double) (clock() - st) / CLOCKS_PER_SEC);
    for (int i = 0; i < recordSize; i++) {
        if (records[i].isxg) {
            auto &cur = records[i];
            int st = i;
            while (st > 0 && cur.time - records[st - 1].time <= 300) {
                st--;
            }
            for (int j = st; j < recordSize && records[j].time - cur.time <= 300; j++) {
                if (!records[j].isxg && cur.IsTouch(records[j])) {
                    long long h = records[j].idHash;
                    int f = startTime.find(h);
                    if (startTime.find(h) == -1) {
                        startTime.insert(h, records[j].time);
                        level.insert(h, 1);
                    } else if (records[j].time < f) {
                        startTime.insert(h, records[j].time);
                    }
                }
            }
        }
    }

    printf("finish phase1 at %f s.\n", (double) (clock() - st) / CLOCKS_PER_SEC);

    int l = 0, r = 0;
    int idCnt = 0;
    for (int i = 0; i < recordSize; i++) {
        if (!records[i].isxg && startTime.find(records[i].idHash) == -1) {
            int timeNow = records[i].time;
            while (r < recordSize && timeNow + 300 >= records[r].time) {
                int it = startTime.find(records[r].idHash);
                if (it != -1 && records[r].time >= it) {
                    //latids.insert(records[r].lat / 100, r);
                    //lngids.insert(records[r].lng / 100, r);
                    ids.insert((records[r].lat / 50 + records[r].lng / 50), r);
                    idCnt++;
                }
                r++;
            }
            while (l < r && records[l].time + 300 < timeNow) {
                int it = startTime.find(records[l].idHash);
                if (it != -1 && records[l].time >= it) {
                    //latids.pop(records[l].lat / 100);
                    //lngids.pop(records[l].lng / 100);
                    ids.pop((records[l].lat / 50 + records[l].lng / 50));
                    idCnt--;
                }
                l++;
            }
            if (idCnt == 0) {
                continue;
            }
            auto &cur = records[i];
            if (level.find(cur.idHash) != -1) {
                continue;
            }
            int intLat = cur.lat / 50;
            int intLng = cur.lng / 50;
            bool touch = false;
            /*for (int d = -1; d <= 1; d++) {
                for (int e = -1; e <= 1; e++) {*/
            for (int de = 0; de <= 2; de++) {
                int id = ids.nodes[intLat + intLng + de].x;
                while (id != 0) {
                    if (cur.IsTouch(records[ids.nodes[id].x])) {
                        level.insert(cur.idHash, 2);
                        touch = true;
                        break;
                    }
                    id = ids.nodes[id].y;
                }
                if (touch) {
                    break;
                }
                if (de != 0) {
                    id = ids.nodes[intLat + intLng - de].x;
                    while (id != 0) {
                        if (cur.IsTouch(records[ids.nodes[id].x])) {
                            level.insert(cur.idHash, 2);
                            touch = true;
                            break;
                        }
                        id = ids.nodes[id].y;
                    }
                    if (touch) {
                        break;
                    }
                }
            }
/*
            for (int d = -1; d <= 1; d++) {
                int id = latids.nodes[intLat + d].x;
                while (id != 0) {
                    if (cur.IsTouch(records[latids.nodes[id].x])) {
                        level.insert(cur.idHash, 2);
                        touch = true;
                        break;
                    }
                    id = latids.nodes[id].y;
                }
                if (touch) {
                    break;
                }

                id = lngids.nodes[intLng + d].x;
                while (id != 0) {
                    if (cur.IsTouch(records[lngids.nodes[id].x])) {
                        level.insert(cur.idHash, 2);
                        touch = true;
                        break;
                    }
                    id = lngids.nodes[id].y;
                }
                if (touch) {
                    break;
                }
            }
*/
        }
    }
    printf("touchCnt = %d\n", touchCnt);
    printf("finish phase2 at %f s.\n", (double) (clock() - st) / CLOCKS_PER_SEC);

    FILE *fo = fopen(outputFile, "wb");
    char *outputBuffer = new char[1024 * 1024];
    char *walk = outputBuffer;
    (*walk++) = 'i';
    (*walk++) = 'd';
    (*walk++) = ',';
    (*walk++) = 'l';
    (*walk++) = 'a';
    (*walk++) = 'b';
    (*walk++) = 'e';
    (*walk++) = 'l';
    (*walk++) = '\n';
    for (int i = 0; i < recordSize; i++) {
        if (records[i].idHash == 0 || idHashSet.has(records[i].idHash)) {
            continue;
        }
        idHashSet.insert(records[i].idHash);
        for (char *pos = records[i].pos; *pos != ','; pos++) {
            (*walk++) = *pos;
        }
        (*walk++) = ',';
        auto it = level.find(records[i].idHash);
        (*walk++) = '0' + (it == -1 ? 0 : it);
        (*walk++) = '\n';
        if (walk - outputBuffer > 128 * 1024) {
            fwrite(outputBuffer, 1, walk - outputBuffer, fo);
            walk = outputBuffer;
        }
    }
    fwrite(outputBuffer, 1, walk - outputBuffer, fo);
    fclose(fo);
    printf("all spend %f s.\n", (double) (clock() - st) / CLOCKS_PER_SEC);
}
}

int main() {
    Record a, b;
    a.lat = 3600000;
    a.lng = 11700000;
    b.lat = 3600020;
    b.lng = 11700000;
    printf("%f\n", a.getDist(b));
    work("../data/record.csv", "../data/df_travel.csv", "../submit.csv");
    return 0;
}
```
