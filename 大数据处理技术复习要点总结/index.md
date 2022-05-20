# 大数据处理技术复习要点总结

本文就『Lending Club贷款数据转换与融合』、『行星数据分组与聚合』以及『德国能源数据时间序列分析』等案例对Pandas中的部分分析方法以及机器学习中部分算法进行总结。
<!--more-->

## Lending Club贷款数据转换与融合
 　　该案例完整Jupyter Notebook可参考[Lending Club贷款数据转换与融合](http://cookdata.cn/note/view_static_note/5acc2adb881ca8e68cfbb1cd1347d28d/)。
### 数据源
```
user = pd.read_csv("./input/user.csv")
loan = pd.read_csv("./input/loan.csv")
history = pd.read_csv("./input/history.csv")
```

### 随机采样(sample)
　　随机查看贷款交易数据中的5行。 

```
loan.sample(n=5)
```

　　随机查看贷款交易数据中的1%。  
```
loan.sample(frac=0.01)
```
　　sample默认的是不放回采样（每个样本只可能出现一次），可以调整replace参数为True改为有放回采样。
```
loan.sample(n=10,replace=True)
```
　　若希望重复调用某次采样的结果，可以设定random_state参数为同一个数来实现。
```
loan.sample(n=5,random_state=42)
```
　　除了行采样，sample也可以实现列采样，只需要调整axis参数为1即可。
```
loan.sample(n=3,axis=1)
```

### 数据融合(merge,join)
```
test_user = user.loc[[1,3,5,7,8]]
test_loan = loan[loan.user.isin([2,3,4,5,6,7])]
```
　　merge和join作为Pandas中常用的数据融合方法，目的都是将两个数据表通过共同变量进行连接。  

#### merge
　　merge参数如下：
```
left.merge(right, how='inner', on=None, 
		left_on=None, right_on=None, 
		left_index=False, right_index=False, 
		sort=False, suffixes=('_x', '_y'), indicator=False)
```
　　left和right分别指代要进行连接的两个数据框。  
　　on, left_on, right_on, 用来指定连接的变量。若这一变量在两个数据框中命名相同，直接使用on指定即可，否则通过left_on和right_on分别指定左表变量名和右表变量名。  
　　若需要基于数据框的索引进行连接，则要通过设定left_index和right_index的参数为True来实现。  
　　how为连接方式，有'inner', 'left', 'right', 'outer'四种。  

　　基于用户信息数据的'user_id'变量和贷款交易数据的'user'变量进行内连接(inner)。这种方式下，只有所选定列在左表与右表能匹配的行会被保留。
```
test_user.merge(test_loan,how="inner",
				left_on="user_id",right_on="user")
```
　　基于用户信息数据的'user_id'变量和贷款交易数据的'user'变量进行左连接(left)。这种方式下，左表所有行都被保留，不能匹配的部分用缺失值填充。
```
test_user.merge(test_loan,how="left",
				left_on="user_id",right_on="user")
```
　　基于用户信息数据的'user_id'变量和贷款交易数据的'user'变量进行右连接(right)。这种方式下，右表所有行都被保留，不能匹配的部分用缺失值填充。
```
test_user.merge(test_loan,how="right",
				left_on="user_id",right_on="user")
```
　　基于用户信息数据的'user_id'变量和贷款交易数据的'user'变量进行外连接(outer)。这种方式下，左表和右表所有行都会被保留，不能匹配的部分用缺失值填充。
```
test_user.merge(test_loan,how="outer",
				left_on="user_id",right_on="user")
```
　　merge中的indicator参数能很好地找到返回结果的来源,设定indicator参数为Ture后，返回结果中多了一列"\_merge"，取值有"both", "left\_only", "right\_only"三种。分别代表左右表匹配成功，左表有而右表没有，右表有而左表没有三种情况。  

$$
\begin{array}{c|c|c}
 & .. & \_merge \\
\hline
 0 & .. & both \\
\hline
 1 & .. & left\_only \\
\hline
 2 & .. & right\_only
\end{array}
$$

　　当左表与右表中变量同名时，我们可以通过suffixes参数为左表变量与右表变量附加不同字段，便于后续区分。  

$$
\begin{array}{c|c|c|c|c|c|c|c}
 & user & term\_l & grade\_l & .. & term\_r & grade\_r & .. \\
\hline
0 & 6 & 60 months & .. & .. & 60 months & .. & .. \\
\hline
.. & .. & .. & .. & .. & .. & .. & ..
\end{array}
$$

#### join
　　join参数如下：
```
left.join(right, on=None, how='left',
		lsuffix='', rsuffix='', sort=False)
```
　　事实上，join就是merge的简化版本，所有join能实现的操作，都可以使用merge实现。  
 - 使用join时，右表只能基于索引进行连接；  
 - 通过on参数，可以指定左表进行连接的变量（可以是索引也可以是任意列）。   


　　merge和join中还有一个参数sort，指定为True会让返回的结果按连接变量进行升序排列。  

### 排序(sort_index,sort_values)
　　Pandas中的sort_index和sort_values也可以对DataFrame进行排序，sort_index是按照索引进行排序，sort_values是按照指定变量排序。  
　　例如想将用户历史数据按账户平均存款排序。
```
history.sort_values(by='avg_cur_bal')
```
　　若要降序排列，可以指定ascending参数为False。
```
history.sort_values(by='avg_cur_bal',
					ascending=False)
```
　　也可以指定对缺失值的排序方式，默认缺失值将排在最后，可以设定na_position为first将缺失值排在最前面。
```
history.sort_values(by='avg_cur_bal',
					na_position='first')
```

### 离散化(cut,qcut)
#### cut
　　使用cut函数按照指定的分割点对数据进行划分。  
　　通过设定bin参数设定了分割点，将数据按照中位数进行了划分。同时设定了参数labels，使用这个参数可以方便地为新的划分区间命名。  
　　**左开右闭区间**  

```
annual_inc = pd.cut(combine.annual_inc,
					bins=[np.min(combine.annual_inc)-1,
                    	np.percentile(combine.annual_inc,50),
                    	np.max(combine.annual_inc)+1],
					labels=['low','high'])
```
　　cut也可以直接指定划分份数，将数据等距划分。  
　　例如，将数据等距分为五份：
```
pd.cut(combine.annual_inc,5)
```
　　理论上，数据应被等距分为了五份，每一个区间的长度都相同，但我们计算可以发现，第一个区间的长度为113364，而其他几个区间的长度都为112800。这并不是cut分割错误，只是为了包含最小值或最大值，**cut的左右端会拓展0.1%**。

#### qcut
　　Pandas中与cut相似的另一个函数是qcut，它将按照每个区间中频数相同的原则进行划分,当我们指定划分份数后，就会用相应的分位数进行划分。例如，当我们使用qcut将数据分为两份时，分割点就是中位数，四份时分割点就是四分位数。  
```
pd.qcut(combine.annual_inc,2)
```

### 值替换(replace,map)
　　认为状态为"Charged Off","In Grace Period", "Late (31-120 days)"的贷款有违约风险，视为不良贷款，将其值标记为1，其他贷款标记为0。  
　　使用replace进行值替换。
```
combine['loan_status'].replace(
	to_replace=['Fully Paid','Current','Charged Off','In Grace Period','Late (31-120 days)'],
    value=[0,0,1,1,1],
    inplace=True)
```
　　除了将需要替换的值与替换的新值分别用列表输入外，也可以使用字典进行指定。
```
test_loan.replace(to_replace={'loan_status':{'Fully Paid':0,'Charged Off':0,'In Grace Period':1}})
```
　　也可以同时指定不同变量的不同值替换为相同新值。  
```
test_loan.replace(to_replace={'loan_status':'Fully Paid','grade':'A'},value='Good')
```
　　也可以指定正则表达式进行替换，这时需要设定参数regex为True，代表to_replace部分输入的是正则表达式。如查找所有以C开头的字段并替换为Bad。
```
test_loan.replace(to_replace='C+.*$', value='Bad', regex=True)
```

#### map
　　在Pandas中，如果只是针对某一个Series进行数值替代，我们也可以使用map方法。
```
test_loan['loan_status'].map({'Fully Paid':0,'Charged Off':0,'In Grace Period':1})
```
　　这同样实现了将贷款状态进行替换的效果，但map不能像replace一样直接对DataFrame进行操作。不过map不仅仅可以像上面一样输入字典作为参数，也可以直接输入一个函数进行映射。  
　　例如，将数据中利率低于12%的映射为'Low'，高于12%的映射为'High'。
```
def f(x):
    if x < 12:
        return 'Low'
    else:
        return 'High'
        
combine['int_rate'].map(f)
```

### 哑变量处理(get_dummies)
```
cat_vars=['term','grade','emp_length','annual_inc','home_ownership','verification_status']
for var in cat_vars:
    cat_list = pd.get_dummies(combine[var], prefix=var, drop_first=True)
    combine=combine.join(cat_list)
```
　　get_dummies函数中使用了两个参数。prefix可以为新生成的哑变量添加前缀，这方便我们识别新生成的变量是从原来哪一个变量中得来的。drop_first设置为True将删去所获得哑变量的第一个，这是因为在建模中，有k类的分类变量只需要k-1个变量就可以将其描述，如果使用k个变量则会出现完全共线性的问题。  
　　此外，在这里选择了使用join而不是merge，这是因为get_dummies返回的结果与原始数据有相同的索引，使用join直接基于索引进行连接更简洁。
```
 pd.get_dummies(combine['grade'], prefix='grade',drop_first=True)[:5]
```

$$
\begin{array}{c|c|c}
&grade\_B&	grade_C&	grade_D&	grade_E&	grade_F&	grade_G\\
\hline
0&	0&	0&	1&	0&	0&	0& \\
\hline
1&	0&	1&	0&	0&	0&	0& \\
\hline
2&	1&	0&	0&	0&	0&	0& \\ 
\hline
3&	0&	0&	0&	0&	0&	0&\\
\hline
4&	0&	1&	0&	0&	0&	0&
 \end{array}
$$

　　如果使用merge：
```
combine=combine.merge(cat_list,left_index=True,right_index=True)
```

### 添加常数项列(concat)
　　在回归分析中，我们往往还需要为自变量添加常数项列，值全为1。  
　　首先创建一个长度为X的行数，值全为1的列表。再将其转化为Series，并命名"const"。
```
const = pd.Series([1] * combine.shape[0],name="const")
```
　　重设X索引，使用concat对数据进行合并，并指定方向为列。  
```
X.reset_index(drop=True,inplace=True)
X = pd.concat([const,X],axis=1)
```
　　这里之所以要先重新设置X的索引，是因为concat是基于索引进行拼接的。这么看来，对于列的拼接其实直接使用join就可以了，不过目前join只能作为DataFrame的方法，想拼接DataFrame和Series就必须把DataFrame写在前面：
```
X.join(const)
```
　　此外，concat更常用的是进行行的连接。concat参数如下：
```
  pandas.concat(objs, axis=0, join='outer')
```
　　objs: Series,DataFrame等构成的list  
　　axis: 合并连接的方向，0是行，1是列  
　　join：连接方式，"inner"或者"outer"  

　　可以看到，concat的对象必须是一个list。  

　　创建两个dataframe:df1,df2。
```
df1 = pd.DataFrame([['a','a', 1], ['b','b', 2]],columns=['letter','letter1','number'])
```
$$
\begin{array}{c|c|c|c}
 & letter & letter1	& number\\
\hline
0	& a	& a	& 1\\
\hline
1	& b	& b	& 2
\end{array}
$$
```
df2 = pd.DataFrame([['c','c', 3], ['d','d', 4]],columns=['letter','letter2','number'])
```

$$
\begin{array}{c|c|c|c}
& letter & letter1	& number\\
\hline
0	& c	& c	& 3\\
\hline
1	& d	& d	& 4
\end{array}
$$

　　使用inner方式进行连接，只有能够匹配的变量才会保留。
```
pd.concat([df1,df2],axis=0,join='inner')
```
$$
\begin{array}{c|c|c}
 & letter & number \\
\hline
0 &	a &	1 \\
\hline
1 &	b &	2 \\
\hline
0 &	c &	3 \\
\hline
1 &	d &	4
\end{array}
$$
　　使用outer方式进行连接，所有变量都会保留，不能匹配的部分用缺失值填充。
```
pd.concat([df1,df2],axis=0,join='outer')
```

$$
\begin{array}{c|c|c}
 & letter &	letter1 & letter2 &	number \\
\hline
0 &	a&	a&	NaN&	1 \\ 
\hline
1&	b&	b&	NaN&	2 \\ 
\hline
0&	c&	NaN&	c&	3 \\
\hline
1&	d&	NaN&d	4&
\end{array}
$$


　　ignore_index参数为Ture将忽略原来的索引，从0开始重建索引。
```
pd.concat([df1,df2],ignore_index=True)
```
　　通过key参数可以建立多层索引，方便识别数据来自于哪个数据源。
```
pd.concat([df1,df2],keys=['df1', 'df2'])
```
$$
\begin{array}{c|c|c|c|c|c}
 & & letter & letter1 & letter2& number \\
\hline
df1 & 0 & a & a & NaN & 1 \\
  & 1 & b & b & NaN & 2 \\
\hline
df2 & 0 & c & NaN & c & 3 \\
  & 1 & d & NaN & d & 4 \\
\end{array}
$$

## 行星数据分组与聚合
 　　该案例完整Jupyter Notebook可参考[行星数据分组与聚合](http://cookdata.cn/note/view_static_note/7b30c741facfb06d83bc37ae3a7fa8a3/)。

### 数据源
　　行星数据集记录了2014年之前发现的行星的信息。
```
planets = pd.read_csv("./input/planets.csv")
```
$$
\begin{array}{c|c|c|c|c|c|c}
 & method&number&orbital\_period&mass&distance&year \\
\hline
0&	Radial Velocity	&1&	269.300	&7.10	&77.40	&2006 \\
\hline
1&	Radial Velocity	&1&	874.774	&2.21	&56.95	&2008 \\
\hline
2&	Radial Velocity	&1&	763.000	&2.60	&19.84	&2011 \\
\hline
3&	Radial Velocity	&1&	326.030	&19.40	&110.62	&2007 \\
\hline
4&	Radial Velocity	&1&	516.220	&10.50	&119.47	&2009
\end{array}
$$

### 数据分组
#### 通过特征分组
　　groupby可以指定某一个特征或指定某一组特征进行分组。  
　　例如按method特征对数据进行分组。  
```
grouped = planets.groupby('method')
```
***Output:***
```
<pandas.core.groupby.generic.DataFrameGroupBy object at 0x7f1d00504a58>
```
　　groupby还能通过指定一个与目标数据等长的array、list或Series进行分组。  
　　例如，行星数据集共有1035条记录，生成一个长度为1035，前500都为0，后535都为1的array。  
　　使用repeat，输入中第一部分指定了值，第二部分指定对应值的重复次数。我们将生成结果记为a。
```
a = np.repeat([0,1], [500, 535])
```
　　以其对原数据进行分组，并计算各特征的均值。  
```
planets.groupby(a).mean()
```
$$
\begin{array}{c|c|c|c|c|c}
 &number&orbital\_period&mass&distance&year\\
\hline
0&	1.644000&	1450.908401&	2.580901&	97.615625&	2007.916000& \\
\hline
1&	1.917757&	2526.729858&	2.800112&	507.660000&	2010.149533&
\end{array}
$$

#### 通过函数分组
　　也可以通过函数进行分组。  
　　例如，想将数据按发现年份在2000年前和2000年后进行分组。使用set_index设定year变量为数据的新索引，然后定义一个函数test，当数据小于2000时返回'Before 2000'，大于等于2000时返回'After 2000'，最后通过自定义函数test进行分组，并求各组各变量均值。
```
new = planets.set_index('year')

def test(x):
    if x<2000:
        return 'Before 2000'
    else:
        return 'After 2000'
        
new.groupby(test).mean()
```
$$
\begin{array}{c|c|c|c|c}
 &	number&	orbital\_period&	mass&	distance\\
\hline
After 2000&	1.780658&	2058.025770&	2.615027&	272.918742&\\
\hline
Before 2000&	1.937500&	349.672379&	3.071469&	26.354483&
\end{array}
$$
　　函数的输入是数据的索引列。以上代码相当于这样两步操作：  
　　1. 对目标数据索引的每一个元素执行相应函数；
```
group_index = new.index.map(test)
```
***Output:***
```
Index(['After 2000', 'After 2000', 'After 2000', 'After 2000', 'After 2000',
       'After 2000', 'After 2000', 'Before 2000', 'After 2000', 'After 2000',
       ...
       'After 2000', 'After 2000', 'After 2000', 'After 2000', 'After 2000',
       'After 2000', 'After 2000', 'After 2000', 'After 2000', 'After 2000'],
      dtype='object', name='year', length=1035)
```
　　2. 以这一个新变量group_index进行分组。
```
new.groupby(group_index).mean()
```
$$
\begin{array}{c|c|c|c|c}
 &	number&	orbital\_period&	mass&	distance\\
\hline
year & & & & \\
\hline
After 2000&	1.780658&	2058.025770&	2.615027&	272.918742\\
\hline
Before 2000&	1.937500&	349.672379&	3.071469&	26.354483
\end{array}
$$

### GroupBy对象的基本操作
#### 对分组进行迭代
　　groupby返回的结果是一个GroupBy类型的对象，可以使用循环查看其内部结构：
```
pd.set_option('expand_frame_repr',False)
for (name,group) in grouped:
    print(name)
    print(group.head(n=2),'\n')
```

　　GroupBy类型的对象是由各组组名与其对应的分组数据构成。这里只依据method一个特征进行了分组，若基于多个特征进行分组，则返回的GroupBy的组名会是一个多元元组。
```
for (name,group) in planets.groupby(['method','year']):
    print(name)
```

　　GroupBy内部是由一个个DataFrame组成，可以在循环中对每组数据进行操作。  
　　例如，对每组数据使用shape。为了使输出更美观，我们使用format指定了输出的字符串格式，第一个{0:30s}匹配format中第一个字符串method，并指定字符串长度为30；第二个{1}匹配format中第二个字符串group.shape。
```
for (method, group) in planets.groupby('method'):
    print("{0:30s} shape={1}".format(method, group.shape))
```
***Output:***
```
Astrometry                     shape=(2, 6)
Eclipse Timing Variations      shape=(9, 6)
Imaging                        shape=(38, 6)
Microlensing                   shape=(23, 6)
Orbital Brightness Modulation  shape=(3, 6)
Pulsar Timing                  shape=(5, 6)
Pulsation Timing Variations    shape=(1, 6)
Radial Velocity                shape=(553, 6)
Transit                        shape=(397, 6)
Transit Timing Variations      shape=(4, 6)
```
　　也有一些可以直接对GroupBy使用的方法，例如size方法可以查看每个分组的数据量。
```
grouped.size()
```
***Output:***
```
method
Astrometry                         2
Eclipse Timing Variations          9
Imaging                           38
Microlensing                      23
Orbital Brightness Modulation      3
Pulsar Timing                      5
Pulsation Timing Variations        1
Radial Velocity                  553
Transit                          397
Transit Timing Variations          4
dtype: int64
```
#### 选择指定特征分析
　　针对GroupBy类型的对象，我们可以直接选取出需要的列。例如，取出year特征。
```
grouped['year']
```
　　这仍然是一个GroupBy类型的对象，但和之前的结果相比，这是一个SeriesGroupBy，而之前的是一个DataFrameGroupBy。  
　　对于这种类型，可以直接使用一些聚合函数（如sum、mean、max、min...）。例如，查看不同方法发现的行星与地球距离的中位数：
```
planets.groupby('method')['distance'].median()
```
　　也可以直接使用describe。例如查看不同方法发现行星的时间情况。  
```
grouped['year'].describe()
```
$$
\begin{array}{c|c|c|c|c|c}
 & count & mean & std & min & 25\% & 50\% & 75\% & max \\
\hline
method\\
\hline
Astrometry	&2.0&2011.500000&2.121320&2010.0&2010.75&2011.5&2012.25&2013.0\\
\hline
...
\end{array}
$$
#### 结合分组方法与聚合函数分析
　　首先将年份按每10年进行划分。通过//将年份整除10（向下取整），再乘以10，可以将年份变换为对应年代。再使用astype将类型转换为字符串并加上's'代表对应年代。
```
decade = 10 * (planets['year'] // 10)
decade = decade.astype(str) + 's'
```
　　按发现行星的方法和发现的年代进行分组，并统计相应分组下发现的行星的总数。
```
planets.groupby(['method', decade])['number'].sum()
```
***Output:***
```
method						year 
Astrometry					2010s      2
Eclipse Timing Variations	2000s      5
							2010s     10
Imaging						2000s     29
 							2010s     21
...
```
　　使用unstack将按层次化索引拆开为新的列索引。
```
planets.groupby(['method', decade])['number'].sum().unstack()
```

$$
\begin{array}{c|c|c|c|c}
year&1980s&1990s&2000s&2010s \\
\hline
method \\
\hline
Astrometry&NaN&NaN&NaN&2.0 \\
\hline
Eclipse Timing Variations&NaN&NaN&5.0&10.0\\
\hline
...
\end{array}
$$

### GroupBy.apply
　　apply方法能够分别对每一份分组数据进行对应的函数操作，再合并成一个数据表。因此，apply中使用的函数必须是以DataFrame作为输入的，而且每一个apply语句只能传入一个函数。  
　　例如：使用apply计算不同方法发现的行星在各特征上的极差(最大值与最小值之差)。
```
grouped.apply(lambda x: x.max() - x.min())
```
$$
\begin{array}{c|c|c|c|c|c}
 &number&orbital\_period&mass&distance&year\\
\hline
method \\
\hline
Astrometry&	0.0&769.640000&NaN&5.79	&3.0 \\
\hline
Eclipse Timing Variations&1.0&8303.750000&1.8500&369.28	&4.0 \\
\hline
...
\end{array}
$$

　　此例中，每一组数据返回了一个Series。apply应用的函数也可以只返回一个标量，例如计算每种方法发现的行星中和地球距离的最大值与轨道周期的最大值之比。
***Output:***
```
method
Astrometry                         0.020443
Eclipse Timing Variations          0.048924
Imaging                            0.000226
Microlensing                       1.513725
Orbital Brightness Modulation    763.789269
Pulsar Timing                      0.032854
Pulsation Timing Variations             NaN
Radial Velocity                    0.020418
Transit                           25.633248
Transit Timing Variations         13.243750
dtype: float64
```
　　apply应用的函数也可以返回一个DataFrame。例如分组中心化数据。
```
grouped.apply(lambda x: x-x.mean())
```
$$
\begin{array}{c|c|c|c|c|c|c}
	&distance&	mass&	method&	number&	orbital\_period&	year\\
\hline
0&	25.799792&	4.469301&	NaN&	-0.721519&	-554.05468&	-1.518987\\
\hline
1&	5.349792&	-0.420699&	NaN&	-0.721519&	51.41932&	0.481013\\
\hline
...
\end{array}
$$

　　当apply中运用的函数除了输入的DataFrame外还有其他参数时，直接在apply中进行赋值即可。例如查看按method分组后各组数据的前两行。
```
def func1(x,n):
    return(x.head(n))
grouped.apply(func1,n=2)
```
　　apply的运行实际过程有分开运算、结果合并两步，因此，在数据量较大时，apply的运行速度会比可以实现同样操作的其他方法要慢。
```
%timeit grouped.apply(np.mean)
```
*16.1 ms ± 50.2 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)*  
```
%timeit grouped.mean()
```
*624 µs ± 337 ns per loop (mean ± std. dev. of 7 runs, 1000 loops each)*  

　　总结来看，apply方法只需要传入的函数的输入为DataFrame即可，函数的输出可以是标量、Series或DataFrame。但必须以DataFrame为输入就会导致当我们想对不同特征分别进行操作时比较麻烦。例如，分别计算各种方法发现的行星的距离的均值和发现的数量之和。
```
def func2(df):
    mean_distance = np.mean(df['distance'])
    sum_number = np.sum(df['number'])
    return(pd.Series({'mean_distance':mean_distance,'sum_number':sum_number}))

grouped.apply(func2) 
```

### GroupBy.agg
　　同样，分别计算各种方法发现的行星的距离的均值和发现的数量之和。
```
grouped.agg({'distance':'mean','number':'sum'})
```
　　这里使用“变量名:函数名”的形式向agg传入一个字典。agg方法可以针对某个特征同时执行多个函数。
```
grouped.agg({'distance':['min','max','mean','median'],'number':'sum'})
```
　　agg方法中同样可以使用自定义函数，例如求极差：
```
grouped.agg(lambda x: x.max()-x.min())
```
　　需要注意，尽管这里使用agg和apply获得了相同的结果，但是，在apply中是对每一组数据整个DataFrame进行一次运算，而在agg中将对每一组数据中的每一个特征进行运算。
```
def test2(x):
    return(x.shape)
```
```
grouped.apply(test2)
```
***Output:***
```
method
Astrometry                         (2, 6)
Eclipse Timing Variations          (9, 6)
Imaging                           (38, 6)
Microlensing                      (23, 6)
Orbital Brightness Modulation      (3, 6)
Pulsar Timing                      (5, 6)
Pulsation Timing Variations        (1, 6)
Radial Velocity                  (553, 6)
Transit                          (397, 6)
Transit Timing Variations          (4, 6)
dtype: object
```
```
grouped.agg(test2)
```
$$
\begin{array}{c|c|c|c|c|c|c}
	& number&	orbital\_period	& mass & distance &	year \\
\hline
method \\
\hline
Astrometry&	(2,)&	(2,)&	(2,)&	(2,)&	(2,)\\
\hline
Eclipse Timing Variations&	(9,)&	(9,)&	(9,)&	(9,)&	(9,)\\
\hline
...
\end{array}
$$
　　可以看到，apply返回的是每一组数据的维度，而agg返回的是每组数据下，每一个特征对应的数据的维度。因此，我们可以将agg的操作分为3步：  
　　1. 对每组数据中的每一列执行函数;  
　　2. 将每一列返回结果合并;  
　　3. 将每一组数据返回结果合并
```
%timeit grouped.apply(lambda x: x.max()-x.min())
```
*32.2 ms ± 28.9 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)*
```
%timeit grouped.agg(lambda x:x.max()-x.min())
```
*19.6 ms ± 7.73 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)*  

