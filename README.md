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
      

        


        
