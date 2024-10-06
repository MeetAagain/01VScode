一、现在有一份数据mingxing_favor表
Id	Name			Age		Favor
1	Huangbo	33	A,B,C,D,E
2	Xuzheng	44	B,C
3	Wangbaoqiang	55	C,D,E
4	Fanbingbing	32	A,B,D
求出每种爱好中年龄最大的人，如果有相同的年龄，请并列显示
CREATE TABLE mingxing_favor
(
id int,
name string,
age int,
favor string)
row format delimited
fields terminated by '\t';

load 数据：
load data local inpath "/home/hivedata/mingxing.txt" into table mingxing_favor;
-- 编写sql
-- 首先把爱好展开
select id,name,age,ah from mingxing_favor lateral view explode(split(favor,',')) aihao as ah;

id      name    age     ah
1       Huangbo 33      A
1       Huangbo 33      B
1       Huangbo 33      C
1       Huangbo 33      D
1       Huangbo 33      E
2       Xuzheng 44      B
2       Xuzheng 44      C
3       Wangbaoqiang    55      C
3       Wangbaoqiang    55      D
3       Wangbaoqiang    55      E
4       Fanbingbing     32      A
4       Fanbingbing     32      B
4       Fanbingbing     32      D

select *,dense_rank() over(partition by ah order by age desc) as paiming from (
select id,name,age,ah  from mingxing_favor 
lateral view explode(split(favor,',')) 
mytable as ah ) a ;

a.id    a.name  a.age   a.ah    paiming
3       Wangbaoqiang    55      E       1
1       Huangbo 33      E       2
1       Huangbo 33      A       1
4       Fanbingbing     32      A       2
2       Xuzheng 44      B       1
1       Huangbo 33      B       2
4       Fanbingbing     32      B       3
3       Wangbaoqiang    55      C       1
2       Xuzheng 44      C       2
1       Huangbo 33      C       3
3       Wangbaoqiang    55      D       1
1       Huangbo 33      D       2
4       Fanbingbing     32      D       3

select ah,age,concat_ws('|',collect_list(name)) from (
select *,dense_rank() over(partition by ah order by age desc) as paiming from (
select id,name,age,ah  from mingxing_favor 
lateral view explode(split(favor,',')) 
mytable as ah ) a ) b where paiming = 1 group by ah,age;


结果：
D       55      Wangbaoqiang
E       55      Wangbaoqiang
A       33      Huangbo
B       44      Xuzheng
C       55      Wangbaoqiang
-- 最后将如果年龄相同，名字就合并在一起   Huangbo |  xiebingbing
假如再追加一条数据，
1       Huangbo 33      A,B,C,D,E
2       Xuzheng 44      B,C
3       Wangbaoqiang    55      C,D,E
4       Fanbingbing     32      A,B,D
5       WuJing  55      C,D,F
结果如下：
D       55      Wangbaoqiang|WuJing
E       55      Wangbaoqiang
A       33      Huangbo
B       44      Xuzheng
C       55      WuJing|Wangbaoqiang
F       55      WuJing
二、我们有如下的用户访问数据
userId  visitDate   visitCount
u01 2017/1/21 5
u02 2017/1/23 6
u03 2017/1/22 8
u04 2017/1/20 3
u01 2017/1/23 6
u01 2017/2/21 8
u02 2017/1/23 6
u01 2017/2/22 4
要求使用SQL统计出每个用户每个月的累积访问次数，如下表所示
用户id	月份	小计	累积
u01	2017-01	11	11
u01	2017-02	12	23
u02	2017-01	12	12
u03	2017-01	8	8
u04	2017-01	3	3
-- 复习：
select unix_timestamp(); //获取当前的时间戳
select unix_timestamp('2017/1/21','yyyy/MM/dd') ;
根据时间戳，指定输出日期的格式：
select from_unixtime(1603843200);
select from_unixtime(1603843200,'yyyy-MM-dd');

