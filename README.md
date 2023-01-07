# SPCN-NMS
# Cacti
**ขั้นตอนการติดตั้งและ Configure Cacti**
 1. คำสั่งอัพเดท package ในระบบ

        apt-get update -y
        apt-get install snmp php-snmp rrdtool librrds-perl unzip curl git gnupg2 -y

        
 2. คำสั่งติดตั้ง LAMP Server(Apache web server, MariaDB, PHP, MySQL)

        apt-get install apache2 mariadb-server php php-mysql libapache2-mod-php php-xml php-ldap php-mbstring php-gd php-gmp -y


**Install LAMP Server**
***

 3. เมื่อติดตั้ง package เสร็จแล้วให้เข้าไปแก้ไข timezone และ date ตามคำสั่งต่อไปนี้
  
        nano /etc/php/7.4/apache2/php.ini

    แก้ไขวันที่และเวลาให้เป็น ดังนี้

        memory_limit = 512M
        max_execution_time = 60
        date.timezone = Asia/Bangkok
        
    เมื่อแก้ไขเรียบร้อยแล้วให้ save และไปแก้ไข timezone และ date ของไฟล์ php.ini อื่นๆโดยใช้คำสั่งดังนี้
        
        nano /etc/php/7.4/cli/php.ini
        
    แก้ไขวันที่และเวลาให้เป็น ดังนี้

        memory_limit = 512M
        max_execution_time = 60
        date.timezone = Asia/Kolkata

        
 4. เมื่อแก้ไขเรียบร้อยแล้วให้ restart Apache โดยใช้คำสั่ง ดังนี้
  
        systemctl restart apache2
     
**Configure MariaDB Server**
***
**Cacti ใช้ MariaDB เป็น database Backend** 

 5. เข้าไปแก้ไขค่าใน MariaDB โดยใช้คำสั่งดังนี้ 

        nano /etc/mysql/mariadb.conf.d/50-server.cnf
        
    เมื่อเข้ามาแล้วให้แก้ไขค่าให้เป็นไปตามต่อไปนี้

        collation-server = utf8mb4_unicode_ci
        max_heap_table_size = 512M
        tmp_table_size = 512M
        join_buffer_size = 1024M
        innodb_file_format = Barracuda
        innodb_large_prefix = 1
        innodb_buffer_pool_size = 16384M
        innodb_flush_log_at_timeout = 3
        innodb_read_io_threads = 32
        innodb_write_io_threads = 32
        innodb_io_capacity = 5000
        innodb_io_capacity_max = 10000
        innodb_buffer_pool_instances = 50
        innodb_doublewrite = OFF
        #collation-server = utf8mb4_general_ci
        
    บันทึกและ restart MariaDB ใหม่ โดยใช้คำสั่งต่อไปนี้
 
        systemctl restart mariadb
        
 6. Login เข้า MariDB shell โดยใช้คำสั่งต่อไปนี้
        
        mysql -u root -p
        
    เมื่อ Login เข้ามาแล้วให้สร้าง database และ user สำหรับ cacti โดยใช้คำสั่งต่อไปนี้

        create database cacti;
        GRANT ALL ON cacti.* TO cactiuser@localhost IDENTIFIED BY 'cactiuser';
        flush privileges;
        exit;
        
        และใช้คำสั่งต่อไปนี้เพื่อนำเข้าเวลาของเครื่องเราไปยัง MySQL
        mysql mysql < /usr/share/mysql/mysql_test_data_timezone.sql
        
 7. Login เข้าไปใน MariaDB shell เพื่อให้สิทธิ์ timezone ใน MySQL โดยใช้คำสั่งต่อไปนี้

        mysql -u root -p
        GRANT SELECT ON mysql.time_zone_name TO cactiuser@localhost;
        flush privileges;
        exit;
        
**Install and Configure Cacti**
***
 8. Dowload Cacti เวอร์ชั่นล่าสุดโดยใช้คำสั่งต่อไปนี้

        wget https://www.cacti.net/downloads/cacti-latest.tar.gz
        
    ใช้คำสั่งต่อไปนี้เพื่อแตกไฟล์ที่ Dowload มา
   
        tar -zxvf cacti-latest.tar.gz

    ใช้คำสั่งต่อไปนี้เพื่อใช้ย้าย Directory ที่แตกไฟล์ออกมาไปยัง Directory ของ Apache
   
        mv cacti-1* /var/www/html/cacti

    ใช้คำสั่งต่อไปนี้นำเข้า database ไปที่ cactiDB
   
        mysql cacti < /var/www/html/cacti/cacti.sql

    เข้าไปที่ไฟล์ config.php ของ Cacti แล้วกำหนดค่าดังนี้
   
        nano /var/www/html/cacti/include/config.php     //คำสั่งเข้าไปในไฟล์ config.php
        
    ตรวจสอบชื่อ, username, password ของ database โดยมีค่า Default ดังต่อไปนี้
   
        $database_type 	= 'mysql';
        $database_default  = 'cacti';
        $database_hostname = 'localhost';
        $database_username = 'cactiuser';
        $database_password = 'cactiuser';
        $database_port 	= '3306';
        
9. สร้าง log file สำหรับ Cacti โดยใช้คำสั่งต่อไปนี้

        touch /var/www/html/cacti/log/cacti.log
        
    กำหนด permission โดยใช้คำสั่งต่อไปนี้ 
   
        chown -R www-data:www-data /var/www/html/cacti/
        chmod -R 775 /var/www/html/cacti/
        
    เข้าไปที่ไฟล์ cron.d โดยใช้คำสั่งต่อไปนี้
   
        nano /etc/cron.d/cacti

    เพิ่มบรรทัดด้านล่างนี้ไปยังไฟล์ /etc/cron.d/cacti
   
        */5 * * * * www-data php /var/www/html/cacti/poller.php > /dev/null 2>&1

**Configure Apache for Cacti**
***

 10. ใช้คำสั่งต่อไปนี้ 
        
        nano /etc/apache2/sites-available/cacti.conf
        
    นำคำสั่งด้านล่างนี้ไปวางในไฟล์ /etc/apache2/sites-available/cacti.conf
    
        Alias /cacti /var/www/html/cacti
 
        <Directory /var/www/html/cacti>
  	        Options +FollowSymLinks
  	        AllowOverride None
  	        <IfVersion >= 2.3>
  	        Require all granted
  	        </IfVersion>
  	        <IfVersion < 2.3>
  	        Order Allow,Deny
  	        Allow from all
  	        </IfVersion>
 
        AddType application/x-httpd-php .php
 
        <IfModule mod_php.c>
  	        php_flag magic_quotes_gpc Off
  	        php_flag short_open_tag On
  	        php_flag register_globals Off
  	        php_flag register_argc_argv On
  	        php_flag track_vars On
  	        # this setting is necessary for some locales
  	        php_value mbstring.func_overload 0
  	        php_value include_path .
        </IfModule>
 
         DirectoryIndex index.php
        </Directory>

    บันทึกและใช้คำสั่งต่อไปนี้เพื่อเข้าไปที่ host file
        
        a2ensite cacti
        
    ใช้คำสั่งต่อไปนี้เพื่อ restart apache
    
        systemctl restart apache2



                

      

        


        
