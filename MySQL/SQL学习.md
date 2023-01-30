
## 常用SQL
### 查询某年数据
```
select count(device_log.id) from device_log WHERE date_format(create_time,'%Y')='2020'
```

### binlog 文件清理
```
# 显示使用中的bin_log
show master status;
# 清除数据
PURGE MASTER LOGS BEFORE DATE_SUB(CURRENT_DATE, INTERVAL 1 DAY);
```

### Mysql修改binlog日志过期时间
```
1.临时生效

# 查看默认设置的过期时间
show variables like "%expire_logs%";
# 设置保留15天
set global expire_logs_days=15

# 刷新日志
flush logs；

# 查看新生成的binlog日志
show master status\G:
# 注意：以上命令在数据库执行会立即生效，请确定设置数据的保留日期，以免误删

2.永久生效　　

# 修改配置文件
vim /etc/my.cnf
[mysqld]模块
expire_logs_days=15

# 注意：在配置文件修改后，需要重启才能永久生效。另，0表示永不过期，单位是天
```

### mysql查看当前实时连接数
- 静态查看
```
SHOW PROCESSLIST;

SHOW FULL PROCESSLIST;

SHOW VARIABLES LIKE '%max_connections%';

SHOW STATUS LIKE '%Connection%';
```

- 实时查看
```
show status like 'Threads%';
```

Threads_connected 跟show processlist结果相同，表示当前连接数。准确的来说，Threads_running是代表当前并发数
max_connections 这是是查询数据库当前设置的最大连接数

### 事务锁表处理

```
SELECT concat('KILL ',id,';') FROM information_schema.processlist p INNER JOIN  information_schema.INNODB_TRX x ON p.id=x.trx_mysql_thread_id WHERE db='db_share_backstage'
```

### 方法函数
```
set global log_bin_trust_function_creators=TRUE
```

## 学生选课表SQL
```
一，建表

/*

Navicat MySQL Data Transfer
Source Server         : mysql
Source Server Version : 50626
Source Host           : localhost:3306
Source Database       : school
Target Server Type    : MYSQL
Target Server Version : 50626
File Encoding         : 65001
Date: 2020-04-24 20:26:21
*/
SET FOREIGN_KEY_CHECKS=0;

-- ----------------------------
-- Table structure for course
-- ----------------------------
DROP TABLE IF EXISTS `course`;
CREATE TABLE `course` (
  `Cid` varchar(11) NOT NULL,
  `Cname` varchar(255) DEFAULT NULL,
  `Tid` varchar(11) DEFAULT NULL,
  PRIMARY KEY (`Cid`)

) ENGINE=InnoDB DEFAULT CHARSET=utf8;
-- ----------------------------
-- Records of course
-- ----------------------------

INSERT INTO `course` VALUES ('001', 'CN001', 'T001');
INSERT INTO `course` VALUES ('002', 'CN002', 'T001');
INSERT INTO `course` VALUES ('003', 'CN003', 'T002');
INSERT INTO `course` VALUES ('004', 'CN004', 'T003');
INSERT INTO `course` VALUES ('005', 'CN005', 'T003');

-- ----------------------------
-- Table structure for sc
-- ----------------------------
DROP TABLE IF EXISTS `sc`;
CREATE TABLE `sc` (
  `Sid` varchar(11) NOT NULL,
  `Cid` varchar(11) NOT NULL,
  `score` float DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of sc
-- ----------------------------

INSERT INTO `sc` VALUES ('1001', '001', '60');
INSERT INTO `sc` VALUES ('1001', '002', '61');
INSERT INTO `sc` VALUES ('1002', '001', '80');
INSERT INTO `sc` VALUES ('1002', '002', '61');
INSERT INTO `sc` VALUES ('1003', '002', '65');
INSERT INTO `sc` VALUES ('1004', '003', '66');
INSERT INTO `sc` VALUES ('1002', '003', '100');
INSERT INTO `sc` VALUES ('1003', '004', '96');
INSERT INTO `sc` VALUES ('1003', '005', '88');
INSERT INTO `sc` VALUES ('1004', '004', '52');
INSERT INTO `sc` VALUES ('1005', '001', '59');
INSERT INTO `sc` VALUES ('1005', '002', '58');
INSERT INTO `sc` VALUES ('1005', '003', '57');
-- ----------------------------
-- Table structure for student
-- ----------------------------
DROP TABLE IF EXISTS `student`;
CREATE TABLE `student` (
  `Sid` int(11) NOT NULL,
  `Sname` varchar(255) DEFAULT NULL,
  `Sage` varchar(255) DEFAULT NULL,
  `Ssex` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`Sid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

  