unix_timestamp -->  是将一个字符串，变为了一个时间戳
from_unixtime  --> 将时间戳变为了一个字符串

-- 加载数据
create table visitors(
  userId string,
  visitDate string,
  visitCount int) 
  row format delimited fields terminated by ' ';
  
  load data local inpath "/home/hivedata/visit.txt" overwrite into table visitors;
 
-- 首先将字符串时间修改为时间戳，然后获取年月信息
 select from_unixtime(unix_timestamp(visitDate,'yyyy/MM/dd'),'yyyy-MM') from visitors;

2017-01
2017-01
2017-01
2017-01
2017-01
2017-02
2017-01
2017-02

-- 按照用户和年月分组
select userId,month,sum(visitCount) from (
  select *,
  from_unixtime(unix_timestamp
  (visitdate,'yyyy/MM/dd'),'yyyy-MM') as month
  from visitors 
  ) a group by userId,month;

  
-- 最终的sql
select *,sum(cs) over(partition by userId order by month ) from (
select userId,month,sum(visitCount) cs from (
  select *,
  from_unixtime(unix_timestamp
  (visitdate,'yyyy/MM/dd'),'yyyy-MM') as month
  from visitors 
  ) a group by userId,month ) b  order by userId ;

  order by 自带累加效果
三、有很多的天猫店铺，每个顾客访客访问任何一个店铺的任何一个商品时都会产生一条访问日志 
访问日志存储的表名为Visit，访客的用户id为user_id，被访问的店铺名称为shop，数据如下
u1	a
u2	b
u1	b
u1	a
u3	c
u4	b
u1	a
u2	c
u5	b
u4	b
u6	c
u2	c
u1	b
u2	a
u2	a
u3	a
u5	a
u5	a
u5	a
(1)每个店铺的UV（访客数） 一个客户不管来多少回，都只算一回。
创建表，导入数据
create table tianmao(
   user_id string,
   shop string
  ) 
  row format delimited 
  fields terminated by '\t';


load data local inpath "/home/hivedata/shop.txt" into table tianmao;

-- 第一题 每个店铺的UV（访客数）   一个客户不管来多少回，都只算一回。
-- 第一种写法：
select shop,count(distinct user_id) from tianmao group by shop;

帅岩的写法：
select shop,size(collect_set(userid)) from tianmao group by shop;

-- 第三种写法
select shop,user_id from tianmao group by shop,user_id;

select shop,count(1) from  
   (select shop,user_id from tianmao group by shop,user_id)  a
   group by shop ;
(2)每个店铺访问次数top3的访客信息。输出店铺名称、访客id、访问次数
select shop,user_id,count(1) from tianmao group by shop,user_id;

排名

select *,dense_rank() over(partition by shop order by cs desc ) paiming from (

  select shop,user_id,count(1) cs from tianmao group by shop,user_id
) a ;

取前三名：
select * from (

  select *,dense_rank() over(partition by shop order by cs desc ) paiming from (

  select shop,user_id,count(1) cs from tianmao group by shop,user_id
) a 
  
) b where paiming < 4;
四、Employee 表包含所有员工和他们的经理。每个员工都有一个 Id，并且还有一列是经理的 Id
Id		Name	department		managerId
101		john	A				NULL
102		dan		A				101
103		james	A				101
104		amy		A				101
105		Anne	A				101
请编写一个SQL查询来查找至少有4名直接下属的经理
select * from emp where id  in (
   select managerId from emp group by managerId having count(1) >= 4
);

其他写法：
select jingli from
(select b1.name jingli ,collect_list(b2.name) yuangong from emp b1,emp b2
      where b1.id = b2.managerid group by b1.id,b1.name)t1
              where size(yuangong) >= 4;
五、编写sql实现每个用户截止到每月为止的最大单月访问次数和累计到该月的总访问次数
统计的结果：每个用户截止到每月为止的最大单月访问次数和累计到该月的总访问次数，结果数据格式如下:

