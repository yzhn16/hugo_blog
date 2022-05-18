# Pyecharts数据可视化（以ZBG财经求职力挑战赛为例）


Python + Echarts = pyecharts，记一次经济管理类的案例分析比赛。

<!--more-->

## 赛题任务
　　**题目1：**  
　　分析小米公司手机业务目前所处的宏观经营环境以及产业环境。  
　　**题目2：**  
　　为小米公司制定未来三年手机业务发展策略，帮助小米公司：  
　　　－　维持在低端手机市场的占有率  
　　　－　突破高端市场，实现更高的高端手机出货量
## pyecharts自适应显示
1. 获取浏览器窗口长宽  
2. 设置显示区域长款百分比  
3. 设置居中参数  
4. 设置自适应（根据显示区域调整刷新）  

　　**step1.** 修改.\Lib\site-packages\pyecharts\render\templates\ **macro**  
```
{%- macro render_chart_content(c) -%}
    <div id="{{ c.chart_id }}" class="chart-container" style="width:95%; height:95%; margin:auto; top:30px"></div>
    <script>
        var chart_{{ c.chart_id }} = echarts.init(
            document.getElementById('{{ c.chart_id }}'), '{{ c.theme }}', {renderer: '{{ c.renderer }}'});
        {% for js in c.js_functions.items %}
            {{ js }}
        {% endfor %}
        var option_{{ c.chart_id }} = {{ c.json_contents }};
        chart_{{ c.chart_id }}.setOption(option_{{ c.chart_id }});
        {% if c._is_geo_chart %}
            var bmap = chart_{{ c.chart_id }}.getModel().getComponent('bmap').getBMap();
            {% if c.bmap_js_functions %}
                {% for fn in c.bmap_js_functions.items %}
                    {{ fn }}
                {% endfor %}
            {% endif %}
        {% endif %}
        {% if c.width.endswith('%') %}
            window.addEventListener('resize', function(){
                chart_{{ c.chart_id }}.resize();
            })
        {% endif %}
    </script>
    <script>   
    var x=window.innerWidth;
    function resizeFresh(){
        if(x!=window.innerWidth)
            location.reload();
    }
    </script>
{%- endmacro %}
```
　　**step2.** 修改.\Lib\site-packages\pyecharts\render\templates\ **simple_chart.html**  
```
{% import 'macro' as macro %}
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>{{ chart.page_title }}</title>
    <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,minimum-scale=1.0,user-scalable=yes">
    {{ macro.render_chart_dependencies(chart) }}
    <style type="text/css">
        html,body{
            height:100%;
            width:100%
    }
    </style>
</head>
<body onresize="resizeFresh()">
    {{ macro.render_chart_content(chart) }}
</body>
</html>

```

## 可视化分析\[pyecharts]

　　pyecharts相关配置：
```Python
!jupyter trust Visualization.ipynb

import warnings
warnings.filterwarnings('ignore')

from pyecharts.globals import CurrentConfig, NotebookType, OnlineHostType
#http://127.0.0.1:8000/assets/

CurrentConfig.ONLINE_HOST = 'https://assets.pyecharts.org/assets/'
CurrentConfig.NOTEBOOK_TYPE = NotebookType.JUPYTER_LAB

CurrentConfig.ONLINE_HOST
```

### 全球手机出货量/市场份额数据可视化

 　　剔除部分有缺失的数据：
```Python
GSMS = pd.read_csv('Global_Smartphone_Market_Share.csv').iloc[:,0:5]

print('deleted:',set(GSMS.iloc[np.where(GSMS.isnull())[0],0].tolist()))
GSMS = GSMS[~GSMS.Vendor.isin(GSMS.iloc[np.where(GSMS.isnull())[0],0].tolist())]

display(GSMS)
```
　　deleted: {'Lenovo', 'Realme', 'LG'}  

#### 2018Q1 - 2021Q1 全球智能手机出货量/市场份额

##### 出货量（按年份）柱状图

```Python
from pyecharts.charts import Bar

grouped_df_year = GSMS[GSMS.Year < 2021].groupby(['Vendor','Year']).agg({'Shipment_Volumes':'sum','Market_Share':'sum'}).reset_index()
grouped_df_year.Shipment_Volumes = round(grouped_df_year.Shipment_Volumes,1)

bar = Bar(init_opts=opts.InitOpts(theme=ThemeType.LIGHT,\
                                  width='1100px',\
                                  height='500px'))\
            .add_xaxis(grouped_df_year.Vendor.unique().tolist())\
            .set_global_opts(title_opts=opts.TitleOpts(title='2018-2020全球年出货量'),\
                             legend_opts=opts.LegendOpts(type_='plain',orient='horizontal'))

for y in grouped_df_year.Year.unique():
    bar = bar.add_yaxis(str(y) + '年全球出货量（百万）',grouped_df_year[grouped_df_year.Year == y].Shipment_Volumes.tolist())
    
bar.render_notebook()
```
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Awesome-pyecharts</title>
    <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,minimum-scale=1.0,user-scalable=yes">
            <script type="text/javascript" src="../../pyecharts-assets-master/assets/echarts.min.js"></script>
    <style type="text/css">
        html,body{
            height:100%;
            width:100%
    }
    </style>
</head>
<body onresize="resizeFresh()">
    <div id="aaf3926ed4944c62aeb73f2b1424ec8e" class="chart-container" style="width:95%; height:395%;margin:auto; top:0px;"></div>
    <script>
        var chart_aaf3926ed4944c62aeb73f2b1424ec8e = echarts.init(
            document.getElementById('aaf3926ed4944c62aeb73f2b1424ec8e'), 'light', {renderer: 'canvas'});
        var option_aaf3926ed4944c62aeb73f2b1424ec8e = {
    "animation": true,
    "animationThreshold": 2000,
    "animationDuration": 1000,
    "animationEasing": "cubicOut",
    "animationDelay": 0,
    "animationDurationUpdate": 300,
    "animationEasingUpdate": "cubicOut",
    "animationDelayUpdate": 0,
    "series": [
        {
            "type": "bar",
            "name": "2018\u5e74\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09",
            "legendHoverLink": true,
            "data": [
                206.3,
                205.2,
                119.0,
                291.8,
                102.4,
                119.0
            ],
            "showBackground": false,
            "barMinHeight": 0,
            "barCategoryGap": "20%",
            "barGap": "30%",
            "large": false,
            "largeThreshold": 400,
            "seriesLayoutBy": "column",
            "datasetIndex": 0,
            "clip": true,
            "zlevel": 0,
            "z": 2,
            "label": {
                "show": true,
                "position": "top",
                "margin": 8
            }
        },
        {
            "type": "bar",
            "name": "2019\u5e74\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09",
            "legendHoverLink": true,
            "data": [
                195.6,
                238.7,
                120.0,
                296.9,
                113.7,
                124.7
            ],
            "showBackground": false,
            "barMinHeight": 0,
            "barCategoryGap": "20%",
            "barGap": "30%",
            "large": false,
            "largeThreshold": 400,
            "seriesLayoutBy": "column",
            "datasetIndex": 0,
            "clip": true,
            "zlevel": 0,
            "z": 2,
            "label": {
                "show": true,
                "position": "top",
                "margin": 8
            }
        },
        {
            "type": "bar",
            "name": "2020\u5e74\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09",
            "legendHoverLink": true,
            "data": [
                201.1,
                187.7,
                111.8,
                255.7,
                108.5,
                145.4
            ],
            "showBackground": false,
            "barMinHeight": 0,
            "barCategoryGap": "20%",
            "barGap": "30%",
            "large": false,
            "largeThreshold": 400,
            "seriesLayoutBy": "column",
            "datasetIndex": 0,
            "clip": true,
            "zlevel": 0,
            "z": 2,
            "label": {
                "show": true,
                "position": "top",
                "margin": 8
            }
        }
    ],
    "legend": [
        {
            "data": [
                "2018\u5e74\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09",
                "2019\u5e74\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09",
                "2020\u5e74\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09"
            ],
            "selected": {
                "2018\u5e74\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09": true,
                "2019\u5e74\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09": true,
                "2020\u5e74\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09": true
            },
            "type": "plain",
            "show": true,
            "orient": "horizontal",
            "padding": 5,
            "itemGap": 10,
            "itemWidth": 25,
            "itemHeight": 14
        }
    ],
    "tooltip": {
        "show": true,
        "trigger": "item",
        "triggerOn": "mousemove|click",
        "axisPointer": {
            "type": "line"
        },
        "showContent": true,
        "alwaysShowContent": false,
        "showDelay": 0,
        "hideDelay": 100,
        "textStyle": {
            "fontSize": 14
        },
        "borderWidth": 0,
        "padding": 5
    },
    "xAxis": [
        {
            "show": true,
            "scale": false,
            "nameLocation": "end",
            "nameGap": 15,
            "gridIndex": 0,
            "inverse": false,
            "offset": 0,
            "splitNumber": 5,
            "minInterval": 0,
            "splitLine": {
                "show": false,
                "lineStyle": {
                    "show": true,
                    "width": 1,
                    "opacity": 1,
                    "curveness": 0,
                    "type": "solid"
                }
            },
            "data": [
                "Apple",
                "Huawei",
                "Oppo",
                "Samsung",
                "Vivo",
                "Xiaomi"
            ]
        }
    ],
    "yAxis": [
        {
            "show": true,
            "scale": false,
            "nameLocation": "end",
            "nameGap": 15,
            "gridIndex": 0,
            "inverse": false,
            "offset": 0,
            "splitNumber": 5,
            "minInterval": 0,
            "splitLine": {
                "show": false,
                "lineStyle": {
                    "show": true,
                    "width": 1,
                    "opacity": 1,
                    "curveness": 0,
                    "type": "solid"
                }
            }
        }
    ],
    "title": [
        {
            "padding": 5,
            "itemGap": 10
        }
    ]
};
        chart_aaf3926ed4944c62aeb73f2b1424ec8e.setOption(option_aaf3926ed4944c62aeb73f2b1424ec8e);
    </script>
    <script>   
    var x=window.innerWidth;
    function resizeFresh(){
        if(x!=window.innerWidth)
            location.reload();
    }
    </script>
</body>
</html>  

##### 出货量（按季度）折线图

```Python
from pyecharts.charts import Line

line = Line(init_opts=opts.InitOpts(theme=ThemeType.LIGHT,
                                  width='1200px',
                                  height='550px'))\
            .add_xaxis((GSMS.Year.astype(str) + GSMS.Quarterly.astype(str)).drop_duplicates(keep='first').tolist())\
            .set_global_opts(title_opts=opts.TitleOpts(title='2018Q1-2021Q1\n季度全球出货量'),\
                             legend_opts=opts.LegendOpts(type_='scroll',pos_top=10,pos_left = 100, orient='horizontal'))#horizontal vertical


for v in grouped_df_year.Vendor.unique():
    line = line.add_yaxis(str(v) + '季度全球出货量（百万）',GSMS[GSMS.Vendor == v].Shipment_Volumes.tolist())

line.render_notebook()
```
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Awesome-pyecharts</title>
    <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,minimum-scale=1.0,user-scalable=yes">
            <script type="text/javascript" src="../../pyecharts-assets-master/assets/echarts.min.js"></script>
    <style type="text/css">
        html,body{
            height:100%;
            width:100%
    }
    </style>
