---
layout: post
title:  Echarts使用笔记
date:   2017-06-11 11:53:00 +0800
categories: 建站
tag: 教程 Eharts
author: Michael Wang
source_id: 170611115300
---

* content
{:toc}

### x轴项目名称处理
x轴每项的名称过长时，可以通过倾斜项目名称，且项目名称可以通过部分名称取代全程时，可以在xAxis-axisLabel-formatter中进行格式的设置。

``` 

	option = {
	    title : {
	        text: '',
	        top : 'bottom',
	        left : 'center',
	        textStyle: {
	        color: '#333',
	        fontStyle: 'normal',
	        fontWeight: 'lighter',
	        fontFamily: 'sans-serif',
	        fontSize: 15,
	        },
	    },
	    tooltip : {
	        trigger: 'axis',
	        axisPointer: {
	            type: 'shadow'
	        }       
	    },
	    xAxis : [
	        {
	            axisLabel: {
	                rotate: 315,//倾斜角度
	                formatter: function(value){//对显示的内容进行设置
	                    var val;
	                    val = value.substring(0,4);
	                    return val;
	                }
	            },
	            type : 'category',
	            data : [],
	            axisTick: {
	                alignWithLabel: true
	            }
	        }
	    ],
	    yAxis : [
	        {
	            type : 'value'
	        }
	    ],
	    series : [
	        {
	            label: {
			                normal: {
			                    show: true,
			                    position: 'top',
			                    formatter: function(obj){
			                        return obj.value.toFixed(2);
			                    },
			                },
			            },
	            name:'直接访问',
	            type:'bar',
	            barWidth: '60%',
	            data:[]
	        }
	    ]
	};
```
### 配置多个series
当需要设置多个series时（比如折线图中有多条不同的曲线，每条曲线为一个series），可以先设置变量`var series = [];`，从后台取得数据后，对每个series进行赋值、设置；对于`legend`同理。

```

	var myChart = echarts.init(document.getElementById('echarts_id'));
	var series=[];
	var option = {
		//其他配置项...
		series: series,
		xAxis:
		//其他配置项...
	}
	myChart.setOption(option);
	//ajax 异步加载配置数据项  
	function loadOption(item, unit, time)  
	{
	    $.ajax({  
	        url: '',
	        type: 'get',
	        dataType: 'json',
	        async: false,
	        success: function (dataMap) {  
	            if (dataMap)  
	            {  
	                if (dataMap.seriesData == null) {  
	                    series = [];
						option.xAxis[0].data = [];//xAxis配置项为数组时，即xAxis后是中括号[]，若为{}，则应该写成option.xAxis.data = [];
	                }  
	                else {                 	
	                	for(var i = 0;i < dataMap.seriesName.length;i++){
					       series.push({
					            name: dataMap.seriesName[i],
					            type: 'line',
					            smooth: true,
					            data: dataMap.seriesData[i]
					        });
					    } 
	                } 
	                myChart.hideLoading();
	                myChart.setOption(option,true);  
	                series=[];//将series变量置空，否则之后的结果会在之前的结果集合上累加
	            }  
	        },  
	        error: function ()  
	        {  
	            alert("请求失败!");  
	        }  
	    })  
	}

```