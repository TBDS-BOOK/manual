# 进度监控

## 介绍

进度监控是用于按照不同维度统计数据接入的各个topic和接口数据接入的指标，用于监控agent、hippo、sort不同组件的数据发送量。进度监控分为数据趋势、数据明细和累加统计三部分。进度监控的入口在数据接入

![](/数据接入/进度监控/imges/progress_moniter_enter.png)

进入之后选择进度监控模块

![](/数据接入/进度监控/imges/progress_moniter_step1.png)

## 数据趋势

数据趋势是展现各个topic在agent、hippo、sort不同组件上每分钟数据指标。数据趋势分为分钟趋势和累计趋势。

* 分钟趋势

点击分钟趋势图，选择好topic和时间范围即可查询该topic在各个组件上的数据指标。

![](/数据接入/进度监控/imges/progress_moniter_step2.png)

* 累计趋势

展现某个topic在各个组件上的累计统计指标

![](/数据接入/进度监控/imges/progress_moniter_step3.png)

* 数据下钻

数据下钻是按照更加细粒度查看指标，目前系统提供了按照host和接口两个维度下钻。按照host下钻是查看agent向某个hippo主机发送数据量、hippo自己接收的数据量以及sort从hippo消费的数据量；按照接口下钻是细分到topic中某个具体的表的指标。在图中点击下钻进入下钻界面

![](/数据接入/进度监控/imges/progress_moniter_step4.png)

假设选择按照接口下钻

1.选择接口类型

![](/数据接入/进度监控/imges/progress_moniter_step5.png)

2.选择具体的接口

![](/数据接入/进度监控/imges/progress_moniter_step6.png)

## 数据明细

    数据明细是从条数、包数、字节数三个维度展现某个topic每分钟各个组件上的指标数据，同时也提供下钻功能，按照host和接口细分查询。

1.点击左侧的数据明细图标，进入数据明细界面

![](/数据接入/进度监控/imges/progress_moniter_step7.png)

2.以数据接入条数为例，选择某个topic及时间范围，可以查询该topic每分钟在各个组件上的指标。

![](/数据接入/进度监控/imges/progress_moniter_step8.png)

选择异常指标则过滤展现该时间段内agent-hippo-sort对不齐的指标

![](/数据接入/进度监控/imges/progress_moniter_step9.png)

3.下钻功能和前面提到的类似，这里不再赘述。

## 累加统计

累加统计是用于统计sort向后端系统真实的输出数据，在左侧点击累加统计图标进入累加统计界面。

选择某个topic及时间范围可以查看该topic在这个时间段输出的数据量。

下钻是按照接口查看具体某个接口的数据输出情况