</head>
<body onresize="resizeFresh()">
    <div id="1c4ca79d15c54b47abf178ba0084ad5b" class="chart-container" style="width:95%; height:495%; margin:auto; top:0px"></div>
    <script>
        var chart_1c4ca79d15c54b47abf178ba0084ad5b = echarts.init(
            document.getElementById('1c4ca79d15c54b47abf178ba0084ad5b'), 'light', {renderer: 'canvas'});
        var option_1c4ca79d15c54b47abf178ba0084ad5b = {
    "animation": true,
    "animationThreshold": 2000,
    "animationDuration": 1000,
    "animationEasing": "cubicOut",
    "animationDelay": 0,
    "animationDurationUpdate": 300,
    "animationEasingUpdate": "cubicOut",
    "animationDelayUpdate": 0,
    "series": [
        {
            "type": "line",
            "name": "Apple\u5b63\u5ea6\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09",
            "connectNulls": false,
            "symbolSize": 4,
            "showSymbol": true,
            "smooth": false,
            "clip": true,
            "step": false,
            "data": [
                [
                    "2018Q1",
                    52.2
                ],
                [
                    "2018Q2",
                    41.3
                ],
                [
                    "2018Q3",
                    46.9
                ],
                [
                    "2018Q4",
                    65.9
                ],
                [
                    "2019Q1",
                    42.0
                ],
                [
                    "2019Q2",
                    36.5
                ],
                [
                    "2019Q3",
                    44.8
                ],
                [
                    "2019Q4",
                    72.3
                ],
                [
                    "2020Q1",
                    40.0
                ],
                [
                    "2020Q2",
                    37.5
                ],
                [
                    "2020Q3",
                    41.7
                ],
                [
                    "2020Q4",
                    81.9
                ],
                [
                    "2021Q1",
                    59.5
                ]
            ],
            "hoverAnimation": true,
            "label": {
                "show": true,
                "position": "top",
                "margin": 8
            },
            "lineStyle": {
                "show": true,
                "width": 1,
                "opacity": 1,
                "curveness": 0,
                "type": "solid"
            },
            "areaStyle": {
                "opacity": 0
            },
            "zlevel": 0,
            "z": 0
        },
        {
            "type": "line",
            "name": "Huawei\u5b63\u5ea6\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09",
            "connectNulls": false,
            "symbolSize": 4,
            "showSymbol": true,
            "smooth": false,
            "clip": true,
            "step": false,
            "data": [
                [
                    "2018Q1",
                    39.3
                ],
                [
                    "2018Q2",
                    54.2
                ],
                [
                    "2018Q3",
                    52.0
                ],
                [
                    "2018Q4",
                    59.7
                ],
                [
                    "2019Q1",
                    59.1
                ],
                [
                    "2019Q2",
                    56.6
                ],
                [
                    "2019Q3",
                    66.8
                ],
                [
                    "2019Q4",
                    56.2
                ],
                [
                    "2020Q1",
                    49.0
                ],
                [
                    "2020Q2",
                    54.8
                ],
                [
                    "2020Q3",
                    50.9
                ],
                [
                    "2020Q4",
                    33.0
                ],
                [
                    "2021Q1",
                    15.0
                ]
            ],
            "hoverAnimation": true,
            "label": {
                "show": true,
                "position": "top",
                "margin": 8
            },
            "lineStyle": {
                "show": true,
                "width": 1,
                "opacity": 1,
                "curveness": 0,
                "type": "solid"
            },
            "areaStyle": {
                "opacity": 0
            },
            "zlevel": 0,
            "z": 0
        },
        {
            "type": "line",
            "name": "Oppo\u5b63\u5ea6\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09",
            "connectNulls": false,
            "symbolSize": 4,
            "showSymbol": true,
            "smooth": false,
            "clip": true,
            "step": false,
            "data": [
                [
                    "2018Q1",
                    24.2
                ],
                [
                    "2018Q2",
                    29.6
                ],
                [
                    "2018Q3",
                    33.9
                ],
                [
                    "2018Q4",
                    31.3
                ],
                [
                    "2019Q1",
                    25.7
                ],
                [
                    "2019Q2",
                    30.6
                ],
                [
                    "2019Q3",
                    32.3
                ],
                [
                    "2019Q4",
                    31.4
                ],
                [
                    "2020Q1",
                    22.3
                ],
                [
                    "2020Q2",
                    24.5
                ],
                [
                    "2020Q3",
                    31.0
                ],
                [
                    "2020Q4",
                    34.0
                ],
                [
                    "2021Q1",
                    38.0
                ]
            ],
            "hoverAnimation": true,
            "label": {
                "show": true,
                "position": "top",
                "margin": 8
            },
            "lineStyle": {
                "show": true,
                "width": 1,
                "opacity": 1,
                "curveness": 0,
                "type": "solid"
            },
            "areaStyle": {
                "opacity": 0
            },
            "zlevel": 0,
            "z": 0
        },
        {
            "type": "line",
            "name": "Samsung\u5b63\u5ea6\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09",
            "connectNulls": false,
            "symbolSize": 4,
            "showSymbol": true,
            "smooth": false,
            "clip": true,
            "step": false,
            "data": [
                [
                    "2018Q1",
                    78.2
                ],
                [
                    "2018Q2",
                    71.5
                ],
                [
                    "2018Q3",
                    72.3
                ],
                [
                    "2018Q4",
                    69.8
                ],
                [
                    "2019Q1",
                    72.0
                ],
                [
                    "2019Q2",
                    76.3
                ],
                [
                    "2019Q3",
                    78.2
                ],
                [
                    "2019Q4",
                    70.4
                ],
                [
                    "2020Q1",
                    58.6
                ],
                [
                    "2020Q2",
                    54.2
                ],
                [
                    "2020Q3",
                    80.4
                ],
                [
                    "2020Q4",
                    62.5
                ],
                [
                    "2021Q1",
                    76.6
                ]
            ],
            "hoverAnimation": true,
            "label": {
                "show": true,
                "position": "top",
                "margin": 8
            },
            "lineStyle": {
                "show": true,
                "width": 1,
                "opacity": 1,
                "curveness": 0,
                "type": "solid"
            },
            "areaStyle": {
                "opacity": 0
            },
            "zlevel": 0,
            "z": 0
        },
        {
            "type": "line",
            "name": "Vivo\u5b63\u5ea6\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09",
            "connectNulls": false,
            "symbolSize": 4,
            "showSymbol": true,
            "smooth": false,
            "clip": true,
            "step": false,
            "data": [
                [
                    "2018Q1",
                    18.9
                ],
                [
                    "2018Q2",
                    26.5
                ],
                [
                    "2018Q3",
                    30.5
                ],
                [
                    "2018Q4",
                    26.5
                ],
                [
                    "2019Q1",
                    23.9
                ],
                [
                    "2019Q2",
                    27.0
                ],
                [
                    "2019Q3",
                    31.3
                ],
                [
                    "2019Q4",
                    31.5
                ],
                [
                    "2020Q1",
                    21.6
                ],
                [
                    "2020Q2",
                    22.5
                ],
                [
                    "2020Q3",
                    31.0
                ],
                [
                    "2020Q4",
                    33.4
                ],
                [
                    "2021Q1",
                    35.5
                ]
            ],
            "hoverAnimation": true,
            "label": {
                "show": true,
                "position": "top",
                "margin": 8
            },
            "lineStyle": {
                "show": true,
                "width": 1,
                "opacity": 1,
                "curveness": 0,
                "type": "solid"
            },
            "areaStyle": {
                "opacity": 0
            },
            "zlevel": 0,
            "z": 0
        },
        {
            "type": "line",
            "name": "Xiaomi\u5b63\u5ea6\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09",
            "connectNulls": false,
            "symbolSize": 4,
            "showSymbol": true,
            "smooth": false,
            "clip": true,
            "step": false,
            "data": [
                [
                    "2018Q1",
                    28.1
                ],
                [
                    "2018Q2",
                    32.0
                ],
                [
                    "2018Q3",
                    33.3
                ],
                [
                    "2018Q4",
                    25.6
                ],
                [
                    "2019Q1",
                    27.8
                ],
                [
                    "2019Q2",
                    32.3
                ],
                [
                    "2019Q3",
                    31.7
                ],
                [
                    "2019Q4",
                    32.9
                ],
                [
                    "2020Q1",
                    29.7
                ],
                [
                    "2020Q2",
                    26.5
                ],
                [
                    "2020Q3",
                    46.2
                ],
                [
                    "2020Q4",
                    43.0
                ],
                [
                    "2021Q1",
                    48.5
                ]
            ],
            "hoverAnimation": true,
            "label": {
                "show": true,
                "position": "top",
                "margin": 8
            },
            "lineStyle": {
                "show": true,
                "width": 1,
                "opacity": 1,
                "curveness": 0,
                "type": "solid"
            },
            "areaStyle": {
                "opacity": 0
            },
            "zlevel": 0,
            "z": 0
        }
    ],
    "legend": [
        {
            "data": [
                "Apple\u5b63\u5ea6\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09",
                "Huawei\u5b63\u5ea6\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09",
                "Oppo\u5b63\u5ea6\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09",
                "Samsung\u5b63\u5ea6\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09",
                "Vivo\u5b63\u5ea6\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09",
                "Xiaomi\u5b63\u5ea6\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09"
            ],
            "selected": {
                "Apple\u5b63\u5ea6\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09": true,
                "Huawei\u5b63\u5ea6\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09": true,
                "Oppo\u5b63\u5ea6\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09": true,
                "Samsung\u5b63\u5ea6\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09": true,
                "Vivo\u5b63\u5ea6\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09": true,
                "Xiaomi\u5b63\u5ea6\u5168\u7403\u51fa\u8d27\u91cf\uff08\u767e\u4e07\uff09": true
            },
            "type": "scroll",
            "show": true,
            "top": 10,
            "orient": "horizontal",
            "padding": 5,
            "itemGap": 10,
            "itemWidth": 25,
            "itemHeight": 14
        }
    ],
    "tooltip": {
        "show": true,
        "trigger": "item",
        "triggerOn": "mousemove|click",
        "axisPointer": {
            "type": "line"
        },
        "showContent": true,
        "alwaysShowContent": false,
        "showDelay": 0,
        "hideDelay": 100,
        "textStyle": {
            "fontSize": 14
        },
        "borderWidth": 0,
        "padding": 5
    },
    "xAxis": [
        {
            "show": true,
            "scale": false,
            "nameLocation": "end",
            "nameGap": 15,
            "gridIndex": 0,
            "inverse": false,
            "offset": 0,
            "splitNumber": 5,
            "minInterval": 0,
            "splitLine": {
                "show": false,
                "lineStyle": {
                    "show": true,
                    "width": 1,
                    "opacity": 1,
                    "curveness": 0,
                    "type": "solid"
                }
            },
            "data": [
                "2018Q1",
                "2018Q2",
                "2018Q3",
                "2018Q4",
                "2019Q1",
                "2019Q2",
                "2019Q3",
                "2019Q4",
                "2020Q1",
                "2020Q2",
                "2020Q3",
                "2020Q4",
                "2021Q1"
            ]
        }
    ],
    "yAxis": [
        {
            "show": true,
            "scale": false,
            "nameLocation": "end",
            "nameGap": 15,
            "gridIndex": 0,
            "inverse": false,
            "offset": 0,
            "splitNumber": 5,
            "minInterval": 0,
            "splitLine": {
                "show": false,
                "lineStyle": {
                    "show": true,
                    "width": 1,
                    "opacity": 1,
                    "curveness": 0,
                    "type": "solid"
                }
            }
        }
    ],
    "title": [
        {
            "padding": 5,
            "itemGap": 10
        }
    ]
};
        chart_1c4ca79d15c54b47abf178ba0084ad5b.setOption(option_1c4ca79d15c54b47abf178ba0084ad5b);
    </script>
    <script>   
    var x=window.innerWidth;
    function resizeFresh(){
        if(x!=window.innerWidth)
            location.reload();
    }
    </script>
</body>
</html>


##### 市场份额（按季度）折线图

```Python
from pyecharts.charts import Line

line = Line(init_opts=opts.InitOpts(theme=ThemeType.LIGHT,
                                  width='1200px',
                                  height='550px'))\
            .add_xaxis((GSMS.Year.astype(str) + GSMS.Quarterly.astype(str)).drop_duplicates(keep='first').tolist())\
            .set_global_opts(title_opts=opts.TitleOpts(title='2018Q1-2021Q1\n季度全球份额'),\
                             legend_opts=opts.LegendOpts(type_='scroll',pos_top=10,pos_left = 210, orient='horizontal'))#horizontal vertical


for v in grouped_df_year.Vendor.unique():
    line = line.add_yaxis(str(v) + '季度全球份额（%）',np.round(100 * GSMS[GSMS.Vendor == v].Market_Share, 0).tolist())

line.render_notebook()
```
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Awesome-pyecharts</title>
    <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,minimum-scale=1.0,user-scalable=yes">
            <script type="text/javascript" src="../../pyecharts-assets-master/assets/echarts.min.js"></script>
    <style type="text/css">
        html,body{
            height:100%;
            width:100%
    }
    </style>