userid,month,visits
A,2015-01,5
A,2015-01,15
B,2015-01,5
A,2015-01,8
B,2015-01,25
A,2015-01,5
A,2015-02,4
A,2015-02,6
B,2015-02,10
B,2015-02,5
A,2015-03,16
A,2015-03,22
B,2015-03,23
B,2015-03,10
B,2015-03,1

新建表和加载数据：
创建表：
create table user_visit(
userid string,
month string,
visits int
)
row format delimited
fields terminated by ',';
加载数据：
load data local inpath '/home/hivedata/1.txt' into table user_visit;

第一步分组：
select userid,month,sum(visits) from user_visit group by userid,month;

userid  month   _c2
A       2015-01 33
A       2015-02 10
A       2015-03 38
B       2015-01 30
B       2015-02 15
B       2015-03 34

第二步：统计累计的访问次数和到目前为止最大访问次数

select 
*,
sum(dycs) over(partition by userid order by month) `目前为止的累计访问次数`,
max(dycs) over(partition by userid order by month) `目前为止的最大访问次数`
from 
(

    select userid,month,sum(visits) dycs from user_visit group by userid,month
) a ;

a.userid        a.month a.dycs  目前为止的累计访问次数  目前为止的最大访问次数
A       2015-01 33      33      33
A       2015-02 10      43      33
A       2015-03 38      81      38
B       2015-01 30      30      30
B       2015-02 15      45      30
B       2015-03 34      79      34
六、求出每个栏目的被观看次数及累计观看时长
数据: vedio表
uid channl min
1 1 23
2 1 12
3 1 12
4 1 32
5 1 342
6 2 13
7 2 34
8 2 13
9 2 134
创建表，导入数据：
CREATE TABLE if not exists vedio
(
uid int,
channl int,
min int
)row format delimited fields terminated by ' ';

load data local inpath '/home/hivedata/2.txt' into table vedio;

select channl,sum(min),count(1) from vedio group by channl;

channl  _c1     _c2
1       421     5
2       194     4
七、编写连续7天登录的总人数
uid dt login_status(1登录成功,0异常)
1 2019-07-11 1
1 2019-07-12 1
1 2019-07-13 1
1 2019-07-14 1
1 2019-07-15 1
1 2019-07-16 1
1 2019-07-17 1
1 2019-07-18 1
2 2019-07-11 1
2 2019-07-12 1
2 2019-07-13 0
2 2019-07-14 1
2 2019-07-15 1
2 2019-07-16 0
2 2019-07-17 1
2 2019-07-18 0
3 2019-07-11 1
3 2019-07-12 1
3 2019-07-13 1
3 2019-07-14 1
3 2019-07-15 1
3 2019-07-16 1
3 2019-07-17 1
3 2019-07-18 1
create table if not exists t1
(
uid int,
dt string,
login_status int
)row format delimited fields terminated by ' ';

load data local inpath '/home/hivedata/3.txt' into table t1;

第一步：先按照uid和dt进行分组和排序，编号
select * ,dense_rank() over(partition by uid order by dt) from t1;

t1.uid  t1.dt   t1.login_status dense_rank_window_0
1       2019-07-11      1       1
1       2019-07-12      1       2
1       2019-07-13      1       3
1       2019-07-14      1       4
1       2019-07-15      1       5
1       2019-07-16      1       6
1       2019-07-17      1       7
1       2019-07-18      1       8
2       2019-07-11      1       1
2       2019-07-12      1       2
2       2019-07-13      0       3
2       2019-07-14      1       4
2       2019-07-15      1       5
2       2019-07-16      0       6
2       2019-07-17      1       7
2       2019-07-18      0       8
3       2019-07-11      1       1
3       2019-07-12      1       2
3       2019-07-13      1       3
3       2019-07-14      1       4
3       2019-07-15      1       5
3       2019-07-16      1       6
3       2019-07-17      1       7
3       2019-07-18      1       8

