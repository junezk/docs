#  Ajax请求服务器数据动态刷新HighCharts表格



```javascript
<script type="text/javascript">  
    $(function() {  
        var realtime = new Array(10);  
        var realvalue = new Array(10);  
        $(document)  
                .ready(  
                        function() {  
                            Highcharts.setOptions({  
                                global : {  
                                    useUTC : false  
                                }  
                            });  
                            var chart;  
                            chart = new Highcharts.Chart(  
                                    {  
                                        chart : {  
                                            renderTo : 'container',  
                                            type : 'spline',  
                                            animation : Highcharts.svg,  
                                            // don't animate in old IE                 
                                            marginRight : 10,  
                                            events : {  
                                                load : function() {  
                                                }  
                                            }  
                                        },  
                                        title : {  
                                            text : 'Sensor\'s Dynamic Temperature'  
                                        },  
                                        subtitle : {  
                                            text : 'Source: inteli405',  
                                            x : -20  
                                        },  
                                        xAxis : {  
                                            type : 'datetime',  
                                            tickPixelInterval : 150  
                                        },  
                                        yAxis : [ {  
                                            title : {  
                                                text : 'Temperature'  
                                            },  
                                            plotLines : [ {  
                                                value : 0,  
                                                width : 1,  
                                                color : '#808080'  
                                            } ]  
                                        }, {  
                                            title : {  
                                                text : '摄氏度'  
                                            },  
                                            plotLines : [ {  
                                                value : 0,  
                                                width : 1,  
                                                color : '#808080'  
                                            } ]  
                                        } ],  
                                        tooltip : {  
                                            formatter : function() {  
                                                return '<b>'  
                                                        + this.series.name  
                                                        + '</b><br/>'  
                                                        + Highcharts  
                                                                .dateFormat(  
                                                                        '%Y-%m-%d %H:%M:%S',  
                                                                        this.x)  
                                                        + '<br/>'  
                                                        + Highcharts  
                                                                .numberFormat(  
                                                                        this.y,  
                                                                        2);  
                                            }  
                                        },  
                                        legend : {  
                                            enabled : false  
                                        },  
                                        exporting : {  
                                            enabled : false  
                                        },  
                                        series : [  
                                                {  
                                                    name : 'Sensor Data',  
                                                    data : (function() { // generate an array of random data                               
                                                        var data = [], time = (new Date())  
                                                                .getTime(), i;  
                                                        for (i = -19; i <= 0; i++) {  
                                                            data.push({  
                                                                x : time + i  
                                                                        * 1000,  
                                                                y : 0  
                                                            });  
                                                        }  
                                                        return data;  
                                                    })()  
                                                },  
                                                {  
                                                    name : 'Sensor Data',  
                                                    data : (function() { // generate an array of random data                               
                                                        var data = [], time = (new Date())  
                                                                .getTime(), i;  
                                                        for (i = -19; i <= 0; i++) {  
                                                            data.push({  
                                                                x : time + i  
                                                                        * 1000,  
                                                                y : 0  
                                                            });  
                                                        }  
                                                        return data;  
                                                    })()  
                                                } ]  
                                    }); // set up the updating of the chart each second                       
                            setInterval(  
                                    function() {  
  
                                        var series = chart.series[0];  
                                        var series1 = chart.series[1];  
                                        $  
                                                .ajax({  
                                                    url : "//192.168.199.187/statistic/Temperature/history",  
                                                    type : "GET",//这里是AJAX请求的方式  
                                                    dataType : "JSON",//如果你回发的内容是JSON格式的就用这个，否则用Text或其他  
                                                    //async: false,  
                                                    success : function(data) {  
                                                        var myobj = eval(JSON  
                                                                .stringify(data));  
                                                        var latest = 0;  
  
                                                        var unixTimestamp = new Date(  
                                                                myobj[latest].timestamp);  
                                                        var commonTime = unixTimestamp  
                                                                .toLocaleString();  
  
                                                        var value = myobj[latest].value;  
  
                                                        for ( var i = 0; i < 1; i++) {  
                                                            unixTimestamp = new Date(  
                                                                    myobj[i].timestamp);  
                                                            realtime[i] = unixTimestamp  
                                                                    .toLocaleString();  
                                                            realvalue[i] = myobj[i].value;  
                                                        }  
  
                                                        var x = (new Date())  
                                                                .getTime(), y = realvalue[0];  
                                                        series.addPoint(  
                                                                [ x, y ], true,  
                                                                true);  
                                                        series1.addPoint(  
                                                                [ x, y ], true,  
                                                                true);  
                                                    },  
                                                    error : function(  
                                                            XMLHttpRequest,  
                                                            Error, F) {  
                                                        //出错后可以在这里给出提示，Error参数表示错误信息  
                                                    }  
                                                });  
                                    }, 3000);  
                        });  
    });  
</script>
```

