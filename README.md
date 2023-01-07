# SPCN-NMS
# Cacti
**ขั้นตอนการติดตั้ง Cacti**
1. คำสั่งอัพเดท package ในระบบ

        $ sudo apt-get update
        
2. คำสั่งติดตั้งโปรแกรม rrdtool

        $ sudo apt-get install rrdtool
        
3. คำสั่งติดตั้งโปรแกรม snmp 
  
        $ sudo apt-get install snmp snmpd
        
4. คำสั่งให้ snmp เริ่มการทำงาน 
  
        $ sudo service snmpd start                    //For SysVinit Systems
        $ sudo systemctl start snmpd.service          //For systemd Systems
        
5. คำสั่งติดตั้ง Cacti และ Cacti-Spine

        $ sudo apt-get install cacti
        $ sudo apt-get install cacti-spine
