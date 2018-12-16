title: 前端技能之 PHP 学习笔记
date: 2016-04-20 11:20:24
categories: PHP
---

##开发准备
我的开发环境是 ubuntu 14.10，选择了比较简单的 *xampp* 这个比较简单易上手的集成环境。
安装流程参考了百度的一篇博文: [ubuntu 下安装 xampp](http://jingyan.baidu.com/article/066074d66e1141c3c21cb0ce.html)。
<!--more-->
###关于 MySQL 连接问题
MySQL 在 xampp 启动后，如果直接连接会出现
```
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (2)
```
这个问题我参考了众多博文后，找到了解决方法：
```
sudo mkdir /var/run/mysqld
sudo ln -s /opt/lampp/var/mysql/mysql.sock /var/run/mysqld/mysqld.sock
mysql -u root -p
```
这样就可以解决问题了。
##MySQL
MySQL 是常与 php 配合使用的关系型数据库，下面给出常用的 SQL 语句：
```
建立数据库：CREATE DATABASE database_name
选择数据库：USE database_name 
建立表：CREATE TABLE table_name (
            first_name VARCHAR(20),
            last_name VARCHAR(30),
            email VARCHAR(60)
)
输入数据：INSERT INTO table_name (column1, column2, ...) VALUES ('value1', 'value2', ...)
选择列：SELECT * FROM list_name where xx='yy' AND zz='xx';
删除列：DELETE FROM table_name WHERE xxxx='yyy' AND zzz="111"
插入列：ALTER TABLE table_name ADD column_name column_type(AFTER xxx)
排序语句：SELECT * FROM guitarwars ORDER BY xxx ASC/DESC  (ASC 表示升序 DESC　表示降序)
分页查询语句：SELECT * FROM table LIMIT [offset,] rows | rows OFFSET offset

```
##几种常见的 Web 组件的 PHP 开发
这里，我并不打算详细地从头介绍 php 的语法，我相信网上的很多资料已经讲的非常清楚，这里希望通过几个例子，
让其他人知道一些常见组件的做法以及了解 php 的使用。
###分页技术
分页技术的思想在于使用 GET 请求，但是通过变量来限制请求的数据的数量。比如说第一页显示 4 条数据，那么从
数据库中选出 id 为 0 - 3 的数据，而第二页就是从 4 - 7 的数据，限制每次查询都只是取 4 条数据，而从哪里
开始取数据，则是依靠偏移量这个变量（从 0 ，4, 8 ...开始取数据）。
```
<?php
    $dbc = mysqli_connect('localhost', 'root', '', 'wheel') or die("数据库连接失败");
    //每页显示4 条数据
    $pagesize = 4;
    
    //确定页数 p 参数
    $p = isset($_GET['p']) ? $_GET['p'] : 1;
    
    //数据指针
    $offset = ($p - 1) * $pagesize;
    
    //查询本页显示数据
    $query_sql = "SELECT * FROM guestbook ORDER BY id DESC LIMIT  $offset , $pagesize"; //$offset 是偏移量
    $result = mysqli_query($dbc, $query_sql);  //这里只是存放了一个查询 id ，真正的数据需要使用 mysqli_fetch_query 来调取，这是一个数组，其中 DESC 的意思是降序
    while($list = mysqli_fetch_array($result) ) {
        echo '<a href="',$list['nickname'],'">',$list['nickname'],'</a>';
        echo '发表于：',date("Y-m-d H:i", $list['createtime']),'<br />';
        echo '内容：',$list['content'],'<br /><hr />';
    }
    
    //分页代码
    //计算留言总数
    $count_result = mysql_query("SELECT count(*) as count FROM guestbook");
    $count_array = mysql_fetch_array($count_result);

    //计算总的页数
    $pagenum = ceil($count_array['count'] / $pagesize); // ceil 取整
    echo '共 ',$count_array['count'],' 条留言';

    //循环输出各页数目及连接
    if ($pagenum > 1) {
        for($i = 1;$i <= $pagenum;$i ++) {
            if($i == $p) {
                echo ' [',$i,']';
            } else {
                echo ' <a href="page.php?p=',$i,'">',$i,'</a>';
            }
        }
    }
    mysqli_close($dbc);  //关闭数据库连接
?>
```
###用户注册与登陆
这里博文没有涉及添加前端的表单验证与 HTML 代码的编写，只是为了能够更好地体验 PHP 服务端开发。
注册模块：
```
<?php
//包含数据库连接文件
include('conn.php');
//检测用户名是否已经存在
$check_query = mysql_query("SELECT uid FROM user WHERE username='$username' LIMIT 1");
if(mysql_fetch_array($check_query)){
    echo '错误：用户名 ',$username,' 已存在。<a href="javascript:history.back(-1);">返回</a>';
    exit;
}
//写入数据, 使用 MD5 函数进行密码加密
$password = MD5($password);
$regdate = time();
$sql = "INSERT INTO user(username,password,email,regdate) VALUES ('$username','$password','$email',
$regdate)";
if(mysql_query($sql,$conn)){
    exit('用户注册成功！点击此处 <a href="login.html">登录</a>');
} else {
    echo '抱歉！添加数据失败：',mysql_error(),'<br />';
    echo '点击此处 <a href="javascript:history.back(-1);">返回</a> 重试';
}
?>
```
登陆模块：
```
<?php
   if(！isset($_POST['submit'])) {
        exit("非法访问");
   }
   $userName = htmlspecialchar($_POST['username']);  //过滤非法字符
   $password = MD5($_POST['password']);
   
   //数据库连接文件
    include(connect.php);
    $check_query = mysql_query("select uid from user where username='$username' and password='$password'     limit 1");
    if($result = mysql_fetch_array($check_query)){
    //登录成功
        $_SESSION['username'] = $username;
        $_SESSION['userid'] = $result['uid'];
        echo $username,' 欢迎你！进入 <a href = "my.php">用户中心</a><br />';
        echo '点击此处 <a href = "login.php ? action = logout">注销</a> 登录！<br />';
        exit;
    } else {
        exit('登录失败！点击此处 <a href="javascript:history.back(-1);">返回</a> 重试');
    }
?>
```
退出模块：
```
    session_start();

    //注销登录
    if($_GET['action'] == "logout"){
        unset($_SESSION['userid']);
        unset($_SESSION['username']);
        echo '注销登录成功！点击此处 <a href="login.html">登录</a>';
        exit;
    }
```

参考文献：
*《Head First PHP and MySQL》*