</head>
<body onresize="resizeFresh()">
    <div id="1a350fedf1644c1b804480e2c16bbd63" class="chart-container" style="width:95%; height:495%; margin:auto; top:0px"></div>
    <script>
        var chart_1a350fedf1644c1b804480e2c16bbd63 = echarts.init(
            document.getElementById('1a350fedf1644c1b804480e2c16bbd63'), 'light', {renderer: 'canvas'});
        var option_1a350fedf1644c1b804480e2c16bbd63 = {
    "animation": true,
    "animationThreshold": 2000,
    "animationDuration": 1000,
    "animationEasing": "cubicOut",
    "animationDelay": 0,
    "animationDurationUpdate": 300,
    "animationEasingUpdate": "cubicOut",
    "animationDelayUpdate": 0,
    "series": [
        {
            "type": "line",
            "name": "Apple\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09",
            "connectNulls": false,
            "symbolSize": 4,
            "showSymbol": true,
            "smooth": false,
            "clip": true,
            "step": false,
            "data": [
                [
                    "2018Q1",
                    14.0
                ],
                [
                    "2018Q2",
                    11.0
                ],
                [
                    "2018Q3",
                    12.0
                ],
                [
                    "2018Q4",
                    17.0
                ],
                [
                    "2019Q1",
                    12.0
                ],
                [
                    "2019Q2",
                    10.0
                ],
                [
                    "2019Q3",
                    12.0
                ],
                [
                    "2019Q4",
                    18.0
                ],
                [
                    "2020Q1",
                    14.0
                ],
                [
                    "2020Q2",
                    14.0
                ],
                [
                    "2020Q3",
                    11.0
                ],
                [
                    "2020Q4",
                    21.0
                ],
                [
                    "2021Q1",
                    17.0
                ]
            ],
            "hoverAnimation": true,
            "label": {
                "show": true,
                "position": "top",
                "margin": 8
            },
            "lineStyle": {
                "show": true,
                "width": 1,
                "opacity": 1,
                "curveness": 0,
                "type": "solid"
            },
            "areaStyle": {
                "opacity": 0
            },
            "zlevel": 0,
            "z": 0
        },
        {
            "type": "line",
            "name": "Huawei\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09",
            "connectNulls": false,
            "symbolSize": 4,
            "showSymbol": true,
            "smooth": false,
            "clip": true,
            "step": false,
            "data": [
                [
                    "2018Q1",
                    11.0
                ],
                [
                    "2018Q2",
                    15.0
                ],
                [
                    "2018Q3",
                    14.0
                ],
                [
                    "2018Q4",
                    15.0
                ],
                [
                    "2019Q1",
                    17.0
                ],
                [
                    "2019Q2",
                    16.0
                ],
                [
                    "2019Q3",
                    18.0
                ],
                [
                    "2019Q4",
                    14.0
                ],
                [
                    "2020Q1",
                    17.0
                ],
                [
                    "2020Q2",
                    20.0
                ],
                [
                    "2020Q3",
                    14.0
                ],
                [
                    "2020Q4",
                    8.0
                ],
                [
                    "2021Q1",
                    4.0
                ]
            ],
            "hoverAnimation": true,
            "label": {
                "show": true,
                "position": "top",
                "margin": 8
            },
            "lineStyle": {
                "show": true,
                "width": 1,
                "opacity": 1,
                "curveness": 0,
                "type": "solid"
            },
            "areaStyle": {
                "opacity": 0
            },
            "zlevel": 0,
            "z": 0
        },
        {
            "type": "line",
            "name": "Oppo\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09",
            "connectNulls": false,
            "symbolSize": 4,
            "showSymbol": true,
            "smooth": false,
            "clip": true,
            "step": false,
            "data": [
                [
                    "2018Q1",
                    7.0
                ],
                [
                    "2018Q2",
                    8.0
                ],
                [
                    "2018Q3",
                    9.0
                ],
                [
                    "2018Q4",
                    8.0
                ],
                [
                    "2019Q1",
                    8.0
                ],
                [
                    "2019Q2",
                    9.0
                ],
                [
                    "2019Q3",
                    9.0
                ],
                [
                    "2019Q4",
                    8.0
                ],
                [
                    "2020Q1",
                    8.0
                ],
                [
                    "2020Q2",
                    9.0
                ],
                [
                    "2020Q3",
                    8.0
                ],
                [
                    "2020Q4",
                    9.0
                ],
                [
                    "2021Q1",
                    11.0
                ]
            ],
            "hoverAnimation": true,
            "label": {
                "show": true,
                "position": "top",
                "margin": 8
            },
            "lineStyle": {
                "show": true,
                "width": 1,
                "opacity": 1,
                "curveness": 0,
                "type": "solid"
            },
            "areaStyle": {
                "opacity": 0
            },
            "zlevel": 0,
            "z": 0
        },
        {
            "type": "line",
            "name": "Samsung\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09",
            "connectNulls": false,
            "symbolSize": 4,
            "showSymbol": true,
            "smooth": false,
            "clip": true,
            "step": false,
            "data": [
                [
                    "2018Q1",
                    22.0
                ],
                [
                    "2018Q2",
                    19.0
                ],
                [
                    "2018Q3",
                    19.0
                ],
                [
                    "2018Q4",
                    18.0
                ],
                [
                    "2019Q1",
                    21.0
                ],
                [
                    "2019Q2",
                    21.0
                ],
                [
                    "2019Q3",
                    21.0
                ],
                [
                    "2019Q4",
                    18.0
                ],
                [
                    "2020Q1",
                    20.0
                ],
                [
                    "2020Q2",
                    20.0
                ],
                [
                    "2020Q3",
                    22.0
                ],
                [
                    "2020Q4",
                    16.0
                ],
                [
                    "2021Q1",
                    22.0
                ]
            ],
            "hoverAnimation": true,
            "label": {
                "show": true,
                "position": "top",
                "margin": 8
            },
            "lineStyle": {
                "show": true,
                "width": 1,
                "opacity": 1,
                "curveness": 0,
                "type": "solid"
            },
            "areaStyle": {
                "opacity": 0
            },
            "zlevel": 0,
            "z": 0
        },
        {
            "type": "line",
            "name": "Vivo\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09",
            "connectNulls": false,
            "symbolSize": 4,
            "showSymbol": true,
            "smooth": false,
            "clip": true,
            "step": false,
            "data": [
                [
                    "2018Q1",
                    5.0
                ],
                [
                    "2018Q2",
                    7.0
                ],
                [
                    "2018Q3",
                    8.0
                ],
                [
                    "2018Q4",
                    7.0
                ],
                [
                    "2019Q1",
                    7.0
                ],
                [
                    "2019Q2",
                    8.0
                ],
                [
                    "2019Q3",
                    8.0
                ],
                [
                    "2019Q4",
                    8.0
                ],
                [
                    "2020Q1",
                    7.0
                ],
                [
                    "2020Q2",
                    8.0
                ],
                [
                    "2020Q3",
                    8.0
                ],
                [
                    "2020Q4",
                    8.0
                ],
                [
                    "2021Q1",
                    10.0
                ]
            ],
            "hoverAnimation": true,
            "label": {
                "show": true,
                "position": "top",
                "margin": 8
            },
            "lineStyle": {
                "show": true,
                "width": 1,
                "opacity": 1,
                "curveness": 0,
                "type": "solid"
            },
            "areaStyle": {
                "opacity": 0
            },
            "zlevel": 0,
            "z": 0
        },
        {
            "type": "line",
            "name": "Xiaomi\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09",
            "connectNulls": false,
            "symbolSize": 4,
            "showSymbol": true,
            "smooth": false,
            "clip": true,
            "step": false,
            "data": [
                [
                    "2018Q1",
                    8.0
                ],
                [
                    "2018Q2",
                    9.0
                ],
                [
                    "2018Q3",
                    9.0
                ],
                [
                    "2018Q4",
                    6.0
                ],
                [
                    "2019Q1",
                    8.0
                ],
                [
                    "2019Q2",
                    9.0
                ],
                [
                    "2019Q3",
                    8.0
                ],
                [
                    "2019Q4",
                    8.0
                ],
                [
                    "2020Q1",
                    10.0
                ],
                [
                    "2020Q2",
                    10.0
                ],
                [
                    "2020Q3",
                    13.0
                ],
                [
                    "2020Q4",
                    11.0
                ],
                [
                    "2021Q1",
                    14.0
                ]
            ],
            "hoverAnimation": true,
            "label": {
                "show": true,
                "position": "top",
                "margin": 8
            },
            "lineStyle": {
                "show": true,
                "width": 1,
                "opacity": 1,
                "curveness": 0,
                "type": "solid"
            },
            "areaStyle": {
                "opacity": 0
            },
            "zlevel": 0,
            "z": 0
        }
    ],
    "legend": [
        {
            "data": [
                "Apple\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09",
                "Huawei\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09",
                "Oppo\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09",
                "Samsung\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09",
                "Vivo\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09",
                "Xiaomi\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09"
            ],
            "selected": {
                "Apple\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09": true,
                "Huawei\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09": true,
                "Oppo\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09": true,
                "Samsung\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09": true,
                "Vivo\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09": true,
                "Xiaomi\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09": true
            },
            "type": "scroll",
            "show": true,
            "top": 10,
            "orient": "horizontal",
            "padding": 5,
            "itemGap": 10,
            "itemWidth": 25,
            "itemHeight": 14
        }
    ],
    "tooltip": {
        "show": true,
        "trigger": "item",
        "triggerOn": "mousemove|click",
        "axisPointer": {
            "type": "line"
        },
        "showContent": true,
        "alwaysShowContent": false,
        "showDelay": 0,
        "hideDelay": 100,
        "textStyle": {
            "fontSize": 14
        },
        "borderWidth": 0,
        "padding": 5
    },
    "xAxis": [
        {
            "show": true,
            "scale": false,
            "nameLocation": "end",
            "nameGap": 15,
            "gridIndex": 0,
            "inverse": false,
            "offset": 0,
            "splitNumber": 5,
            "minInterval": 0,
            "splitLine": {
                "show": false,
                "lineStyle": {
                    "show": true,
                    "width": 1,
                    "opacity": 1,
                    "curveness": 0,
                    "type": "solid"
                }
            },
            "data": [
                "2018Q1",
                "2018Q2",
                "2018Q3",
                "2018Q4",
                "2019Q1",
                "2019Q2",
                "2019Q3",
                "2019Q4",
                "2020Q1",
                "2020Q2",
                "2020Q3",
                "2020Q4",
                "2021Q1"
            ]
        }
    ],
    "yAxis": [
        {
            "show": true,
            "scale": false,
            "nameLocation": "end",
            "nameGap": 15,
            "gridIndex": 0,
            "inverse": false,
            "offset": 0,
            "splitNumber": 5,
            "minInterval": 0,
            "splitLine": {
                "show": false,
                "lineStyle": {
                    "show": true,
                    "width": 1,
                    "opacity": 1,
                    "curveness": 0,
                    "type": "solid"
                }
            }
        }
    ],
    "title": [
        {
            "padding": 5,
            "itemGap": 10
        }
    ]
};
        chart_1a350fedf1644c1b804480e2c16bbd63.setOption(option_1a350fedf1644c1b804480e2c16bbd63);
    </script>
    <script>   
    var x=window.innerWidth;
    function resizeFresh(){
        if(x!=window.innerWidth)
            location.reload();
    }
    </script>
</body>
</html>

##### 市场份额（按季度）饼状图/南丁格尔玫瑰图

```Python
from pyecharts.charts import Pie

pies = []
pie = Pie(init_opts=opts.InitOpts(theme=ThemeType.LIGHT,width='1500px',height='1500px'))
for xi, y in enumerate(GSMS.Year.unique()):
    _ = GSMS[(GSMS.Year == y)]
    for yi, q in enumerate(_.Quarterly.unique()):
        df_temp = GSMS[(GSMS.Year == y) & (GSMS.Quarterly == q)]
        
        data_pair = [list(_) for _ in zip(df_temp.Vendor.tolist() + ['Others'], 
                                          df_temp.Market_Share.tolist() + [round(1 - sum(df_temp.Market_Share.tolist()),1)])]
        pie = pie.add(series_name=str(y) + str(q),
                     data_pair=data_pair,
                     rosetype=None,#'area',
                     radius=75,
                     center=[300 * (yi + 1) - 150,  300 * (xi + 1) - 150],
                     label_opts=opts.LabelOpts(is_show=False, position="center"),)\
                .set_series_opts(tooltip_opts=opts.TooltipOpts(trigger="item", formatter="{b} <br/>{a}: {c}"),
                                 label_opts=opts.LabelOpts(color="rgba(0, 0, 1, 0.8)"),)\
                .set_global_opts(legend_opts=opts.LegendOpts(type_='scroll',pos_left=0, orient='horizontal'))
                
pie.render_notebook()
```