　　因此，尽管相比于apply，agg可以从更细的维度进行数据处理，但也意味着更多的运算消耗。同时，agg方法对使用的函数的返回也有一定要求：对每一个特征，函数只能返回一个标量。例如，使用apply进行中心化的操作无法使用agg完成。
```
grouped.agg(lambda x: x-x.mean())
```
***Output:***
```
ValueError: Shape of passed values is (6, 10), indices imply (5, 10)
```

### GroupBy.transform
　　transform方法中传入的函数只能返回两种结果，可以广播的标量值或者与分组数据维度相同的数据。  
　　对分组数据求均值，然后把这个均值赋值给整个组（可广播的标量值）。

$$
\begin{array}{c|c|c|c|c|c}
 & number&	orbital\_period&	mass&	distance&	year \\
\hline
0&	1.721519&	823.35468&	2.630699&	51.600208&	2007.518987\\
\hline
1&	1.721519&	823.35468&	2.630699&	51.600208&	2007.518987\\
\hline
...
\end{array}
$$

 　　使用`transform`实现分组数据标准化($\dfrac{x-\bar{x}}{s}$)(分组数据维度相同的数据):
```
grouped.transform(lambda x: (x - x.mean()) / x.std())
```

　　apply中自定义函数对每个分组数据单独进行处理，再将结果合并；整个DataFrame的函数输出可以是标量、Series或DataFrame；每个apply语句只能传入一个函数；  
　　agg可以通过字典方式指定特征进行不同的函数操作，每一特征的函数输出必须为标量；  
　　transform不可以通过字典方式指定特征进行不同的函数操作，但函数运算单位也是DataFrame的每一特征，每一特征的函数输出可以是标量或者Series，但标量会被广播。  