-- ----------------------------
-- Records of student
-- ----------------------------
INSERT INTO `student` VALUES ('1001', 'SN1001', '11', '1');
INSERT INTO `student` VALUES ('1002', 'SN1002', '12', '0');
INSERT INTO `student` VALUES ('1003', 'SN1003', '13', '1');
INSERT INTO `student` VALUES ('1004', 'SN1004', '14', '1');
INSERT INTO `student` VALUES ('1005', 'SN1005', '15', '1');

-- ----------------------------

-- Table structure for teacher
-- ----------------------------
DROP TABLE IF EXISTS `teacher`;
CREATE TABLE `teacher` (
  `Tid` varchar(11) NOT NULL,
  `Tname` varchar(255) NOT NULL,
  PRIMARY KEY (`Tid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of teacher
-- ----------------------------

INSERT INTO `teacher` VALUES ('T001', '李老师');
INSERT INTO `teacher` VALUES ('T002', '李老师');
INSERT INTO `teacher` VALUES ('T003', '叶平');

二 查询

# 1 查询“001”课程比“002”课程成绩高的所有学生的学号;
SELECT a.Sid
FROM (SELECT Sid,score FROM sc WHERE sc.Cid='001') AS a,
(SELECT Sid,score FROM sc WHERE sc.Cid='002') AS b
WHERE a.Sid = b.Sid and a.score>b.score;

# 2、查询平均成绩大于65分的同学的学号和平均成绩
SELECT sc.Sid,AVG(sc.score)
FROM sc
GROUP BY sc.Sid
HAVING AVG(sc.score)>65

# 3、查询所有同学的学号、姓名、选课数、总成绩;

SELECT sc.Sid,student.Sname,COUNT(sc.Cid),SUM(sc.score)
FROM sc,student
where sc.Sid = student.Sid
GROUP BY sc.Sid

select Student.Sid,Student.Sname,count(SC.Cid),sum(score)
from Student left join SC on Student.Sid=SC.Sid
group by Student.Sid,Sname


# 4、查询姓“李”的老师的个数;

select count(Tname)
from Teacher
where Tname like '李%';

# 5、查询没学过“叶平”老师课的同学的学号、姓名;

SELECT student.Sid ,student.Sname from student
where Sid not in (
SELECT DISTINCT(sc.Sid) FROM sc,course,teacher
where sc.Cid=course.Cid and teacher.Tid=course.Tid and teacher.Tname='叶平' )

# 6、查询学过“001”并且也学过编号“002”课程的同学的学号、姓名;

SELECT DISTINCT(student.Sid),student.Sname From student,
(SELECT sc.Sid FROM sc where sc.Cid='001') as a,
(SELECT sc.Sid FROM sc where sc.Cid='002') as b
where a.Sid = b.Sid and student.Sid = a.Sid;

# 7、查询学过“叶平”老师所教的所有课的同学的学号、姓名;

SELECT student.Sid,student.Sname FROM teacher,sc,course,student
WHERE student.Sid=sc.Sid AND sc.Cid=course.Cid AND course.Tid = teacher.Tid AND teacher.Tname='叶平'
GROUP BY student.Sid
HAVING COUNT(sc.Sid) =
(SELECT COUNT(course.Cid) FROM teacher,course
where course.Tid = teacher.Tid AND teacher.Tname='叶平')

#8、查询课程编号“002”的成绩比课程编号“001”课程低的所有同学的学号、姓名;

SELECT student.Sid ,student.Sname from student,
(SELECT Sid,score FROM sc WHERE sc.Cid='001') AS a,
(SELECT Sid,score FROM sc WHERE sc.Cid='002') AS b
where a.score > b.score and a.Sid=b.Sid and student.Sid= a.Sid;

# 9、查询所有课程成绩小于60分的同学的学号、姓名;

SELECT student.Sid,student.Sname FROM student,sc
where student.Sid=sc.Sid AND sc.score < 60
GROUP BY student.Sid

HAVING COUNT(student.Sid) =
(SELECT COUNT(sc.Sid) FROM sc
student.Sid=sc.Sid
GROUP BY sc.Sid)
  
# 10 查询没有学全所有课的同学的学号、姓名

SELECT student.Sid,student.Sname FROM student,sc
where student.Sid=sc.Sid
GROUP BY student.Sid
HAVING COUNT(student.Sid) <
(SELECT COUNT(course.Cid) FROM course)


# 11、查询至少有一门课与学号为“1001”的同学所学相同的同学的学号和姓名;

SELECT DISTINCT(student.Sid) , student.Sname FROM student ,sc
WHERE sc.Sid = student.Sid AND sc.Cid IN(
SELECT sc.Cid FROM sc
WHERE sc.Sid = '1001')

# 12、查询至少学过学号为“001”同学所学一门课的其他同学学号和姓名;

SELECT DISTINCT(student.Sid) , student.Sname FROM student ,sc
WHERE sc.Sid = student.Sid AND sc.Cid IN(
SELECT sc.Cid FROM sc
WHERE sc.Sid = '1001')

AND sc.Sid != '1001'
```