Openldap-on-CentOS
==================
OpenLdap thường xuất hiện trong những hệ thống quản lý tập trung, được dùng mới mục đích xác thực người dùng. Đây là phần cơ bản của hệ thống. Trong bài viết này tôi sẽ hướng dẫn các bạn cách triển khai hệ thống OpenLdap đơn giản.

Tài liệu triển khai OpenLdap thử nghiệm
 
Domain name :   khanhnn.com
System IP        :   192.168.1.25
Software      :    Asianux server 3, openldap
 
     Bước 1:Tiến hành kiểm tra các gói cài đặt
            openldap
            openldap-clients
            openldap-devel
            nss-ldap
            openldap-servers
-Thiết lập Openldap khởi động khi bật máy
            [root@ldap ~]# chkconfig --levels 235 ldap on
            [root@ldap ~]# service ldap start
 
  Bước 2: Tạo mật khẩu LDAP cho root
            [root@ldap ~]# slappasswd
            New password:
            Re-enter new password:          {SSHA}cWB1VzxDXZLf6F4pwvyNvApBQ8G/DltW
 
Bước 3. Cấu hình LDAP Server
            -Tạo thư mục lưu database
            #mkdir -p /var/lib/ldap/khanhnn.com
            #chown ldap:ldap /var/lib/ldap/khanhnn.com
         -Tạo file cấu hình cho database
#cp /etc/openldap/DB_CONFIG.example/var/lib/ldap/khanhnn.com/DB_CONFIG
           
         -Cấu hình  file /etc/openldap/slap.conf sửa đổi các dòng 68,69,70,71,80
            #68 database        bdb
            #69 suffix             " dc=khanhnn,dc=com"
            #70 rootdn          " cn=Manager, dc=khanhnn,dc=com "
            #71 rootpw          {SSHA}cWB1VzxDXZLf6F4pwvyNvApBQ8G/DltW
 
            #80 directory /var/lib/ldap
           
-Tạo người dùng ldap: Tạo user test1,test2 và đặt password
            [root@ldap ~]# useradd test1
            [root@ldap ~]# passwd test1
            [root@ldap ~]# useradd test2
            [root@ldap ~]# passwd test2
            -Thay đổi cấu hình mặc định file                    /usr/share/openldap/migration/migrate_common.ph
                         #71 $DEFAULT_MAIL_DOMAIN = " khanhnn.com";
                         #74 $DEFAULT_BASE = " dc=khanhnn,dc=com";
     Bước 4 :Chuyển user hệ thống sang LDAP
            [root@ldap ~]# grep root /etc/passwd > /etc/openldap/passwd.root
            [root@ldap ~]# grep test1 /etc/passwd > /etc/openldap/passwd.test1
            [root@ldap ~]# grep test2 /etc/passwd > /etc/openldap/passwd.test2
           
            -Chuyển đổi file passwd sang định dạng file của LDAP là ldif 
            [root@ldap ~]# /usr/share/openldap/migration/migrate_passwd.pl   /etc/openldap/passwd.root /etc/openldap/root.ldif
            [root@ldap ~]# /usr/share/openldap/migration/migrate_passwd.pl /etc/openldap/passwd.test1 /etc/openldap/test1.ldif
            [root@ldap ~]# /usr/share/openldap/migration/migrate_passwd.pl   /etc/openldap/passwd.test2 /etc/openldap/test2.ldif
 
Bước 5. Cập nhật ”Manager" của LDAP Server trong file root.ldif 
            [root@ldap ~]# vi /etc/openldap/root.ldif
               #1 dn: uid=root,ou=People,dc=khanhnn,dc=com 
              #2 uid: root
               #3 cn: Manager
               #4 objectClass: account
Bước 6.Tạo tên miền dạng ldif file (/etc/openldap/khanhnn.com.ldif)
            [root@ldap ~]# vi /etc/openldap/khanhnn.com.ldif
             dn: dc=khanhnn,dc=com
             dc: khanhnn
             description: LDAP Admin
             objectClass: dcObject
             objectClass: organizationalUnit
             ou: rootobject
             dn: ou=People, dc=khanhnn,dc=com 
            ou: People
             description: Users of KhanhNN
             objectClass: organizationalUnit
 
            -Thêm  Domain ldif file
            [root@ldap ~]# ldapadd -x -D "cn=Manager,dc=dc=khanhnn,dc=com" -W-f              /etc/openldap/caobang.gov.vn.ldif
             Enter LDAP Password:
             adding new entry "dc=khanhnn,dc=com"
            adding new entry "ou=People, dc=khanhnn,dc=com"
           
            -Thêm người dùng vào LDAP
            [root@ldap ~]# ldapadd -x -D "cn=Manager,dc=khanhnn,dc=com" -W -f        /etc/openldap/root.ldif
             Enter LDAP Password:
             adding new entry "uid=root,ou=People,dc=khanhnn,dc=com"
             adding new entry "uid=operator,ou=People,dc=khanhnn,dc=com"
            [root@ldap ~]# ldapadd -x -D "cn=Manager,dc=khanhnn,dc=com" -W -f        /etc/openldap/test1.ldif
            Enter LDAP Password:
            adding new entry "uid=test1,ou=People,dc=khanhnn,dc=com”
            [root@ldap ~]# ldapadd -x -D "cn=Manager,dc=khanhnn,dc=com" -W -f        /etc/openldap/test2.ldif
            Enter LDAP Password:
            adding new entry "uid=test2,ou=People,dc=khanhnn,dc=com"
 
            Bước 7: Khởi tạo lại dịch vụ để nhận các thay đổi
            [root@ldap ~]# service ldap restart
 
            Bước 8: TestLDAP Server
                        -Đưa ra tất cả thông tin người dùng
            [root@ldap ~]# ldapsearch -x -b ''''''''dc=khanhnn,dc=com'''''''' ''''''''(objectclass=*)''''''''
