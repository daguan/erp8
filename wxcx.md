# 微信超级查询
* ESAP提供了快速对接微信公众号/企业号的回调接口，自定义SQL语句，自定义权限，实现引擎式超级查询。
* 需配置应用的token和EncodingAesKey，回调URL规则：
 - 企业号(企业微信)：http://外网host/wx/应用名
 - 服务号(订阅号)：http://外网host/wxs/应用名

回调示例:
 - 企业号：http://io.erp8.net:9090/wx/esap
 - 服务号：http://io.erp8.net:9090/wxs/esap

> 注意，公众号(服务号/订阅号)必须使用80端口，如无80端口可以用nginx或caddy做反向代理。

## 微信查询原理
* 系统根据用户输入的第一个关键字（例如下图的`库存`），扫描esap_cx表中对应的sql，执行并返回结果。
* 其他关键字被视为查询参数（例如下图中的`名`,`手机`），sql中可以使用`:p1,:p2`替换where条件。

> select 品号,名 as 品名,库存 from 库存表 where {\{.p1\}} like '%{\{.p2\}}%'

![](./img/s2.png)

**微信查询使用的必要前提是会sql，请自行学习！**

- [动态模板](#动态模板)
 - [参数解释](#参数解释)
 - [进入提示](#进入提示) <span style="color:red">new!</span>
 - [多重查询](#多重查询) <span style="color:red">new!</span>
 - [文章返回](#文章返回) <span style="color:red">new!</span>
 - [UUID](#UUID)
- [多重权限](#多重权限)
- [企业号通讯录变量](#企业号通讯录变量)
- [语音查询](#语音查询)
- [扫码查询](#扫码查询)
- [多媒体查询](#多媒体查询)
- [回写数据](#回写数据)
- [专用查询](#专用查询)
- [菜单查询](#菜单查询)
- [*数据字典](#数据字典)


#### 动态模板
* 支持golang模板语法，函数，可实现动态参数和动态sql

**[点击这里参考更多用法](https://app.esap.vip/admin#/wxcx)**

![](./img/8.27.png)

<img src="./img/8.35.jpg" width="240">
<img src="./img/8.36.jpg" width="240">
<img src="./img/8.37.jpg" width="240">

##### 参数解释

* 用户的输入(逗号空格分隔)会被解析为p0,p1,p2...pn,以及一个特别的P(大写)代表所有输入内容

例如用户输入：`库存，手机，一号仓` 会被解析为参数组：`p0=库存，p1=手机，p2=一号仓,P=库存，手机，一号仓`

* 在sql中使用\{\{.pn\}\}作参数替换，如果跟在等号（=）后面作为值替换，可以直接使用:pn，例如： 

> select 姓名,工号 from \{\{.p0\}\} where 姓名=:p1

* 要注意的是golang原生模板\{\{.pn\}\}形式用在文本型值位置应该加上单引号，以免被sql误认为是字段，而`:pn`形式则不需要。

所以上面的语句等价于： 
> select 姓名,工号 from \{\{.p0\}\} where 姓名='\{\{.p1\}\}'


##### 进入提示
* esap.yml配置entermsg=true

* 企业号应用回调勾选用户进入提示

* esap_cx表中的entermsg可写select来作为进入提醒。

![](./img/s2-1.png)
 
##### 多重查询
* 可以写多个select语句，这些语句结果都能返回（sql2005+）。

![](./img/s2-2.png)
<img src="./img/s2-3.jpg" width="240">
 
##### 文章返回
* url有值时触发,url可使用参数{\{.pn\}}。

![](./img/wxcx-1.png)
<img src="./img/wxcx-2.jpg" width="240">
<img src="./img/wxcx-3.jpg" width="240">

##### UUID
* \{\{uuid\}\} 对应一个全球唯一的id, 可用于构建自动编号。

#### 多重权限
用逗号隔开，可用姓名，账号，部门，应用随意组合。

#### 企业号通讯录变量
* \{\{姓名\}\} 对应 `姓名`
* \{\{账号\}\} 对应 `账号`
* \{\{部门\}\} 对应 `部门`
* \{\{职位\}\} 对应 `职位`
* \{\{性别\}\} 对应 `性别`
* \{\{手机\}\} 对应 `手机号`
* \{\{邮箱\}\} 对应 `邮箱`

#### 语音查询
<img src="./img/8.13.png" width="240">
<img src="./img/8.14.png" width="240">
<img src="./img/8.15.png" width="240">

#### 扫码查询
二维码或其他条码等,esap_cx表中的mkey与自定义菜单的扫码弹框菜单key对应即可

<img src="./img/8.16.png" width="240">
<img src="./img/8.6.jpg" width="240">
<img src="./img/8.18.png" width="240">

#### 多媒体查询
* sql结果字段或别名为：
 * `pic`或`图片`
 * `files`或`附件`
 * `voice`或`语音`
 * `video`或`视频`

* 支持返回多个`图片`或`附件`

<img src="./img/8.21.jpg" width="240">
<img src="./img/8.22.jpg" width="240">
<img src="./img/mutimedia-2.jpg" width="240">

> 注意微信限制：图片一般不能超过2M,附件不超过20M

#### 回写数据
sql中使用update或insert，最后一句可使用select语句提示用户，例如：

```sql
insert xxx(...) values(...)
...
select N'你的信息已成功提交，谢谢使用'
```

<img src="./img/8.24.jpg" width="240">
![](./img/8.23.png)

#### 专用查询
* 专用查询字段(app)为配置应用名（appName），查询不再需要关键字引导，该应用也不再响应其他关键字，仅需输入参数就可查询。

![](./img/8.40.jpg)
<img src="./img/8.38.jpg" width="240">

* 专用查询可直接扫二维码秒查。

![](./img/8.42.png)
![](./img/8.41.png)
<img src="./img/8.43.jpg" width="240">
<img src="./img/8.44.jpg" width="240">

#### 菜单查询
设置mkey，与企业号应用自定义菜单中的key一致。

<img src="./img/8.27.jpg" width="320">
<img src="./img/8.28.jpg" width="320">

## 数据字典
微信查询主要扫描`esap_cx`表，对应字段解析如下：

|字段|描述|必填|备注|
|:----:|:--:|:--:|:----|
|mKey|微信自定义菜单id|否|绑定菜单id|
|name|查询名称|是|绑定查询第一个关键字|
|entermsg|进入提醒|否|可以是一个select语句|
|tmpl|sql模板|是||
|aclUser|用户权限|否|@all代表全体可用|
|aclDept|部门权限|否|逗号隔开多个ID或名称|
|aclTag|标签权限|否|逗号隔开多个ID或名称|
|aclApp|应用权限|是|逗号(小写)隔开多个配置应用名或用@all|
|mode|模式|否|1代表仅返回图片或附件|
|app|专用查询|否|绑定一个配置的应用名,即专用查询|
|db|数据库名称|否|跨账套|
|url|文章URL|否|有值时返回文章，可使用{\{.pn\}}参数|
|pic|文章封面图片|否||
|safe|保密消息模式|否|1表示保密|
|id|自增编号|否|唯一|

* sql模板位置：sql/esap/wxcx.get