## 德国能源数据时间序列分析
 　　该案例完整Jupyter Notebook可参考[德国能源数据时间序列分析](http://cookdata.cn/note/view_static_note/abdeedf821256e7ffacffe03f68e4cf2/)。
### 数据源
　　数据的字段及其说明如下： 
$$
\begin{array}{c|c}
变量名称&含义说明 \\
\hline
Date&日期 \\
\hline
Consumption&电力消耗 \\
\hline
Wind&风能发电量 \\
\hline
Solar&太阳能发电量
\end{array}
$$

　　使用dtypes查看数据类型。
```
opsd.dtypes
```
***Output:***
```
Date           datetime64[ns]
Consumption           float64
Wind                  float64
Solar                 float64
Wind+Solar            float64
dtype: object
```
　　使用set_index将Date变量设定为索引。
```
opsd.set_index('Date',inplace=True)
```
　　也可以在数据导入时通过参数设置实现这些操作。设定index_col为0即以数据中第一列为索引，设定parse_dates为True，会把索引识别为时间数据类型。
```
opsd = pd.read_csv('./input/opsd_germany_daily.csv', index_col=0, parse_dates=True)
```
　　查看此时索引格式。
```
opsd.index
```
***Output:***
```
DatetimeIndex(['2006-01-01', '2006-01-02', '2006-01-03', '2006-01-04',
               '2006-01-05', '2006-01-06', '2006-01-07', '2006-01-08',
               '2006-01-09', '2006-01-10',
               ...
               '2017-12-22', '2017-12-23', '2017-12-24', '2017-12-25',
               '2017-12-26', '2017-12-27', '2017-12-28', '2017-12-29',
               '2017-12-30', '2017-12-31'],
              dtype='datetime64[ns]', name='Date', length=4383, freq=None)
```
　　可以使用asfreq进行指定。如果数据中缺失了某个时间，asfreq将自动为这些时间添加新行，并默认分配空值。
```
opsd = opsd.asfreq('D')
```
***Output:***
```
DatetimeIndex(['2006-01-01', '2006-01-02', '2006-01-03', '2006-01-04',
               '2006-01-05', '2006-01-06', '2006-01-07', '2006-01-08',
               '2006-01-09', '2006-01-10',
               ...
               '2017-12-22', '2017-12-23', '2017-12-24', '2017-12-25',
               '2017-12-26', '2017-12-27', '2017-12-28', '2017-12-29',
               '2017-12-30', '2017-12-31'],
              dtype='datetime64[ns]', name='Date', length=4383, freq='D')
```
### 基于时间索引筛选数据
　　对于时间数据索引，可以使用loc提取数据。例如，查找2017年8月10日的数据。
```
opsd.loc['2017-08-10']
```
　　也可以选择一段时间，例如2014年1月20日至2014年1月22日的数据。与使用loc的常规索引一样，切片将包含两个端点。
```
opsd.loc['2014-01-20':'2014-01-22']
```
　　可以不具体到日，而仅仅指定对应的年和月，将返回当月的所有数据。例如，查找2017年1月份的数据。
```
opsd.loc['2017-01']
```
　　获取时间范围内的数据也可以使用truncate进行筛选。before将删去给定日期之前的数据，after将删去给定日期之后的数据。例如，筛选2017年1月份的数据。
```
opsd.truncate(before='2017-01-01',after='2017-01-31')
```
### 时间数据基本操作
　　针对时间数据，可以使用year，month，weekday等多种方法获取对应时间的年份、月份和星期。  
　　首先使用index提取数据的索引。
```
opsdtime = opsd.index
```
***Output:***
```
DatetimeIndex(['2006-01-01', '2006-01-02', '2006-01-03', '2006-01-04',
               '2006-01-05', '2006-01-06', '2006-01-07', '2006-01-08',
               '2006-01-09', '2006-01-10',
               ...
               '2017-12-22', '2017-12-23', '2017-12-24', '2017-12-25',
               '2017-12-26', '2017-12-27', '2017-12-28', '2017-12-29',
               '2017-12-30', '2017-12-31'],
              dtype='datetime64[ns]', name='Date', length=4383, freq='D')
```
　　使用year提取每个数据对应的年份。
``` 
opsdtime.year
```
***Output:***
```
Int64Index([2006, 2006, 2006, 2006, 2006, 2006, 2006, 2006, 2006, 2006,
            ...
            2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017],
           dtype='int64', name='Date', length=4383)
```
　　使用month提取月份。
```
opsdtime.month
```
***Output:***
```
Int64Index([ 1,  1,  1,  1,  1,  1,  1,  1,  1,  1,
            ...
            12, 12, 12, 12, 12, 12, 12, 12, 12, 12],
           dtype='int64', name='Date', length=4383)
```
　　month返回的是对应月份的数字，若想要获得月份的名字可以使用month_name。
```
opsdtime.month_name()
```
***Output:***
```
Index(['January', 'January', 'January', 'January', 'January', 'January',
       'January', 'January', 'January', 'January',
       ...
       'December', 'December', 'December', 'December', 'December', 'December',
       'December', 'December', 'December', 'December'],
      dtype='object', name='Date', length=4383)
```
　　可以使用weekday和weekday_name（新版本API已修改为`day_name()`）查看日期是星期几。
```
opsdtime.weekday
```
***Output:***
```
Int64Index([6, 0, 1, 2, 3, 4, 5, 6, 0, 1,
            ...
            4, 5, 6, 0, 1, 2, 3, 4, 5, 6],
           dtype='int64', name='Date', length=4383)
```
```
opsdtime.weekday_name
# opsdtime.day_name()
```
***Output:***
```
Index(['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday',
       'Saturday', 'Sunday', 'Monday', 'Tuesday',
       ...
       'Friday', 'Saturday', 'Sunday', 'Monday', 'Tuesday', 'Wednesday',
       'Thursday', 'Friday', 'Saturday', 'Sunday'],
      dtype='object', name='Date', length=4383)
```
　　构建月份与对应季节间的映射字典。
```
seasons = [1, 1, 2, 2, 2, 3, 3, 3, 4, 4, 4, 1]
month_to_season = dict(zip(range(1,13), seasons))

opsdtime.month.map(month_to_season)
```
***Output:***
```
{1: 1, 2: 1, 3: 2, 4: 2, 5: 2, 6: 3, 7: 3, 8: 3, 9: 4, 10: 4, 11: 4, 12: 1}

Int64Index([1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
            ...
            1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
           dtype='int64', name='Date', length=4383)
```

### 周期性分析
#### 重采样分析周期性
　　使用plot查看数据整体情况，电力消耗总量：
```
opsd['Consumption'].plot(figsize=(12,6))
```
　　![电力消耗整体情况](https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/BDA%E5%A4%8D%E4%B9%A0/0x7fbc523cae48.png)
　　具体查看2007年的数据。
```
opsd.loc['2007','Consumption'].plot(figsize=(12,6))
```
　　![2007年电力消耗情况](https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/BDA%E5%A4%8D%E4%B9%A0/0x7fbc5007e668.png)
　　使用groupby按变量season分组，并计算每个季节的用电量均值。
```
opsd.groupby('season')['Consumption'].mean().plot()
```
　　![按season分组均值](https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/BDA%E5%A4%8D%E4%B9%A0/0x7fbc4ff8acc0.png)
　　使用groupby进行重采样，将数据按是星期几进行分组，并计算每组的用电量均值。这里使用lambda函数传入weekday进行分组。
```
opsd.groupby(lambda x:x.weekday)['Consumption'].mean().plot()
```
　　![按dayofweek分组均值](https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/BDA%E5%A4%8D%E4%B9%A0/0x7fbc4ff1a5f8.png)
　　使用resample对风能发电数据进行降采样。按每个月重采样，并计算每月的均值。
```
wind = opsd['Wind'].resample('M').mean()
wind.plot(figsize=(12,6))
```
　　![按月重采样均值](https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/BDA%E5%A4%8D%E4%B9%A0/0x7fbc4e11ae48.png)

#### 数据差分分析周期性
　　在分析周期性的过程中，很重要的一点就是要消除数据的趋势性，常见的消除数据趋势的方法就是差分：计算连续数据点间的差异（这里特指一阶差分）。例如，t时刻的差分值：$\Delta d_t=d_t - d_{t-1}$。可以使用diff方法实现差分操作。   
　　例如，计算太阳能发电的差分序列并绘图：
```
opsd['Solar'].diff().plot(figsize=(12,6))
```
　　![一阶差分序列](https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/BDA%E5%A4%8D%E4%B9%A0/0x7fbc4e0e3fd0.png)
　　也可以通过移动时间序列自行计算差分值。  
　　移动序列可以使用shift方法。shift方法可以沿着时间轴将数据前移或后移，保持索引不变。
```
opsd['Solar'].tail()
```
***Output:***
```
Date
2017-12-27    16.530
2017-12-28    14.162
2017-12-29    29.854
2017-12-30     7.467
2017-12-31    19.980
Freq: D, Name: Solar, dtype: float64
```
```
opsd['Solar'].shift(1).tail()
```
***Output:***
```
Date
2017-12-27    30.923
2017-12-28    16.530
2017-12-29    14.162
2017-12-30    29.854
2017-12-31     7.467
Freq: D, Name: Solar, dtype: float64
```
　　两个序列相减即可得到原始数据的一阶差分序列。
```
dif = opsd['Solar']-opsd['Solar'].shift(1)
dif.plot(figsize=(12,6))
```
　　也可以通过设定shift方法中的参数freq移动索引而数据保持不变，例如指定时间移动一天。
```
opsd['Solar'].shift(1,freq='d').tail()
```
***Output:***
```
Date
2017-12-28    16.530
2017-12-29    14.162
2017-12-30    29.854
2017-12-31     7.467
2018-01-01    19.980
Freq: D, Name: Solar, dtype: float64
```
### 滚动窗口
　　与降采样类似，滚动窗口将数据拆分为时间窗口，并且对每个窗口中的数据使用诸如mean，median等函数进行聚合。但是，与降采样不同，滚动窗口以与数据相同的频率重叠和“滚动”，因此变换的时间序列与原始时间序列的频率相同。  
　　例如，设定窗口为7天，且以数据中心为基准点，则每一个数据对应的窗口将包含前面三天与后面三天。具体来看，2017-07-06对应的窗口就是2017-07-03到2017-07-09。  
```
opsd['Wind'].rolling(7).mean().plot(figsize=(12,6))
```
　　![7天滑窗均值](https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/BDA%E5%A4%8D%E4%B9%A0/0x7fbc4fcae400.png)

```
opsd['Wind'].rolling(30).mean().plot(figsize=(12,6))
```
　　当窗口范围中存在缺失值时，窗口将会返回为缺失值，可以设定min_periods为360，只需要对应窗口中有360个以上数据就可以，这样可以容忍一小部分的缺失数据。
```
opsd['Wind'].rolling(window=365,min_periods=360).mean().plot(figsize=(12,6))
```
　　![365天滑窗均值](https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/BDA%E5%A4%8D%E4%B9%A0/0x7fbc4ddfe080.png)

## K-means clustering algorithm
　　一个简单的算法伪代码描述如下：
$$
\begin{aligned} 
\hline
&1:\ 选择K个点作为初始质心。\\
&2:\ repeat \\
&3:\ \quad 将每个点指派到最近的质心，形成K个簇。\\
&4:\ \quad 重新计算每个簇的质心。\\
&5:\ until 质心不发生变化。\\
\hline
\end{aligned}
$$
　　相似度使用欧氏距离(Euclidean Distance)度量，给定两个样本$X=(x_1,x_2,...,x_n)$与$Y=(y_1,y_2,...,y_n)$，$X$和$Y$两个向量间的欧氏距离表示为：
$$
\begin{aligned} 
dist_{ed}(X,Y)=\Vert X-Y \Vert ^2=\sqrt[2]{(x_1-y_1)^2+...+(x_n-y_n)^2}
\end{aligned}
$$

### Python实现
　　距离计算函数*point_dist*。
```
def point_dist(x,c): 
    return np.linalg.norm(x-c)
```
#### iterrows 遍历方式实现
```
def k_means(X,k):
    centers = X.sample(k).values #从数据集随机选择 K 个样本作为初始化的类中心，k 行 d 列
    X_labels = np.zeros(len(X)) #样本的类别
    error = 10e10
    while(error > 1e-6):
        for i,x in X.iterrows():#指派样本类标签
            X_labels[i] = np.argmin([point_dist(x,centers[i,:]) for i in range(k)])
        centers_pre = centers
        centers = X.groupby(X_labels).mean().values #更新样本均值，即类中心
        error = np.linalg.norm(centers_pre - centers)#计算error
    return X_labels, centers
```
#### apply 遍历方式实现
```
def k_means(X,k):
    #初始化 K 个中心，从原始数据中选择样本
    centers = X.sample(k).values
    X_labels = np.zeros(len(X)) #样本的类别
    error = 10e10
    while(error > 1e-6):
        X_labels = X.apply(lambda r : np.argmin([point_dist(r,centers[i,:]) for i in range(k)]),axis=1)
        centers_pre = centers
        centers = X.groupby(X_labels).mean().values #更新样本均值，即类中心
        error = np.linalg.norm(centers_pre - centers)#计算error
    return X_labels, centers
```
#### 矩阵运算方式实现
　　数据集表示成 $n \times d$ 矩阵 $\mathbf{X}$，其中 $n$ 为样本数量，$d$ 为样本的维度。 $k$ 个聚类中心表示成 $k \times d$ 矩阵 $\mathbf{C}$，$\mathbf{C}$ 每一行表示一个聚类中心。样本到 $k$ 个中心的距离表示成 $n \times k$ 矩阵 $\mathbf{D}$。  

　　已知聚类中心，计算样本到中心距离，并将样本划分到距离最小的类的流程如下图所示。

　　![样本列表计算流程](https://hexo-img-meurice.oss-cn-beijing.aliyuncs.com/BDA%E5%A4%8D%E4%B9%A0/%E6%A0%B7%E6%9C%AC%E5%88%97%E8%A1%A8%E8%AE%A1%E7%AE%97%E6%B5%81%E7%A8%8B.jpg)

　　使用 Numpy 实现上述计算流程的代码为：

```
for i in range(k):
	D[:,i] = np.sqrt(np.sum(np.square(X - C[i,:]),axis=1))
labels = np.argmin(D,axis=1)
```
　　得到样本的类标签后，聚类中心的更新流程为：1）根据类标签对样本进行分组；2）将聚类中心更新为每一组样本的均值。Python 实现的代码为：

```
C = X.groupby(labels).mean().values
```

　　完整实现代码：
```
def k_means(X,k):
    C = X.sample(k).values  #从数据集随机选择 K 个样本作为初始化的类中心，k 行 d 列
    X_labels = np.zeros(len(X)) #记录样本的类别
    error = 10e10 #停止迭代的阈值
    while(error > 1e-6):
        D = np.zeros((len(X),k)) #样本到每一个中心的距离，n 行 k 列
        for i in range(k):
            D[:,i] = np.sqrt(np.sum(np.square(X - C[i,:]),axis=1))
        labels = np.argmin(D,axis=1)
        C_pre = C
        
        temp_C = X.groupby(labels).mean() #更新样本均值，即类中心
        C = np.zeros((k,X.shape[1]))
        for i in temp_C.index:
            C[i,:] = temp_C.loc[i,:].values
            
        if C.shape == C_pre.shape:
            error = np.linalg.norm(C_pre - C)#计算error
        else:
            print(C.shape, C_pre.shape)
    return labels, C
```

## Logistic regression
### 线性回归
$$
\begin{aligned}
h_\theta(x)&=\theta^Tx \\
 &=\sum_{i=0}^n{\theta_i x_i} \\
 &=\theta_0 + \theta_1 x_1 + \theta_2 x_2 + ... + \theta_n x_n
\end{aligned}
$$

### sigmoid
$$
\begin{aligned}
g(z)=\frac{1}{1+e^{-z}}
\end{aligned}
$$

### 逻辑回归公式
　　线性回归公式带入Sigmoid即得：
$$
\begin{aligned}
h_\theta(x)=\frac{1}{1+e^{-{\theta^Tx}}}
\end{aligned}
$$

### 损失函数
　　对数形式的似然函数如下：  
$$
\begin{aligned}
logL(\theta)=\sum_{i=1}^n{log(p(x_i;\theta))} 
\end{aligned}
$$
　　用sigmoid函数表示0-1中取1的概率，损失函数可以定义为：
$$
\begin{aligned}
y&=0时，Cost(h_\theta(x),y)=-log(1-h_\theta(x)) \\
y&=1时，Cost(h_\theta(x),y)=-log(h_\theta(x)) 
\end{aligned}
$$
　　损失函数的要求是预测结果与真实结果越相近，函数值越小，故在前面加上负号。取对数和上面提到的最大似然函数有关，不影响原函数的单调性，且会放大概率之间的差异，更好的区分各个样本的类别。  
　　故逻辑回归的损失函数如下：
$$
\begin{aligned}
J(\theta)=-\frac{1}{m}\sum_{i=1}^m{[y^{(i)}logh_\theta(x^{(i)})+(1-y^{(i)})log(1-h_\theta(x^{(i)}))]}
\end{aligned}
$$

### 梯度下降
　　要求出最优参数$\theta$，需要最小化$J(\theta)$，更新参数：
$$
\begin{aligned}
\theta_j := \theta_j-\alpha \frac{\partial J(\theta)}{\partial \theta_j}
\end{aligned}
$$
　　sigmoid函数求导：
$$
\begin{aligned}
\frac{\partial g(z)}{\partial z}=g(z)(1-g(z))
\end{aligned}
$$
　　对g(\theta^Tx)求导：
$$
\begin{aligned}
\frac{\partial g(\theta^Tx)}{\partial z}=g(\theta^Tx)(1-g(\theta^Tx))x_j^{(i)}
\end{aligned}
$$
　　**损失函数求导：**
$$
\begin{aligned}
\frac{\partial J(\theta)}{\partial \theta_j}&= ...\\
&=\frac{1}{m}\sum_{i}^m(h_\theta(x^{(i)})-y^{(i)})x_j^{(i)}
\end{aligned}
$$
　　得到逻辑回归的梯度下降更新公式：
$$
\begin{aligned}
\theta_j := \theta_j-\alpha  \frac{1}{m}\sum_{i}^m(h_\theta(x^{(i)})-y^{(i)})x_j^{(i)}
\end{aligned}
$$
### 伪代码描述
$$
\begin{aligned} 
\hline
&1:\ 初始化回归系数。\\
&2:\ repeat \\
&3:\ \quad 计算梯度\frac{\partial J(\theta)}{\partial \theta}。\\
&4:\ \quad \theta:=\theta+\alpha * \frac{\partial J(\theta)}{\partial \theta}。\\
&5:\ until 收敛 \  or \  max\_loop。\\
&6:\ return\ \theta \\
\hline
\end{aligned}
$$

### Python实现
```
class LR:
    def __init__(self, alpha=0.01, max_iter=100):
        self.alpha = alpha
        self.max_iter = max_iter
        
    def fit(self, X, y):
        X = np.mat(X)  # (rows,cols)
        m, n = np.shape(X)
        y = np.mat(y).T  # (rows,cols)

        self.weight = np.ones((n, 1))

        for i in range(self.max_iter):
            h = self.sigmoid(X * self.weight)
            error = y - h
            self.weight = self.weight + self.alpha * X.T * error
```r
```
