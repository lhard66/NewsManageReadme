### 二、行情管理

#### 1.添加经销商

Title|Link|City|SteelName|AddDate|IsTop|Remark|isrecommend
---|---|---|---|---|---|---|---
标题|链接|地区|钢材类型|添加时间|排序|备注(默认为空)|是否推荐

#### 2.经销商管理
* 此页面展示数据为调用存储过程直接返回

#### 3.添加价格汇总

#### 4.价格汇总列表

* 此页面加载时是通过存储过程调取数据，表名为：Summaries

#### 5.价格汇总分类

* 页面加载时调取数据的表：

id|typename|fid|itemvalue 
---|---|---|---
ID|分类名|子分类对应的父分类ID|下拉框项目的Value值

#### 6.行情查询

*页面加载时查询pricelist表，内容如下：

id|cityid|steeltypeid|pricedetail|pricefrom|pricehash|createdate|createtime|softversion
---|---|---|---|---|---|---|---|---
ID|城市ID|品名ID|价格|价格来源|涨跌|日期|时间|版本

#### 7.行情订阅查询

* 点击“查询”按钮时，会执行此SQL语句：
```
select * from SmsSubscribes where endtime>getdate()
```
执行后，拿到表结构数据，直接赋值给Repeater对象的数据源。