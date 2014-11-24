Hướng dẫn cài đặt OpenLdap trên CentOS 6

OpenLdap thường xuất hiện trong những hệ thống quản lý tập trung, được dùng mới mục đích xác thực người dùng. Đây là phần cơ bản của hệ thống. Trong bài viết này tôi sẽ hướng dẫn các bạn cách triển khai hệ thống OpenLdap đơn giản.
Mô hình thử nghiệm như sau:

Domain name: khanhnn.com

IP: 192.168.0.124

OS: CentOS

Bước 1:

Cài đặt các gói cần thiết:

yum install openldap-servers openldap-clients

yum install sssd perl-LDAP.noarch

Bước 2:

LDAP sẽ cần một cơ sở dữ liệu mẫu, bạn cần copy chúng. Chúng ta sử dụng câu lệnh sau:

cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG

Bước 3: 