**2018 Q1 - Q4**

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Awesome-pyecharts</title>
    <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,minimum-scale=1.0,user-scalable=yes">
            <script type="text/javascript" src="../../pyecharts-assets-master/assets/echarts.min.js"></script>
    <style type="text/css">
        html,body{
            height:100%;
            width:100%
    }
    </style>
</head>
<body onresize="resizeFresh()">
    <div id="23d7d227653d4d80bcaa8c33887917e6" class="chart-container" style="width:95%; height:495%; margin:auto; top:0px"></div>
    <script>
        var chart_23d7d227653d4d80bcaa8c33887917e6 = echarts.init(
            document.getElementById('23d7d227653d4d80bcaa8c33887917e6'), 'light', {renderer: 'canvas'});
        var option_23d7d227653d4d80bcaa8c33887917e6 = {
    "animation": true,
    "animationThreshold": 2000,
    "animationDuration": 1000,
    "animationEasing": "cubicOut",
    "animationDelay": 0,
    "animationDurationUpdate": 300,
    "animationEasingUpdate": "cubicOut",
    "animationDelayUpdate": 0,
    "series": [
        {
            "type": "pie",
            "name": "2018Q1",
            "clockwise": true,
            "data": [
                {
                    "name": "Samsung",
                    "value": 0.22
                },
                {
                    "name": "Apple",
                    "value": 0.14
                },
                {
                    "name": "Huawei",
                    "value": 0.11
                },
                {
                    "name": "Xiaomi",
                    "value": 0.08
                },
                {
                    "name": "Oppo",
                    "value": 0.07
                },
                {
                    "name": "Vivo",
                    "value": 0.05
                },
                {
                    "name": "Others",
                    "value": 0.3
                }
            ],
            "radius": "25%",
            "center": [
                "25%",
                "25%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        },
        {
            "type": "pie",
            "name": "2018Q2",
            "clockwise": true,
            "data": [
                {
                    "name": "Samsung",
                    "value": 0.19
                },
                {
                    "name": "Apple",
                    "value": 0.11
                },
                {
                    "name": "Huawei",
                    "value": 0.15
                },
                {
                    "name": "Xiaomi",
                    "value": 0.09
                },
                {
                    "name": "Oppo",
                    "value": 0.08
                },
                {
                    "name": "Vivo",
                    "value": 0.07
                },
                {
                    "name": "Others",
                    "value": 0.3
                }
            ],
            "radius": "25%",
            "center": [
                "75%",
                "25%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        },
        {
            "type": "pie",
            "name": "2018Q3",
            "clockwise": true,
            "data": [
                {
                    "name": "Samsung",
                    "value": 0.19
                },
                {
                    "name": "Apple",
                    "value": 0.12
                },
                {
                    "name": "Huawei",
                    "value": 0.14
                },
                {
                    "name": "Xiaomi",
                    "value": 0.09
                },
                {
                    "name": "Oppo",
                    "value": 0.09
                },
                {
                    "name": "Vivo",
                    "value": 0.08
                },
                {
                    "name": "Others",
                    "value": 0.3
                }
            ],
            "radius": "25%",
            "center": [
                "25%",
                "75%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        },
        {
            "type": "pie",
            "name": "2018Q4",
            "clockwise": true,
            "data": [
                {
                    "name": "Samsung",
                    "value": 0.18
                },
                {
                    "name": "Apple",
                    "value": 0.17
                },
                {
                    "name": "Huawei",
                    "value": 0.15
                },
                {
                    "name": "Xiaomi",
                    "value": 0.06
                },
                {
                    "name": "Oppo",
                    "value": 0.08
                },
                {
                    "name": "Vivo",
                    "value": 0.07
                },
                {
                    "name": "Others",
                    "value": 0.3
                }
            ],
            "radius": "25%",
            "center": [
                "75%",
                "75%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        }
    ],
    "legend": [
        {
            "data": [
                "Samsung",
                "Apple",
                "Huawei",
                "Xiaomi",
                "Oppo",
                "Vivo",
                "Others"
            ],
            "selected": {},
            "type": "scroll",
            "show": true,
            "left": 0,
            "orient": "horizontal",
            "padding": 5,
            "itemGap": 10,
            "itemWidth": 25,
            "itemHeight": 14
        }
    ],
    "tooltip": {
        "show": true,
        "trigger": "item",
        "triggerOn": "mousemove|click",
        "axisPointer": {
            "type": "line"
        },
        "showContent": true,
        "alwaysShowContent": false,
        "showDelay": 0,
        "hideDelay": 100,
        "textStyle": {
            "fontSize": 14
        },
        "borderWidth": 0,
        "padding": 5
    },
    "title": [
        {
            "padding": 5,
            "itemGap": 10
        }
    ]
};
        chart_23d7d227653d4d80bcaa8c33887917e6.setOption(option_23d7d227653d4d80bcaa8c33887917e6);
    </script>
    <script>   
    var x=window.innerWidth;
    function resizeFresh(){
        if(x!=window.innerWidth)
            location.reload();
    }
    </script>
</body>
</html>

**2019 Q1 - Q4**

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Awesome-pyecharts</title>
    <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,minimum-scale=1.0,user-scalable=yes">
            <script type="text/javascript" src="../../pyecharts-assets-master/assets/echarts.min.js"></script>
    <style type="text/css">
        html,body{
            height:100%;
            width:100%
    }
    </style>
</head>
<body onresize="resizeFresh()">
    <div id="4e8fa8394cc74b9a8a84bb8d38012a9b" class="chart-container" style="width:95%; height:495%; margin:auto; top:0px"></div>
    <script>
        var chart_4e8fa8394cc74b9a8a84bb8d38012a9b = echarts.init(
            document.getElementById('4e8fa8394cc74b9a8a84bb8d38012a9b'), 'light', {renderer: 'canvas'});
        var option_4e8fa8394cc74b9a8a84bb8d38012a9b = {
    "animation": true,
    "animationThreshold": 2000,
    "animationDuration": 1000,
    "animationEasing": "cubicOut",
    "animationDelay": 0,
    "animationDurationUpdate": 300,
    "animationEasingUpdate": "cubicOut",
    "animationDelayUpdate": 0,
    "series": [
        {
            "type": "pie",
            "name": "2019Q1",
            "clockwise": true,
            "data": [
                {
                    "name": "Samsung",
                    "value": 0.21
                },
                {
                    "name": "Apple",
                    "value": 0.12
                },
                {
                    "name": "Huawei",
                    "value": 0.17
                },
                {
                    "name": "Xiaomi",
                    "value": 0.08
                },
                {
                    "name": "Oppo",
                    "value": 0.08
                },
                {
                    "name": "Vivo",
                    "value": 0.07
                },
                {
                    "name": "Others",
                    "value": 0.3
                }
            ],
            "radius": "25%",
            "center": [
                "25%",
                "25%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        },
        {
            "type": "pie",
            "name": "2019Q2",
            "clockwise": true,
            "data": [
                {
                    "name": "Samsung",
                    "value": 0.21
                },
                {
                    "name": "Apple",
                    "value": 0.1
                },
                {
                    "name": "Huawei",
                    "value": 0.16
                },
                {
                    "name": "Xiaomi",
                    "value": 0.09
                },
                {
                    "name": "Oppo",
                    "value": 0.09
                },
                {
                    "name": "Vivo",
                    "value": 0.08
                },
                {
                    "name": "Others",
                    "value": 0.3
                }
            ],
            "radius": "25%",
            "center": [
                "75%",
                "25%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        },
        {
            "type": "pie",
            "name": "2019Q3",
            "clockwise": true,
            "data": [
                {
                    "name": "Samsung",
                    "value": 0.21
                },
                {
                    "name": "Apple",
                    "value": 0.12
                },
                {
                    "name": "Huawei",
                    "value": 0.18
                },
                {
                    "name": "Xiaomi",
                    "value": 0.08
                },
                {
                    "name": "Oppo",
                    "value": 0.09
                },
                {
                    "name": "Vivo",
                    "value": 0.08
                },
                {
                    "name": "Others",
                    "value": 0.2
                }
            ],
            "radius": "25%",
            "center": [
                "25%",
                "75%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        },
        {
            "type": "pie",
            "name": "2019Q4",
            "clockwise": true,
            "data": [
                {
                    "name": "Samsung",
                    "value": 0.18
                },
                {
                    "name": "Apple",
                    "value": 0.18
                },
                {
                    "name": "Huawei",
                    "value": 0.14
                },
                {
                    "name": "Xiaomi",
                    "value": 0.08
                },
                {
                    "name": "Oppo",
                    "value": 0.08
                },
                {
                    "name": "Vivo",
                    "value": 0.08
                },
                {
                    "name": "Others",
                    "value": 0.3
                }
            ],
            "radius": "25%",
            "center": [
                "75%",
                "75%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        }
    ],
    "legend": [
        {
            "data": [
                "Samsung",
                "Apple",
                "Huawei",
                "Xiaomi",
                "Oppo",
                "Vivo",
                "Others"
            ],
            "selected": {},
            "type": "scroll",
            "show": true,
            "left": 0,
            "orient": "horizontal",
            "padding": 5,
            "itemGap": 10,
            "itemWidth": 25,
            "itemHeight": 14
        }
    ],
    "tooltip": {
        "show": true,
        "trigger": "item",
        "triggerOn": "mousemove|click",
        "axisPointer": {
            "type": "line"
        },
        "showContent": true,
        "alwaysShowContent": false,
        "showDelay": 0,
        "hideDelay": 100,
        "textStyle": {
            "fontSize": 14
        },
        "borderWidth": 0,
        "padding": 5
    },
    "title": [
        {
            "padding": 5,
            "itemGap": 10
        }
    ]
};
        chart_4e8fa8394cc74b9a8a84bb8d38012a9b.setOption(option_4e8fa8394cc74b9a8a84bb8d38012a9b);
    </script>
    <script>   
    var x=window.innerWidth;
    function resizeFresh(){
        if(x!=window.innerWidth)
            location.reload();
    }
    </script>
</body>
</html>

**2020 Q1 - Q4**
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Awesome-pyecharts</title>
    <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,minimum-scale=1.0,user-scalable=yes">
            <script type="text/javascript" src="../../pyecharts-assets-master/assets/echarts.min.js"></script>
    <style type="text/css">
        html,body{
            height:100%;
            width:100%
    }
    </style>
</head>
<body onresize="resizeFresh()">
    <div id="fe82505eb0764a5aa8c1c73870c13b0c" class="chart-container" style="width:95%; height:495%; margin:auto; top:0px"></div>
    <script>
        var chart_fe82505eb0764a5aa8c1c73870c13b0c = echarts.init(
            document.getElementById('fe82505eb0764a5aa8c1c73870c13b0c'), 'light', {renderer: 'canvas'});
        var option_fe82505eb0764a5aa8c1c73870c13b0c = {
    "animation": true,
    "animationThreshold": 2000,
    "animationDuration": 1000,
    "animationEasing": "cubicOut",
    "animationDelay": 0,
    "animationDurationUpdate": 300,
    "animationEasingUpdate": "cubicOut",
    "animationDelayUpdate": 0,
    "series": [
        {
            "type": "pie",
            "name": "2020Q1",
            "clockwise": true,
            "data": [
                {
                    "name": "Samsung",
                    "value": 0.2
                },
                {
                    "name": "Apple",
                    "value": 0.14
                },
                {
                    "name": "Huawei",
                    "value": 0.17
                },
                {
                    "name": "Xiaomi",
                    "value": 0.1
                },
                {
                    "name": "Oppo",
                    "value": 0.08
                },
                {
                    "name": "Vivo",
                    "value": 0.07
                },
                {
                    "name": "Others",
                    "value": 0.2
                }
            ],
            "radius": "25%",
            "center": [
                "25%",
                "25%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        },
        {
            "type": "pie",
            "name": "2020Q2",
            "clockwise": true,
            "data": [
                {
                    "name": "Samsung",
                    "value": 0.2
                },
                {
                    "name": "Apple",
                    "value": 0.14
                },
                {
                    "name": "Huawei",
                    "value": 0.2
                },
                {
                    "name": "Xiaomi",
                    "value": 0.1
                },
                {
                    "name": "Oppo",
                    "value": 0.09
                },
                {
                    "name": "Vivo",
                    "value": 0.08
                },
                {
                    "name": "Others",
                    "value": 0.2
                }
            ],
            "radius": "25%",
            "center": [
                "75%",
                "25%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        },
        {
            "type": "pie",
            "name": "2020Q3",
            "clockwise": true,
            "data": [
                {
                    "name": "Samsung",
                    "value": 0.22
                },
                {
                    "name": "Apple",
                    "value": 0.11
                },
                {
                    "name": "Huawei",
                    "value": 0.14
                },
                {
                    "name": "Xiaomi",
                    "value": 0.13
                },
                {
                    "name": "Oppo",
                    "value": 0.08
                },
                {
                    "name": "Vivo",
                    "value": 0.08
                },
                {
                    "name": "Others",
                    "value": 0.2
                }
            ],
            "radius": "25%",
            "center": [
                "25%",
                "75%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        },
        {
            "type": "pie",
            "name": "2020Q4",
            "clockwise": true,
            "data": [
                {
                    "name": "Samsung",
                    "value": 0.16
                },
                {
                    "name": "Apple",
                    "value": 0.21
                },
                {
                    "name": "Huawei",
                    "value": 0.08
                },
                {
                    "name": "Xiaomi",
                    "value": 0.11
                },
                {
                    "name": "Oppo",
                    "value": 0.09
                },
                {
                    "name": "Vivo",
                    "value": 0.08
                },
                {
                    "name": "Others",
                    "value": 0.3
                }
            ],
            "radius": "25%",
            "center": [
                "75%",
                "75%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        }
    ],
    "legend": [
        {
            "data": [
                "Samsung",
                "Apple",
                "Huawei",
                "Xiaomi",
                "Oppo",
                "Vivo",
                "Others"
            ],
            "selected": {},
            "type": "scroll",
            "show": true,
            "left": 0,
            "orient": "horizontal",
            "padding": 5,
            "itemGap": 10,
            "itemWidth": 25,
            "itemHeight": 14
        }
    ],
    "tooltip": {
        "show": true,
        "trigger": "item",
        "triggerOn": "mousemove|click",
        "axisPointer": {
            "type": "line"
        },
        "showContent": true,
        "alwaysShowContent": false,
        "showDelay": 0,
        "hideDelay": 100,
        "textStyle": {
            "fontSize": 14
        },
        "borderWidth": 0,
        "padding": 5
    },
    "title": [
        {
            "padding": 5,
            "itemGap": 10
        }
    ]
};
        chart_fe82505eb0764a5aa8c1c73870c13b0c.setOption(option_fe82505eb0764a5aa8c1c73870c13b0c);
    </script>
    <script>   
    var x=window.innerWidth;
    function resizeFresh(){
        if(x!=window.innerWidth)
            location.reload();
    }
    </script>
</body>
</html>

**2021 Q1**

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Awesome-pyecharts</title>
    <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,minimum-scale=1.0,user-scalable=yes">
            <script type="text/javascript" src="../../pyecharts-assets-master/assets/echarts.min.js"></script>
    <style type="text/css">
        html,body{
            height:100%;
            width:100%
    }
    </style>
</head>
<body onresize="resizeFresh()">
    <div id="e9932162f97641d68914deb431d49724" class="chart-container" style="width:95%; height:495%; margin:auto; top:0px"></div>
    <script>
        var chart_e9932162f97641d68914deb431d49724 = echarts.init(
            document.getElementById('e9932162f97641d68914deb431d49724'), 'light', {renderer: 'canvas'});
        var option_e9932162f97641d68914deb431d49724 = {
    "animation": true,
    "animationThreshold": 2000,
    "animationDuration": 1000,
    "animationEasing": "cubicOut",
    "animationDelay": 0,
    "animationDurationUpdate": 300,
    "animationEasingUpdate": "cubicOut",
    "animationDelayUpdate": 0,
    "series": [
        {
            "type": "pie",
            "name": "2021Q1",
            "clockwise": true,
            "data": [
                {
                    "name": "Samsung",
                    "value": 0.22
                },
                {
                    "name": "Apple",
                    "value": 0.17
                },
                {
                    "name": "Huawei",
                    "value": 0.04
                },
                {
                    "name": "Xiaomi",
                    "value": 0.14
                },
                {
                    "name": "Oppo",
                    "value": 0.11
                },
                {
                    "name": "Vivo",
                    "value": 0.1
                },
                {
                    "name": "Others",
                    "value": 0.2
                }
            ],
            "radius": "75%",
            "center": [
                "50%",
                "50%"
            ],
            "roseType": "area",
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        }
    ],
    "legend": [
        {
            "data": [
                "Samsung",
                "Apple",
                "Huawei",
                "Xiaomi",
                "Oppo",
                "Vivo",
                "Others"
            ],
            "selected": {},
            "type": "scroll",
            "show": true,
            "left": 0,
            "orient": "horizontal",
            "padding": 5,
            "itemGap": 10,
            "itemWidth": 25,
            "itemHeight": 14
        }
    ],
    "tooltip": {
        "show": true,
        "trigger": "item",
        "triggerOn": "mousemove|click",
        "axisPointer": {
            "type": "line"
        },
        "showContent": true,
        "alwaysShowContent": false,
        "showDelay": 0,
        "hideDelay": 100,
        "textStyle": {
            "fontSize": 14
        },
        "borderWidth": 0,
        "padding": 5
    },
    "title": [
        {
            "padding": 5,
            "itemGap": 10
        }
    ]
};
        chart_e9932162f97641d68914deb431d49724.setOption(option_e9932162f97641d68914deb431d49724);
    </script>
    <script>   
    var x=window.innerWidth;
    function resizeFresh(){
        if(x!=window.innerWidth)
            location.reload();
    }
    </script>
</body>
</html>

### 国内手机市场份额数据可视化
```Python
CSMS = pd.read_csv('China_Smartphone_Market_Share.csv').iloc[:,0:4]
CSMS.Market_Share = CSMS.Market_Share.str[:-1].astype(int)
```

#### 2017Q1 - 2021Q1 国内智能手机市场份额
##### 市场份额（按季度）折线图
```Python
from pyecharts.charts import Line

line = Line(init_opts=opts.InitOpts(theme=ThemeType.LIGHT,
                                  width='1200px',
                                  height='550px'))\
            .add_xaxis((CSMS.Year.astype(str) + CSMS.Quarterly.astype(str)).drop_duplicates(keep='first').tolist())\
            .set_global_opts(title_opts=opts.TitleOpts(title=''),\
                             legend_opts=opts.LegendOpts(type_='scroll',pos_top=10, orient='horizontal'))#horizontal vertical


for v in grouped_df_year.Vendor.unique():
    line = line.add_yaxis(str(v) + '季度全球份额（%）',np.round(100 * CSMS[CSMS.Vendor == v].Market_Share, 0).tolist())

line.render_notebook()
```

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Awesome-pyecharts</title>
    <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,minimum-scale=1.0,user-scalable=yes">
            <script type="text/javascript" src="../../pyecharts-assets-master/assets/echarts.min.js"></script>
    <style type="text/css">
        html,body{
            height:100%;
            width:100%
    }
    </style>
</head>
<body onresize="resizeFresh()">
    <div id="622d0adc2696444983cb91188184e257" class="chart-container" style="width:95%; height:495%; margin:auto; top:0px"></div>
    <script>
        var chart_622d0adc2696444983cb91188184e257 = echarts.init(
            document.getElementById('622d0adc2696444983cb91188184e257'), 'light', {renderer: 'canvas'});
        var option_622d0adc2696444983cb91188184e257 = {
    "animation": true,
    "animationThreshold": 2000,
    "animationDuration": 1000,
    "animationEasing": "cubicOut",
    "animationDelay": 0,
    "animationDurationUpdate": 300,
    "animationEasingUpdate": "cubicOut",
    "animationDelayUpdate": 0,
    "series": [
        {
            "type": "line",
            "name": "Apple\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09",
            "connectNulls": false,
            "symbolSize": 4,
            "showSymbol": true,
            "smooth": false,
            "clip": true,
            "step": false,
            "data": [
                [
                    "2017Q1",
                    10
                ],
                [
                    "2017Q2",
                    8
                ],
                [
                    "2017Q3",
                    10
                ],
                [
                    "2017Q4",
                    15
                ],
                [
                    "2018Q1",
                    13
                ],
                [
                    "2018Q2",
                    8
                ],
                [
                    "2018Q3",
                    9
                ],
                [
                    "2018Q4",
                    12
                ],
                [
                    "2019Q1",
                    9
                ],
                [
                    "2019Q2",
                    6
                ],
                [
                    "2019Q3",
                    8
                ],
                [
                    "2019Q4",
                    14
                ],
                [
                    "2020Q1",
                    9
                ],
                [
                    "2020Q2",
                    8
                ],
                [
                    "2020Q3",
                    8
                ],
                [
                    "2020Q4",
                    8
                ],
                [
                    "2021Q1",
                    13
                ]
            ],
            "hoverAnimation": true,
            "label": {
                "show": true,
                "position": "top",
                "margin": 8
            },
            "lineStyle": {
                "show": true,
                "width": 1,
                "opacity": 1,
                "curveness": 0,
                "type": "solid"
            },
            "areaStyle": {
                "opacity": 0
            },
            "zlevel": 0,
            "z": 0
        },
        {
            "type": "line",
            "name": "Huawei\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09",
            "connectNulls": false,
            "symbolSize": 4,
            "showSymbol": true,
            "smooth": false,
            "clip": true,
            "step": false,
            "data": [
                [
                    "2017Q1",
                    20
                ],
                [
                    "2017Q2",
                    20
                ],
                [
                    "2017Q3",
                    19
                ],
                [
                    "2017Q4",
                    20
                ],
                [
                    "2018Q1",
                    22
                ],
                [
                    "2018Q2",
                    26
                ],
                [
                    "2018Q3",
                    23
                ],
                [
                    "2018Q4",
                    28
                ],
                [
                    "2019Q1",
                    34
                ],
                [
                    "2019Q2",
                    35
                ],
                [
                    "2019Q3",
                    40
                ],
                [
                    "2019Q4",
                    35
                ],
                [
                    "2020Q1",
                    41
                ],
                [
                    "2020Q2",
                    46
                ],
                [
                    "2020Q3",
                    43
                ],
                [
                    "2020Q4",
                    30
                ],
                [
                    "2021Q1",
                    16
                ]
            ],
            "hoverAnimation": true,
            "label": {
                "show": true,
                "position": "top",
                "margin": 8
            },
            "lineStyle": {
                "show": true,
                "width": 1,
                "opacity": 1,
                "curveness": 0,
                "type": "solid"
            },
            "areaStyle": {
                "opacity": 0
            },
            "zlevel": 0,
            "z": 0
        },
        {
            "type": "line",
            "name": "Oppo\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09",
            "connectNulls": false,
            "symbolSize": 4,
            "showSymbol": true,
            "smooth": false,
            "clip": true,
            "step": false,
            "data": [
                [
                    "2017Q1",
                    17
                ],
                [
                    "2017Q2",
                    19
                ],
                [
                    "2017Q3",
                    19
                ],
                [
                    "2017Q4",
                    17
                ],
                [
                    "2018Q1",
                    18
                ],
                [
                    "2018Q2",
                    18
                ],
                [
                    "2018Q3",
                    21
                ],
                [
                    "2018Q4",
                    19
                ],
                [
                    "2019Q1",
                    18
                ],
                [
                    "2019Q2",
                    19
                ],
                [
                    "2019Q3",
                    18
                ],
                [
                    "2019Q4",
                    16
                ],
                [
                    "2020Q1",
                    15
                ],
                [
                    "2020Q2",
                    16
                ],
                [
                    "2020Q3",
                    16
                ],
                [
                    "2020Q4",
                    16
                ],
                [
                    "2021Q1",
                    22
                ]
            ],
            "hoverAnimation": true,
            "label": {
                "show": true,
                "position": "top",
                "margin": 8
            },
            "lineStyle": {
                "show": true,
                "width": 1,
                "opacity": 1,
                "curveness": 0,
                "type": "solid"
            },
            "areaStyle": {
                "opacity": 0
            },
            "zlevel": 0,
            "z": 0
        },
        {
            "type": "line",
            "name": "Samsung\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09",
            "connectNulls": false,
            "symbolSize": 4,
            "showSymbol": true,
            "smooth": false,
            "clip": true,
            "step": false,
            "data": [],
            "hoverAnimation": true,
            "label": {
                "show": true,
                "position": "top",
                "margin": 8
            },
            "lineStyle": {
                "show": true,
                "width": 1,
                "opacity": 1,
                "curveness": 0,
                "type": "solid"
            },
            "areaStyle": {
                "opacity": 0
            },
            "zlevel": 0,
            "z": 0
        },
        {
            "type": "line",
            "name": "Vivo\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09",
            "connectNulls": false,
            "symbolSize": 4,
            "showSymbol": true,
            "smooth": false,
            "clip": true,
            "step": false,
            "data": [
                [
                    "2017Q1",
                    17
                ],
                [
                    "2017Q2",
                    17
                ],
                [
                    "2017Q3",
                    19
                ],
                [
                    "2017Q4",
                    15
                ],
                [
                    "2018Q1",
                    16
                ],
                [
                    "2018Q2",
                    19
                ],
                [
                    "2018Q3",
                    21
                ],
                [
                    "2018Q4",
                    20
                ],
                [
                    "2019Q1",
                    19
                ],
                [
                    "2019Q2",
                    18
                ],
                [
                    "2019Q3",
                    19
                ],
                [
                    "2019Q4",
                    17
                ],
                [
                    "2020Q1",
                    17
                ],
                [
                    "2020Q2",
                    16
                ],
                [
                    "2020Q3",
                    18
                ],
                [
                    "2020Q4",
                    18
                ],
                [
                    "2021Q1",
                    23
                ]
            ],
            "hoverAnimation": true,
            "label": {
                "show": true,
                "position": "top",
                "margin": 8
            },
            "lineStyle": {
                "show": true,
                "width": 1,
                "opacity": 1,
                "curveness": 0,
                "type": "solid"
            },
            "areaStyle": {
                "opacity": 0
            },
            "zlevel": 0,
            "z": 0
        },
        {
            "type": "line",
            "name": "Xiaomi\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09",
            "connectNulls": false,
            "symbolSize": 4,
            "showSymbol": true,
            "smooth": false,
            "clip": true,
            "step": false,
            "data": [
                [
                    "2017Q1",
                    8
                ],
                [
                    "2017Q2",
                    13
                ],
                [
                    "2017Q3",
                    14
                ],
                [
                    "2017Q4",
                    15
                ],
                [
                    "2018Q1",
                    13
                ],
                [
                    "2018Q2",
                    13
                ],
                [
                    "2018Q3",
                    13
                ],
                [
                    "2018Q4",
                    9
                ],
                [
                    "2019Q1",
                    12
                ],
                [
                    "2019Q2",
                    11
                ],
                [
                    "2019Q3",
                    8
                ],
                [
                    "2019Q4",
                    9
                ],
                [
                    "2020Q1",
                    11
                ],
                [
                    "2020Q2",
                    10
                ],
                [
                    "2020Q3",
                    13
                ],
                [
                    "2020Q4",
                    13
                ],
                [
                    "2021Q1",
                    15
                ]
            ],
            "hoverAnimation": true,
            "label": {
                "show": true,
                "position": "top",
                "margin": 8
            },
            "lineStyle": {
                "show": true,
                "width": 1,
                "opacity": 1,
                "curveness": 0,
                "type": "solid"
            },
            "areaStyle": {
                "opacity": 0
            },
            "zlevel": 0,
            "z": 0
        }
    ],
    "legend": [
        {
            "data": [
                "Apple\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09",
                "Huawei\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09",
                "Oppo\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09",
                "Samsung\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09",
                "Vivo\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09",
                "Xiaomi\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09"
            ],
            "selected": {
                "Apple\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09": true,
                "Huawei\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09": true,
                "Oppo\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09": true,
                "Samsung\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09": true,
                "Vivo\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09": true,
                "Xiaomi\u5b63\u5ea6\u5168\u7403\u4efd\u989d\uff08%\uff09": true
            },
            "type": "scroll",
            "show": true,
            "top": 10,
            "orient": "horizontal",
            "padding": 5,
            "itemGap": 10,
            "itemWidth": 25,
            "itemHeight": 14
        }
    ],
    "tooltip": {
        "show": true,
        "trigger": "item",
        "triggerOn": "mousemove|click",
        "axisPointer": {
            "type": "line"
        },
        "showContent": true,
        "alwaysShowContent": false,
        "showDelay": 0,
        "hideDelay": 100,
        "textStyle": {
            "fontSize": 14
        },
        "borderWidth": 0,
        "padding": 5
    },
    "xAxis": [
        {
            "show": true,
            "scale": false,
            "nameLocation": "end",
            "nameGap": 15,
            "gridIndex": 0,
            "inverse": false,
            "offset": 0,
            "splitNumber": 5,
            "minInterval": 0,
            "splitLine": {
                "show": false,
                "lineStyle": {
                    "show": true,
                    "width": 1,
                    "opacity": 1,
                    "curveness": 0,
                    "type": "solid"
                }
            },
            "data": [
                "2017Q1",
                "2017Q2",
                "2017Q3",
                "2017Q4",
                "2018Q1",
                "2018Q2",
                "2018Q3",
                "2018Q4",
                "2019Q1",
                "2019Q2",
                "2019Q3",
                "2019Q4",
                "2020Q1",
                "2020Q2",
                "2020Q3",
                "2020Q4",
                "2021Q1"
            ]
        }
    ],
    "yAxis": [
        {
            "show": true,
            "scale": false,
            "nameLocation": "end",
            "nameGap": 15,
            "gridIndex": 0,
            "inverse": false,
            "offset": 0,
            "splitNumber": 5,
            "minInterval": 0,
            "splitLine": {
                "show": false,
                "lineStyle": {
                    "show": true,
                    "width": 1,
                    "opacity": 1,
                    "curveness": 0,
                    "type": "solid"
                }
            }
        }
    ],
    "title": [
        {
            "padding": 5,
            "itemGap": 10
        }
    ]
};
        chart_622d0adc2696444983cb91188184e257.setOption(option_622d0adc2696444983cb91188184e257);
    </script>
    <script>   
    var x=window.innerWidth;
    function resizeFresh(){
        if(x!=window.innerWidth)
            location.reload();
    }
    </script>
</body>
</html>

##### 市场份额（按季度）饼状图/南丁格尔玫瑰图

```Python
from pyecharts.charts import Pie

pies = []
pie = Pie(init_opts=opts.InitOpts(theme=ThemeType.LIGHT,width='1500px',height='1500px'))
for xi, y in enumerate(GSMS.Year.unique()):
    _ = CSMS[(CSMS.Year == y)]
    for yi, q in enumerate(_.Quarterly.unique()):
        df_temp = CSMS[(CSMS.Year == y) & (CSMS.Quarterly == q)]
        
        data_pair = [list(_) for _ in zip(df_temp.Vendor.tolist() + ['Others'], 
                                          df_temp.Market_Share.tolist() + [round(1 - sum(df_temp.Market_Share.tolist()),1)])]
        pie = pie.add(series_name=str(y) + str(q),
                     data_pair=data_pair,
                     rosetype=None,#'area',
                     radius=75,
                     center=[300 * (yi + 1) - 150,  300 * (xi + 1) - 150],
                     label_opts=opts.LabelOpts(is_show=False, position="center"),)\
                .set_series_opts(tooltip_opts=opts.TooltipOpts(trigger="item", formatter="{b} <br/>{a}: {c}"),
                                 label_opts=opts.LabelOpts(color="rgba(0, 0, 1, 0.8)"),)\
                .set_global_opts(legend_opts=opts.LegendOpts(type_='scroll',pos_left=0, orient='horizontal'))

pie.render_notebook()
```

**2017 Q1 - Q4**  

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Awesome-pyecharts</title>
    <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,minimum-scale=1.0,user-scalable=yes">
            <script type="text/javascript" src="../../pyecharts-assets-master/assets/echarts.min.js"></script>
    <style type="text/css">
        html,body{
            height:100%;
            width:100%
    }
    </style>
</head>
<body onresize="resizeFresh()">
    <div id="88cd37287e9f41af83bdf529e6605e92" class="chart-container" style="width:95%; height:495%; margin:auto; top:0px"></div>
    <script>
        var chart_88cd37287e9f41af83bdf529e6605e92 = echarts.init(
            document.getElementById('88cd37287e9f41af83bdf529e6605e92'), 'light', {renderer: 'canvas'});
        var option_88cd37287e9f41af83bdf529e6605e92 = {
    "animation": true,
    "animationThreshold": 2000,
    "animationDuration": 1000,
    "animationEasing": "cubicOut",
    "animationDelay": 0,
    "animationDurationUpdate": 300,
    "animationEasingUpdate": "cubicOut",
    "animationDelayUpdate": 0,
    "series": [
        {
            "type": "pie",
            "name": "2017Q1",
            "clockwise": true,
            "data": [
                {
                    "name": "Huawei",
                    "value": 0.2
                },
                {
                    "name": "Vivo",
                    "value": 0.17
                },
                {
                    "name": "Oppo",
                    "value": 0.17
                },
                {
                    "name": "Xiaomi",
                    "value": 0.08
                },
                {
                    "name": "Apple",
                    "value": 0.1
                },
                {
                    "name": "Others",
                    "value": 0.3
                }
            ],
            "radius": "25%",
            "center": [
                "25%",
                "25%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        },
        {
            "type": "pie",
            "name": "2017Q2",
            "clockwise": true,
            "data": [
                {
                    "name": "Huawei",
                    "value": 0.2
                },
                {
                    "name": "Vivo",
                    "value": 0.17
                },
                {
                    "name": "Oppo",
                    "value": 0.19
                },
                {
                    "name": "Xiaomi",
                    "value": 0.13
                },
                {
                    "name": "Apple",
                    "value": 0.08
                },
                {
                    "name": "Others",
                    "value": 0.2
                }
            ],
            "radius": "25%",
            "center": [
                "75%",
                "25%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        },
        {
            "type": "pie",
            "name": "2017Q3",
            "clockwise": true,
            "data": [
                {
                    "name": "Huawei",
                    "value": 0.19
                },
                {
                    "name": "Vivo",
                    "value": 0.19
                },
                {
                    "name": "Oppo",
                    "value": 0.19
                },
                {
                    "name": "Xiaomi",
                    "value": 0.14
                },
                {
                    "name": "Apple",
                    "value": 0.1
                },
                {
                    "name": "Others",
                    "value": 0.2
                }
            ],
            "radius": "25%",
            "center": [
                "25%",
                "75%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        },
        {
            "type": "pie",
            "name": "2017Q4",
            "clockwise": true,
            "data": [
                {
                    "name": "Huawei",
                    "value": 0.2
                },
                {
                    "name": "Vivo",
                    "value": 0.15
                },
                {
                    "name": "Oppo",
                    "value": 0.17
                },
                {
                    "name": "Xiaomi",
                    "value": 0.15
                },
                {
                    "name": "Apple",
                    "value": 0.15
                },
                {
                    "name": "Others",
                    "value": 0.2
                }
            ],
            "radius": "25%",
            "center": [
                "75%",
                "75%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        }
    ],
    "legend": [
        {
            "data": [
                "Huawei",
                "Vivo",
                "Oppo",
                "Xiaomi",
                "Apple",
                "Others"
            ],
            "selected": {},
            "type": "scroll",
            "show": true,
            "left": 0,
            "orient": "horizontal",
            "padding": 5,
            "itemGap": 10,
            "itemWidth": 25,
            "itemHeight": 14
        }
    ],
    "tooltip": {
        "show": true,
        "trigger": "item",
        "triggerOn": "mousemove|click",
        "axisPointer": {
            "type": "line"
        },
        "showContent": true,
        "alwaysShowContent": false,
        "showDelay": 0,
        "hideDelay": 100,
        "textStyle": {
            "fontSize": 14
        },
        "borderWidth": 0,
        "padding": 5
    },
    "title": [
        {
            "padding": 5,
            "itemGap": 10
        }
    ]
};
        chart_88cd37287e9f41af83bdf529e6605e92.setOption(option_88cd37287e9f41af83bdf529e6605e92);
    </script>
    <script>   
    var x=window.innerWidth;
    function resizeFresh(){
        if(x!=window.innerWidth)
            location.reload();
    }
    </script>
</body>
</html>


**2018 Q1 - Q4**

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Awesome-pyecharts</title>
    <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,minimum-scale=1.0,user-scalable=yes">
            <script type="text/javascript" src="../../pyecharts-assets-master/assets/echarts.min.js"></script>
    <style type="text/css">
        html,body{
            height:100%;
            width:100%
    }
    </style>
</head>
<body onresize="resizeFresh()">
    <div id="4d03d0cb260141bdb749db0a60e4c887" class="chart-container" style="width:95%; height:495%; margin:auto; top:0px"></div>
    <script>
        var chart_4d03d0cb260141bdb749db0a60e4c887 = echarts.init(
            document.getElementById('4d03d0cb260141bdb749db0a60e4c887'), 'light', {renderer: 'canvas'});
        var option_4d03d0cb260141bdb749db0a60e4c887 = {
    "animation": true,
    "animationThreshold": 2000,
    "animationDuration": 1000,
    "animationEasing": "cubicOut",
    "animationDelay": 0,
    "animationDurationUpdate": 300,
    "animationEasingUpdate": "cubicOut",
    "animationDelayUpdate": 0,
    "series": [
        {
            "type": "pie",
            "name": "2018Q1",
            "clockwise": true,
            "data": [
                {
                    "name": "Huawei",
                    "value": 0.22
                },
                {
                    "name": "Vivo",
                    "value": 0.16
                },
                {
                    "name": "Oppo",
                    "value": 0.18
                },
                {
                    "name": "Xiaomi",
                    "value": 0.13
                },
                {
                    "name": "Apple",
                    "value": 0.13
                },
                {
                    "name": "Others",
                    "value": 0.2
                }
            ],
            "radius": "25%",
            "center": [
                "25%",
                "25%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        },
        {
            "type": "pie",
            "name": "2018Q2",
            "clockwise": true,
            "data": [
                {
                    "name": "Huawei",
                    "value": 0.26
                },
                {
                    "name": "Vivo",
                    "value": 0.19
                },
                {
                    "name": "Oppo",
                    "value": 0.18
                },
                {
                    "name": "Xiaomi",
                    "value": 0.13
                },
                {
                    "name": "Apple",
                    "value": 0.08
                },
                {
                    "name": "Others",
                    "value": 0.2
                }
            ],
            "radius": "25%",
            "center": [
                "75%",
                "25%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        },
        {
            "type": "pie",
            "name": "2018Q3",
            "clockwise": true,
            "data": [
                {
                    "name": "Huawei",
                    "value": 0.23
                },
                {
                    "name": "Vivo",
                    "value": 0.21
                },
                {
                    "name": "Oppo",
                    "value": 0.21
                },
                {
                    "name": "Xiaomi",
                    "value": 0.13
                },
                {
                    "name": "Apple",
                    "value": 0.09
                },
                {
                    "name": "Others",
                    "value": 0.1
                }
            ],
            "radius": "25%",
            "center": [
                "25%",
                "75%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        },
        {
            "type": "pie",
            "name": "2018Q4",
            "clockwise": true,
            "data": [
                {
                    "name": "Huawei",
                    "value": 0.28
                },
                {
                    "name": "Vivo",
                    "value": 0.2
                },
                {
                    "name": "Oppo",
                    "value": 0.19
                },
                {
                    "name": "Xiaomi",
                    "value": 0.09
                },
                {
                    "name": "Apple",
                    "value": 0.12
                },
                {
                    "name": "Others",
                    "value": 0.1
                }
            ],
            "radius": "25%",
            "center": [
                "75%",
                "75%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        }
    ],
    "legend": [
        {
            "data": [
                "Huawei",
                "Vivo",
                "Oppo",
                "Xiaomi",
                "Apple",
                "Others"
            ],
            "selected": {},
            "type": "scroll",
            "show": true,
            "left": 0,
            "orient": "horizontal",
            "padding": 5,
            "itemGap": 10,
            "itemWidth": 25,
            "itemHeight": 14
        }
    ],
    "tooltip": {
        "show": true,
        "trigger": "item",
        "triggerOn": "mousemove|click",
        "axisPointer": {
            "type": "line"
        },
        "showContent": true,
        "alwaysShowContent": false,
        "showDelay": 0,
        "hideDelay": 100,
        "textStyle": {
            "fontSize": 14
        },
        "borderWidth": 0,
        "padding": 5
    },
    "title": [
        {
            "padding": 5,
            "itemGap": 10
        }
    ]
};
        chart_4d03d0cb260141bdb749db0a60e4c887.setOption(option_4d03d0cb260141bdb749db0a60e4c887);
    </script>
    <script>   
    var x=window.innerWidth;
    function resizeFresh(){
        if(x!=window.innerWidth)
            location.reload();
    }
    </script>
</body>
</html>

**2019 Q1 - Q4**

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Awesome-pyecharts</title>
    <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,minimum-scale=1.0,user-scalable=yes">
            <script type="text/javascript" src="../../pyecharts-assets-master/assets/echarts.min.js"></script>
    <style type="text/css">
        html,body{
            height:100%;
            width:100%
    }
    </style>
</head>
<body onresize="resizeFresh()">
    <div id="1b0c39282f4748ceb082461450acc201" class="chart-container" style="width:95%; height:495%; margin:auto; top:0px"></div>
    <script>
        var chart_1b0c39282f4748ceb082461450acc201 = echarts.init(
            document.getElementById('1b0c39282f4748ceb082461450acc201'), 'light', {renderer: 'canvas'});
        var option_1b0c39282f4748ceb082461450acc201 = {
    "animation": true,
    "animationThreshold": 2000,
    "animationDuration": 1000,
    "animationEasing": "cubicOut",
    "animationDelay": 0,
    "animationDurationUpdate": 300,
    "animationEasingUpdate": "cubicOut",
    "animationDelayUpdate": 0,
    "series": [
        {
            "type": "pie",
            "name": "2019Q1",
            "clockwise": true,
            "data": [
                {
                    "name": "Huawei",
                    "value": 0.34
                },
                {
                    "name": "Vivo",
                    "value": 0.19
                },
                {
                    "name": "Oppo",
                    "value": 0.18
                },
                {
                    "name": "Xiaomi",
                    "value": 0.12
                },
                {
                    "name": "Apple",
                    "value": 0.09
                },
                {
                    "name": "Others",
                    "value": 0.1
                }
            ],
            "radius": "25%",
            "center": [
                "25%",
                "25%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        },
        {
            "type": "pie",
            "name": "2019Q2",
            "clockwise": true,
            "data": [
                {
                    "name": "Huawei",
                    "value": 0.35
                },
                {
                    "name": "Vivo",
                    "value": 0.18
                },
                {
                    "name": "Oppo",
                    "value": 0.19
                },
                {
                    "name": "Xiaomi",
                    "value": 0.11
                },
                {
                    "name": "Apple",
                    "value": 0.06
                },
                {
                    "name": "Others",
                    "value": 0.1
                }
            ],
            "radius": "25%",
            "center": [
                "75%",
                "25%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        },
        {
            "type": "pie",
            "name": "2019Q3",
            "clockwise": true,
            "data": [
                {
                    "name": "Huawei",
                    "value": 0.4
                },
                {
                    "name": "Vivo",
                    "value": 0.19
                },
                {
                    "name": "Oppo",
                    "value": 0.18
                },
                {
                    "name": "Xiaomi",
                    "value": 0.08
                },
                {
                    "name": "Apple",
                    "value": 0.08
                },
                {
                    "name": "Others",
                    "value": 0.1
                }
            ],
            "radius": "25%",
            "center": [
                "25%",
                "75%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        },
        {
            "type": "pie",
            "name": "2019Q4",
            "clockwise": true,
            "data": [
                {
                    "name": "Huawei",
                    "value": 0.35
                },
                {
                    "name": "Vivo",
                    "value": 0.17
                },
                {
                    "name": "Oppo",
                    "value": 0.16
                },
                {
                    "name": "Xiaomi",
                    "value": 0.09
                },
                {
                    "name": "Apple",
                    "value": 0.14
                },
                {
                    "name": "Others",
                    "value": 0.1
                }
            ],
            "radius": "25%",
            "center": [
                "75%",
                "75%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        }
    ],
    "legend": [
        {
            "data": [
                "Huawei",
                "Vivo",
                "Oppo",
                "Xiaomi",
                "Apple",
                "Others"
            ],
            "selected": {},
            "type": "scroll",
            "show": true,
            "left": 0,
            "orient": "horizontal",
            "padding": 5,
            "itemGap": 10,
            "itemWidth": 25,
            "itemHeight": 14
        }
    ],
    "tooltip": {
        "show": true,
        "trigger": "item",
        "triggerOn": "mousemove|click",
        "axisPointer": {
            "type": "line"
        },
        "showContent": true,
        "alwaysShowContent": false,
        "showDelay": 0,
        "hideDelay": 100,
        "textStyle": {
            "fontSize": 14
        },
        "borderWidth": 0,
        "padding": 5
    },
    "title": [
        {
            "padding": 5,
            "itemGap": 10
        }
    ]
};
        chart_1b0c39282f4748ceb082461450acc201.setOption(option_1b0c39282f4748ceb082461450acc201);
    </script>
    <script>   
    var x=window.innerWidth;
    function resizeFresh(){
        if(x!=window.innerWidth)
            location.reload();
    }
    </script>
</body>
</html>


**2020 Q1 - Q4**

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Awesome-pyecharts</title>
    <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,minimum-scale=1.0,user-scalable=yes">
            <script type="text/javascript" src="../../pyecharts-assets-master/assets/echarts.min.js"></script>
    <style type="text/css">
        html,body{
            height:100%;
            width:100%
    }
    </style>
</head>
<body onresize="resizeFresh()">
    <div id="f88c166921aa484b8c5757553af520d3" class="chart-container" style="width:95%; height:495%; margin:auto; top:0px"></div>
    <script>
        var chart_f88c166921aa484b8c5757553af520d3 = echarts.init(
            document.getElementById('f88c166921aa484b8c5757553af520d3'), 'light', {renderer: 'canvas'});
        var option_f88c166921aa484b8c5757553af520d3 = {
    "animation": true,
    "animationThreshold": 2000,
    "animationDuration": 1000,
    "animationEasing": "cubicOut",
    "animationDelay": 0,
    "animationDurationUpdate": 300,
    "animationEasingUpdate": "cubicOut",
    "animationDelayUpdate": 0,
    "series": [
        {
            "type": "pie",
            "name": "2020Q1",
            "clockwise": true,
            "data": [
                {
                    "name": "Huawei",
                    "value": 0.41
                },
                {
                    "name": "Vivo",
                    "value": 0.17
                },
                {
                    "name": "Oppo",
                    "value": 0.15
                },
                {
                    "name": "Xiaomi",
                    "value": 0.11
                },
                {
                    "name": "Apple",
                    "value": 0.09
                },
                {
                    "name": "Others",
                    "value": 0.1
                }
            ],
            "radius": "25%",
            "center": [
                "25%",
                "25%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        },
        {
            "type": "pie",
            "name": "2020Q2",
            "clockwise": true,
            "data": [
                {
                    "name": "Huawei",
                    "value": 0.46
                },
                {
                    "name": "Vivo",
                    "value": 0.16
                },
                {
                    "name": "Oppo",
                    "value": 0.16
                },
                {
                    "name": "Xiaomi",
                    "value": 0.1
                },
                {
                    "name": "Apple",
                    "value": 0.08
                },
                {
                    "name": "Others",
                    "value": 0.0
                }
            ],
            "radius": "25%",
            "center": [
                "75%",
                "25%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        },
        {
            "type": "pie",
            "name": "2020Q3",
            "clockwise": true,
            "data": [
                {
                    "name": "Huawei",
                    "value": 0.43
                },
                {
                    "name": "Vivo",
                    "value": 0.18
                },
                {
                    "name": "Oppo",
                    "value": 0.16
                },
                {
                    "name": "Xiaomi",
                    "value": 0.13
                },
                {
                    "name": "Apple",
                    "value": 0.08
                },
                {
                    "name": "Others",
                    "value": 0.0
                }
            ],
            "radius": "25%",
            "center": [
                "25%",
                "75%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        },
        {
            "type": "pie",
            "name": "2020Q4",
            "clockwise": true,
            "data": [
                {
                    "name": "Huawei",
                    "value": 0.3
                },
                {
                    "name": "Vivo",
                    "value": 0.18
                },
                {
                    "name": "Oppo",
                    "value": 0.16
                },
                {
                    "name": "Xiaomi",
                    "value": 0.13
                },
                {
                    "name": "Apple",
                    "value": 0.08
                },
                {
                    "name": "Others",
                    "value": 0.2
                }
            ],
            "radius": "25%",
            "center": [
                "75%",
                "75%"
            ],
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        }
    ],
    "legend": [
        {
            "data": [
                "Huawei",
                "Vivo",
                "Oppo",
                "Xiaomi",
                "Apple",
                "Others"
            ],
            "selected": {},
            "type": "scroll",
            "show": true,
            "left": 0,
            "orient": "horizontal",
            "padding": 5,
            "itemGap": 10,
            "itemWidth": 25,
            "itemHeight": 14
        }
    ],
    "tooltip": {
        "show": true,
        "trigger": "item",
        "triggerOn": "mousemove|click",
        "axisPointer": {
            "type": "line"
        },
        "showContent": true,
        "alwaysShowContent": false,
        "showDelay": 0,
        "hideDelay": 100,
        "textStyle": {
            "fontSize": 14
        },
        "borderWidth": 0,
        "padding": 5
    },
    "title": [
        {
            "padding": 5,
            "itemGap": 10
        }
    ]
};
        chart_f88c166921aa484b8c5757553af520d3.setOption(option_f88c166921aa484b8c5757553af520d3);
    </script>
    <script>   
    var x=window.innerWidth;
    function resizeFresh(){
        if(x!=window.innerWidth)
            location.reload();
    }
    </script>
</body>
</html>

**2021 Q1**

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Awesome-pyecharts</title>
    <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,minimum-scale=1.0,user-scalable=yes">
            <script type="text/javascript" src="../../pyecharts-assets-master/assets/echarts.min.js"></script>
    <style type="text/css">
        html,body{
            height:100%;
            width:100%
    }
    </style>
</head>
<body onresize="resizeFresh()">
    <div id="a5130f29eb0a4159a0af97ec3301f27d" class="chart-container" style="width:95%; height:495%; margin:auto; top:0px"></div>
    <script>
        var chart_a5130f29eb0a4159a0af97ec3301f27d = echarts.init(
            document.getElementById('a5130f29eb0a4159a0af97ec3301f27d'), 'light', {renderer: 'canvas'});
        var option_a5130f29eb0a4159a0af97ec3301f27d = {
    "animation": true,
    "animationThreshold": 2000,
    "animationDuration": 1000,
    "animationEasing": "cubicOut",
    "animationDelay": 0,
    "animationDurationUpdate": 300,
    "animationEasingUpdate": "cubicOut",
    "animationDelayUpdate": 0,
    "series": [
        {
            "type": "pie",
            "name": "2021Q1",
            "clockwise": true,
            "data": [
                {
                    "name": "Huawei",
                    "value": 0.16
                },
                {
                    "name": "Vivo",
                    "value": 0.23
                },
                {
                    "name": "Oppo",
                    "value": 0.22
                },
                {
                    "name": "Xiaomi",
                    "value": 0.15
                },
                {
                    "name": "Apple",
                    "value": 0.13
                },
                {
                    "name": "Others",
                    "value": 0.1
                }
            ],
            "radius": "75%",
            "center": [
                "50%",
                "50%"
            ],
            "roseType": "area",
            "label": {
                "show": true,
                "position": "top",
                "color": "rgba(0, 0, 1, 0.8)",
                "margin": 8
            },
            "tooltip": {
                "show": true,
                "trigger": "item",
                "triggerOn": "mousemove|click",
                "axisPointer": {
                    "type": "line"
                },
                "showContent": true,
                "alwaysShowContent": false,
                "showDelay": 0,
                "hideDelay": 100,
                "formatter": "{b} <br/>{a}: {c}",
                "textStyle": {
                    "fontSize": 14
                },
                "borderWidth": 0,
                "padding": 5
            },
            "rippleEffect": {
                "show": true,
                "brushType": "stroke",
                "scale": 2.5,
                "period": 4
            }
        }
    ],
    "legend": [
        {
            "data": [
                "Huawei",
                "Vivo",
                "Oppo",
                "Xiaomi",
                "Apple",
                "Others"
            ],
            "selected": {},
            "type": "scroll",
            "show": true,
            "left": 0,
            "orient": "horizontal",
            "padding": 5,
            "itemGap": 10,
            "itemWidth": 25,
            "itemHeight": 14
        }
    ],
    "tooltip": {
        "show": true,
        "trigger": "item",
        "triggerOn": "mousemove|click",
        "axisPointer": {
            "type": "line"
        },
        "showContent": true,
        "alwaysShowContent": false,
        "showDelay": 0,
        "hideDelay": 100,
        "textStyle": {
            "fontSize": 14
        },
        "borderWidth": 0,
        "padding": 5
    },
    "title": [
        {
            "padding": 5,
            "itemGap": 10
        }
    ]
};
        chart_a5130f29eb0a4159a0af97ec3301f27d.setOption(option_a5130f29eb0a4159a0af97ec3301f27d);
    </script>
    <script>   
    var x=window.innerWidth;
    function resizeFresh(){
        if(x!=window.innerWidth)
            location.reload();
    }
    </script>
</body>
</html>

### 小米公司财报情况
```Python
FR = pd.read_csv('Financial_Report_Xiaomi.csv')
FR = FR.sort_values(by='年度')
FR
```


### 全球市场Top国家/地区可视化
```Python
Global_top5 = pd.read_csv('Global_Top5.csv')

from pyecharts.charts import Map, Geo

value = Global_top5['rank'].tolist()
attr = Global_top5['country'].tolist()
map = Map( init_opts=opts.InitOpts(width="1900px", height="900px", bg_color="#ADD8E6",
                                       page_title="-",theme=ThemeType.LIGHT))

map.add("Rank",[list(z) for z in zip(attr, value)],is_map_symbol_show=False,
    maptype="world",label_opts=opts.LabelOpts(is_show=False),)

map.set_global_opts(title_opts = opts.TitleOpts(title='全球销量Top5国家和地区（不完全统计，至2021/05）'),
    legend_opts=opts.LegendOpts(is_show=False),
    visualmap_opts = opts.VisualMapOpts(min_=1,max_=5,range_color=map.colors))

map.render('map.html')
```

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>-</title>
    <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,minimum-scale=1.0,user-scalable=yes">
            <script type="text/javascript" src="../../pyecharts-assets-master/assets/echarts.min.js"></script>
        <script type="text/javascript" src="https://assets.pyecharts.org/assets/maps/world.js"></script>
    <style type="text/css">
        html,body{
            height:100%;
            width:100%
    }
    </style>
</head>
<body onresize="resizeFresh()">
    <div id="738b6258e1184f95a31b7f9acb9cf69a" class="chart-container" style="width:100%; height:495%; margin:auto; top:0px"></div>
    <script>
        var chart_738b6258e1184f95a31b7f9acb9cf69a = echarts.init(
            document.getElementById('738b6258e1184f95a31b7f9acb9cf69a'), 'light', {renderer: 'canvas'});
        var option_738b6258e1184f95a31b7f9acb9cf69a = {
    "backgroundColor": "#ADD8E6",
    "animation": true,
    "animationThreshold": 2000,
    "animationDuration": 1000,
    "animationEasing": "cubicOut",
    "animationDelay": 0,
    "animationDurationUpdate": 300,
    "animationEasingUpdate": "cubicOut",
    "animationDelayUpdate": 0,
    "series": [
        {
            "type": "map",
            "name": "Rank",
            "label": {
                "show": false,
                "position": "top",
                "margin": 8
            },
            "mapType": "world",
            "data": [
                {
                    "name": "Belarus",
                    "value": 1
                },
                {
                    "name": "Greece",
                    "value": 1
                },
                {
                    "name": "India",
                    "value": 1
                },
                {
                    "name": "Myanmar",
                    "value": 1
                },
                {
                    "name": "Poland",
                    "value": 1
                },
                {
                    "name": "Spain",
                    "value": 1
                },
                {
                    "name": "Ukraine",
                    "value": 1
                },
                {
                    "name": "Croatia",
                    "value": 1
                },
                {
                    "name": "France",
                    "value": 2
                },
                {
                    "name": "Latvia",
                    "value": 2
                },
                {
                    "name": "Nepal",
                    "value": 1
                },
                {
                    "name": "Russia",
                    "value": 1
                },
                {
                    "name": "Slovakia",
                    "value": 2
                },
                {
                    "name": "Austria",
                    "value": 3
                },
                {
                    "name": "Cambodia",
                    "value": 3
                },
                {
                    "name": "Hungary",
                    "value": 3
                },
                {
                    "name": "Israel",
                    "value": 3
                },
                {
                    "name": "Laos",
                    "value": 3
                },
                {
                    "name": "Lithuania",
                    "value": 3
                },
                {
                    "name": "Nigeria",
                    "value": 3
                },
                {
                    "name": "Peru",
                    "value": 3
                },
                {
                    "name": "Portugal",
                    "value": 3
                },
                {
                    "name": "Qatar",
                    "value": 3
                },
                {
                    "name": "Sweden",
                    "value": 3
                },
                {
                    "name": "Turkey",
                    "value": 3
                },
                {
                    "name": "China",
                    "value": 4
                },
                {
                    "name": "Colombia",
                    "value": 4
                },
                {
                    "name": "Czech Republic",
                    "value": 4
                },
                {
                    "name": "Egypt",
                    "value": 4
                },
                {
                    "name": "Germany",
                    "value": 4
                },
                {
                    "name": "Indonesia",
                    "value": 4
                },
                {
                    "name": "Italy",
                    "value": 4
                },
                {
                    "name": "Kenya",
                    "value": 4
                },
                {
                    "name": "Kuwait",
                    "value": 4
                },
                {
                    "name": "Netherlands",
                    "value": 4
                },
                {
                    "name": "Romania",
                    "value": 4
                },
                {
                    "name": "Saudi Arabia",
                    "value": 4
                },
                {
                    "name": "Slovenia",
                    "value": 4
                },
                {
                    "name": "South Korea",
                    "value": 4
                },
                {
                    "name": "United Arab Emirates",
                    "value": 4
                },
                {
                    "name": "Brazil",
                    "value": 5
                },
                {
                    "name": "Chile",
                    "value": 5
                },
                {
                    "name": "Estonia",
                    "value": 5
                },
                {
                    "name": "Malaysia",
                    "value": 5
                },
                {
                    "name": "Mexico",
                    "value": 5
                },
                {
                    "name": "Singapore",
                    "value": 5
                },
                {
                    "name": "Sri Lanka",
                    "value": 5
                },
                {
                    "name": "Switzerland",
                    "value": 5
                }
            ],
            "roam": true,
            "aspectScale": 0.75,
            "nameProperty": "name",
            "selectedMode": false,
            "zoom": 1,
            "mapValueCalculation": "sum",
            "showLegendSymbol": false,
            "emphasis": {}
        }
    ],
    "legend": [
        {
            "data": [
                "Rank"
            ],
            "selected": {
                "Rank": true
            },
            "show": false,
            "padding": 5,
            "itemGap": 10,
            "itemWidth": 25,
            "itemHeight": 14
        }
    ],
    "tooltip": {
        "show": true,
        "trigger": "item",
        "triggerOn": "mousemove|click",
        "axisPointer": {
            "type": "line"
        },
        "showContent": true,
        "alwaysShowContent": false,
        "showDelay": 0,
        "hideDelay": 100,
        "textStyle": {
            "fontSize": 14
        },
        "borderWidth": 0,
        "padding": 5
    },
    "title": [
        {
            "text": "\u5168\u7403\u9500\u91cfTop5\u56fd\u5bb6\u548c\u5730\u533a\uff08\u4e0d\u5b8c\u5168\u7edf\u8ba1\uff0c\u81f32021/05\uff09",
            "padding": 5,
            "itemGap": 10
        }
    ],
    "visualMap": {
        "show": true,
        "type": "continuous",
        "min": 1,
        "max": 5,
        "inRange": {
            "color": [
                "#c23531",
                "#2f4554",
                "#61a0a8",
                "#d48265",
                "#749f83",
                "#ca8622",
                "#bda29a",
                "#6e7074",
                "#546570",
                "#c4ccd3",
                "#f05b72",
                "#ef5b9c",
                "#f47920",
                "#905a3d",
                "#fab27b",
                "#2a5caa",
                "#444693",
                "#726930",
                "#b2d235",
                "#6d8346",
                "#ac6767",
                "#1d953f",
                "#6950a1",
                "#918597"
            ]
        },
        "calculable": true,
        "inverse": false,
        "splitNumber": 5,
        "orient": "vertical",
        "showLabel": true,
        "itemWidth": 20,
        "itemHeight": 140,
        "borderWidth": 0
    }
};
        chart_738b6258e1184f95a31b7f9acb9cf69a.setOption(option_738b6258e1184f95a31b7f9acb9cf69a);
    </script>
    <script>   
    var x=window.innerWidth;
    function resizeFresh(){
        if(x!=window.innerWidth)
            location.reload();
    }
    </script>
</body>
</html>
