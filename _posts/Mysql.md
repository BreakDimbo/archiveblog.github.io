 MySql 密码： u%g&1f:E_pu.
 
 
 日期：  
 now（）  
 将日期转换为字符串：  
 date_format(now(), '%Y-%m-%d %H:%i:%s');  
 将字符串转换成日期
 
 
 自增序列：auto_increment
 
 ~~~sql
 create table artical 
(
id int primary key auto_increment,
title varchar(255)
);
 ~~~
 
 
 使用jdbc连接mysql