# SPCN-NMS
# Cacti
![5142645](https://user-images.githubusercontent.com/115150753/211187480-30a5fb2c-9f23-4f67-b472-c9cab383c5c9.png)

**ขั้นตอนการติดตั้งและ Configure Cacti**
 1. คำสั่งอัพเดท package ในระบบ และ set timezone ของเครื่องเป็นเขตเวลาของตัวเอง

        apt-get update -y
        
        apt-get install snmp php-snmp rrdtool librrds-perl unzip curl git gnupg2 -y
        
        timedatectl set-timezone Asia/Bangkok
        

**Install LAMP Server**
***
 2. คำสั่งติดตั้ง LAMP Server(Apache web server, MariaDB, PHP, MySQL)

        apt-get install apache2 mariadb-server php php-mysql libapache2-mod-php php-xml php-ldap php-mbstring php-gd php-gmp -y

 3. เมื่อติดตั้ง package เสร็จแล้วให้เข้าไปแก้ไข timezone และ date ตามคำสั่งต่อไปนี้
  
        nano /etc/php/*/apache2/php.ini
        
        nano /etc/php/*/cli/php.ini
        
        ( * คือ version ของ php เปลี่ยนให้ตรงกับ version ที่ตัวเองใช้อยู่ )
        

    แก้ไขวันที่และเวลาให้เป็นและค่า configuration ของทั้ง 2 ไฟล์ให้เป็นดังนี้

        memory_limit = 512M
        max_execution_time = 60
        date.timezone = Asia/Bangkok
        
        
ตัวอย่าง


![3](https://user-images.githubusercontent.com/98762543/211158493-dae18e14-e93d-4006-a737-f921a3425d6c.PNG)

![5](https://user-images.githubusercontent.com/98762543/211158495-8b23496c-1aaa-4ef8-8db3-6e021d9078e5.PNG)

![4](https://user-images.githubusercontent.com/98762543/211158494-1acba1b5-98fe-4303-b872-14c475e98f29.PNG)
        

        
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
 
 
รูปตัวอย่าง

![6](https://user-images.githubusercontent.com/98762543/211158496-23b9033b-ff5b-488c-9dd5-5bd3100739d1.PNG)
        
        
   บันทึกและ restart MariaDB ใหม่ โดยใช้คำสั่งต่อไปนี้
 
        systemctl restart mariadb
        
 6. Login เข้า MariaDB shell โดยใช้คำสั่งต่อไปนี้
        
        mysql -u root -p
        
    เมื่อ Login เข้ามาแล้วให้สร้าง database และ user สำหรับ cacti โดยใช้คำสั่งต่อไปนี้

        create database cacti;
        GRANT ALL ON cacti.* TO cactiuser@localhost IDENTIFIED BY 'cactiuser';
        flush privileges;
        exit;
   
   
รูปตัวอย่าง

![7](https://user-images.githubusercontent.com/98762543/211158498-c1595016-fc17-4067-ae84-ddcd611a192f.PNG)
        

   และใช้คำสั่งต่อไปนี้เพื่อนำเข้าเวลาของเครื่องเราไปยัง MySQL   

        mysql mysql < /usr/share/mysql/mysql_test_data_timezone.sql
        
        
        
 7. Login เข้าไปใน MariaDB shell เพื่อให้สิทธิ์ timezone ใน MySQL โดยใช้คำสั่งต่อไปนี้

        mysql -u root -p
        GRANT SELECT ON mysql.time_zone_name TO cactiuser@localhost;
        flush privileges;
        exit;
  
  
รูปตัวอย่าง  

![8](https://user-images.githubusercontent.com/98762543/211158499-ce13d647-32da-455e-b403-b24ea4ca0c68.PNG)

        
**Install and Configure Cacti**
***


 8. Dowload Cacti เวอร์ชั่นล่าสุดโดยใช้คำสั่งต่อไปนี้

        wget https://www.cacti.net/downloads/cacti-latest.tar.gz
        


รูปตัวอย่าง

![9](https://user-images.githubusercontent.com/98762543/211158501-ea8f2567-06e3-477c-bb5d-f44ceaa0832d.PNG)
   
   ใช้คำสั่งต่อไปนี้เพื่อแตกไฟล์ที่ Dowload มา
   
        tar -zxvf cacti-latest.tar.gz
        
รูปตัวอย่าง

![10](https://user-images.githubusercontent.com/98762543/211158503-7b3711af-5e8c-4545-b437-1ed423e75309.PNG)

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
        
  รูปตัวอย่าง
  
  ![11](https://user-images.githubusercontent.com/98762543/211158506-82f9290e-5e1c-43be-b5ed-9b68e026d943.PNG)
  
        
9. สร้าง log file สำหรับ Cacti โดยใช้คำสั่งต่อไปนี้

        touch /var/www/html/cacti/log/cacti.log
        
    กำหนด permission โดยใช้คำสั่งต่อไปนี้ 
   
        chown -R www-data:www-data /var/www/html/cacti/
        chmod -R 775 /var/www/html/cacti/
        
    เข้าไปที่ไฟล์ cron.d โดยใช้คำสั่งต่อไปนี้
   
        nano /etc/cron.d/cacti

    เพิ่มบรรทัดด้านล่างนี้ไปยังไฟล์ /etc/cron.d/cacti
   
        */5 * * * * www-data php /var/www/html/cacti/poller.php > /dev/null 2>&1
        
        
    รูปตัวอย่าง
    
    ![12](https://user-images.githubusercontent.com/98762543/211158507-fefff877-912c-44d1-a86f-2d19da58ff22.PNG)
    

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
        
        
   รูปตัวอย่าง
   
   ![13](https://user-images.githubusercontent.com/98762543/211158508-0cf01513-19f8-4bac-97f4-e5ba2da72d78.PNG)


   บันทึกและใช้คำสั่งต่อไปนี้เพื่อเข้าไปที่ host file
        
        a2ensite cacti
        
   ใช้คำสั่งต่อไปนี้เพื่อ restart apache
    
        systemctl restart apache2


ขั้นตอน setup Cacti บน webbrowser

  เปิด browser และไปยัง ip ของเครื่อง host เช่น 
  
           http://localhost/cacti
           
           หรือ
           
           172.31.*.**/cacti
           
  จะเจอกับหน้า login ครั้งแรก 
  user คือ admin
  password คือ admin
  
  รูปตัวอย่าง
  
  ![15](https://user-images.githubusercontent.com/98762543/211158510-a54f2f06-2f72-471f-833b-3beb5bf3bbf2.PNG)
  
  เมื่อ login สำเร็จ ระบบจะให้เปลี่ยน password
  
  รูปตัวอย่าง
  
  ![16](https://user-images.githubusercontent.com/98762543/211158511-01bb6bc7-d93b-4363-b405-671f76d1e3d9.PNG)
  
  ติ๊กถูกที่ Accept GPL แล้วกด Begin
  
  รูปตัวอย่าง
  
  ![17](https://user-images.githubusercontent.com/98762543/211158512-c7c3995d-7244-43a7-a0fd-9283792aa54e.PNG)
  
  ต่อไปจะเป็นหน้าที่แสดงค่า Configure ต่าง ๆ ระบบจะบอกว่าต้องการอะไรเพิ่มหรือการตั้งค่าอะไรที่ผิดพลาด อย่างในตัวอย่างนี้ ระบบต้องการ Modules php intl 
  ก็ให้ติดตั้งเพิ่มโดยใช้คำสั่ง
        
        apt-get install -y php-intl
  
  รูปตัวอย่าง
  
  ![18](https://user-images.githubusercontent.com/98762543/211158513-5e8d77df-9d03-4836-8e69-e492d3a12e20.PNG)
  
  ![19](https://user-images.githubusercontent.com/98762543/211158514-9f27f9b7-1844-467f-9f64-c68c1828e4ed.PNG)
  
  กด next ไป Step ต่อไปเรื่อยๆ 
  
  รูปตัวอย่าง
  - เมื่อ Pre-Installation Checks ผ่าน ให้ทำการกด Next
  ![20](https://user-images.githubusercontent.com/98762543/211158515-bd029abe-3441-442c-a7f0-0846b660a370.PNG)
  
  - เลือก New Primary Server และตรวจสอบ Local Database Connection Information ให้เรียบร้อย
  ![21](https://user-images.githubusercontent.com/98762543/211158516-2e677f25-71b2-400a-be45-5b85f87ddc4b.PNG)

  - เมื่อ Directory Permission Checks ผ่าน ให้ทำการกด Next
  ![22](https://user-images.githubusercontent.com/98762543/211158517-d9bac59b-8da7-4c5a-99d0-e9e6e5760130.PNG)
  
  - ตรวจสอบ Path Locations ให้เรียบร้อย
  ![23](https://user-images.githubusercontent.com/98762543/211158518-6a600ece-432e-4b68-a61f-c24b07b68e12.PNG)
  
  - อ่าน Statement ให้เรียบร้อย และเมื่ออ่านเสร็จแล้วให้ติ๊กที่ I have read this statement
  ![24](https://user-images.githubusercontent.com/98762543/211158519-ad9c3413-db30-4b2f-8d71-90d861f4ba34.PNG)
  
  - ตั้งค่า Default Profile และ Default Automation Network
  ![25](https://user-images.githubusercontent.com/98762543/211158520-0d0841df-6b9f-454b-8229-f5617d5dfd1f.PNG)
  
  - เลือก Template Setup ที่ต้องการติดตั้ง 
  ![26](https://user-images.githubusercontent.com/98762543/211158521-fcbdb191-97c5-4d15-b67e-d5904728b5a3.PNG)
  - กด Next 
  ![27](https://user-images.githubusercontent.com/98762543/211158522-a8ec8706-2d32-437f-b37b-f0ba8396c9af.PNG)
  
  - กด Confirm Installation และกด Install เพื่อติดตั้ง

  ![28](https://user-images.githubusercontent.com/98762543/211158524-0aa3348b-cf80-4b92-bc66-cd779c582f87.PNG)
  
  - รอติดตั้งจนสำเร็จ
  ![29](https://user-images.githubusercontent.com/98762543/211158526-2ccecd2b-6b52-44c1-9b7c-c6014b5e1fc0.PNG)
  
  ![30](https://user-images.githubusercontent.com/98762543/211158528-ddf40762-6f6c-40b0-8c2a-c36d9be24a1a.PNG)
  
  
  - เมื่อติดตั้งเสร็จ จะเจอหน้าเริ่มต้นของ cacti 
  
  - รูปเมื่อติดตั้งเสร็จเรียบร้อย
  
  ![40](https://user-images.githubusercontent.com/98762543/211161007-7fee833c-b344-4c12-97a8-4c158691e6f0.PNG)
  
  **วิธี Create devices** 
  - คลิกที่ Management แล้วเลือก Devices และคลิกที่ สัญลักษณ์ +(บวก) ด้านบน
  ![31](https://user-images.githubusercontent.com/98762543/211158530-80e0ef06-75e0-4e50-aa25-c668d55907db.PNG)
  - ใส่ข้อมูลต่างๆ
     - Description ใส่ชื่อ
     - Hostname ใส่ IP
     - Location เลือก None
     - Device Template เลือกเป็น Local Linux Machine
  ![32](https://user-images.githubusercontent.com/98762543/211158531-62ec09a3-8c5c-412e-91e2-0027c180ef27.PNG)
  - จะมี Graph ขึ้นมาให้อัตโนมัติ และสามารถเพิ่ม Graph ได้
  ![33](https://user-images.githubusercontent.com/98762543/211158532-f12082b9-9f4b-4606-b2fc-9eebebb76e4d.PNG)
  
  **วิธี Create graphs**

  ![34](https://user-images.githubusercontent.com/98762543/211158534-6251f662-4895-4aff-911c-04a6d6aa65c1.PNG)
  
  ![35](https://user-images.githubusercontent.com/98762543/211158535-cf6b76d3-1a39-4203-a186-7b0797e91d88.PNG)

   
  
  วิธี ลง plugin thold ให้เข้าไปที่ /var/www/html/cacti/plugins ให้นำไฟล์ที่โหลดมาจาก github วางไว้ในนี้ ขั้นต่อไปให้ไปที่ plugin และกด install จากนั้นจะเห็น thold ด้านบน ซึ่งเป็น plugin ที่ถูกเพิ่มเข้ามา 
  
  รูปตัวอย่าง
  
  ![38](https://user-images.githubusercontent.com/98762543/211158540-3434492d-ad4d-4a05-ae8f-59a981b90f94.PNG)
  
  ![39](https://user-images.githubusercontent.com/98762543/211158541-4a2e2ca0-a303-476e-90b6-7590eb341f9d.PNG)
  
  ![36](https://user-images.githubusercontent.com/98762543/211158536-60e0c0de-8f65-4c91-ae00-d6ed5b7ec37c.PNG)
  
  ![37](https://user-images.githubusercontent.com/98762543/211158539-ff1c9d70-be4f-421b-8d90-0316a25a282a.PNG)

  ![chrome_4dKLWNlvpW](https://user-images.githubusercontent.com/115150753/211187806-8607e2c4-46f4-492d-9ec4-21eb368fb698.png)

      

        


        