第二步 让dt 和 序号，相减  date_sub()

select *,date_sub(dt,xh) rq  from (
select * ,dense_rank() over(partition by uid order by dt) xh from t1 ) a;

a.uid   a.dt    a.login_status  a.xh    rq
1       2019-07-11      1       1       2019-07-10
1       2019-07-12      1       2       2019-07-10
1       2019-07-13      1       3       2019-07-10
1       2019-07-14      1       4       2019-07-10
1       2019-07-15      1       5       2019-07-10
1       2019-07-16      1       6       2019-07-10
1       2019-07-17      1       7       2019-07-10
1       2019-07-18      1       8       2019-07-10
2       2019-07-11      1       1       2019-07-10
2       2019-07-12      1       2       2019-07-10
2       2019-07-13      0       3       2019-07-10
2       2019-07-14      1       4       2019-07-10
2       2019-07-15      1       5       2019-07-10
2       2019-07-16      0       6       2019-07-10
2       2019-07-17      1       7       2019-07-10
2       2019-07-18      0       8       2019-07-10
3       2019-07-11      1       1       2019-07-10
3       2019-07-12      1       2       2019-07-10
3       2019-07-13      1       3       2019-07-10
3       2019-07-14      1       4       2019-07-10
3       2019-07-15      1       5       2019-07-10
3       2019-07-16      1       6       2019-07-10
3       2019-07-17      1       7       2019-07-10
3       2019-07-18      1       8       2019-07-10

第三步：根据uid 和 rq记性分组，然后count, group by 后面可以直接使用别名

select uid,rq,count(1) ljdl from (
select *,date_sub(dt,xh) rq  from (
select * ,dense_rank() over(partition by uid order by dt) xh from t1 
    where login_status = 1
) 
    a ) 
b group by uid,rq;

uid     rq      ljdl
1       2019-07-10      8
2       2019-07-10      2
2       2019-07-11      2
2       2019-07-12      1
3       2019-07-10      8

第四步： 过滤出来累计登录7次以上的用户
select count(*) from (
select uid,rq,count(1) ljdl from (
select *,date_sub(dt,xh) rq  from (
select * ,dense_rank() over(partition by uid order by dt) xh from t1 
    where login_status = 1
) 
    a ) 
b group by uid,rq having ljdl >=7 ) c ;

八、使用hive编写sql

1 time1 read
3 time2 comment
1 time3 share
2 time4 like
1 time5 write
2 time6 like
3 time7 write
2 time8 read

建表，导入数据

create table if not exists user_action_log
(
u_id int,
`time` string,
action string
)row format delimited fields terminated by ' ';
load data local inpath '/home/hivedata/user_action_log.txt' into table user_action_log;

第一步：
 select *,row_number() over(partition by u_id order by `time`) bianhao from user_action_log ;

第二步：求编号为1的数据
select * from (
 select *,row_number() over(partition by u_id order by `time`) bianhao from user_action_log ) a where bianhao=1 ;

a.u_id  a.time  a.action        a.bianhao
1       time1   read    1
2       time4   like    1
3       time2   comment 1

另外一种写法：使用first_value
select distinct u_id ,first_value(action) over(partition by u_id order by `time`) from user_action_log;

u_id    _c1
1       read
2       like
3       comment
九、每个店铺的当月销售额和累计到当月的总销售额
店铺,月份,金额
a,01,150
a,01,200
b,01,1000
b,01,800
c,01,250
c,01,220
b,01,6000
a,02,2000
a,02,3000
b,02,1000
b,02,1500
c,02,350
c,02,280
a,03,350
a,03,250

建表，导入数据：
create table if not exists stores
(
store string,
month string,
money int
)row format delimited fields terminated by ',';

load data local inpath '/home/hivedata/5.txt' into table stores;

select * ,sum(dyxl) over(partition by store order by month)
select store,month,sum(money) dyxl from stores group by store,month;
