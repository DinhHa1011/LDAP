## Cài đặt và cấu hình OpenLDAP Server
### Cài đặt
- Cập nhật các chỉ mục gói của hệ thống Linux
```
sudo apt update
```
- Cài đặt Snapd và bộ công cụ cho LDAP (ldap-utils)
``` 
sudo apt-get install slapd ldap-utils -y
```
### Cấu hình
- Sau khi cài đặt xong, cần cấu hình lại: 
```
dpkg-reconfigure slapd
```
- Trong cửa sổ hiện ra, chọn NO:
![](https://i.imgur.com/e4rUlnI.png)
- Tiếp theo, bạn sẽ được hỏi về domain của LDAP, hãy điền vào domain của bạn:
![image](https://github.com/trangnth/BizflyCloudEmail/assets/119484840/70e52ec5-5553-42b5-bd69-e4474828f951)
- Tiếp tục bạn sẽ được hỏi về tên tổ chức, công ty, thực hiện điền tên công ty, tổ chức của bạn tại đó:
![image](https://github.com/trangnth/BizflyCloudEmail/assets/119484840/f458ea99-1a82-4ef5-82c4-390e5dd95218)
- Bạn sẽ được hỏi về mật khẩu của người dùng admin trong LDAP:
![](https://i.imgur.com/Fd6A9OB.png)
- Cuối cùng là chọn Yes để lưu lại cấu hình vừa điền:
![](https://i.imgur.com/DHKx4jH.png)


- Bạn có thể xem lại cấu hình hiện giờ của LDAP bằng câu lệnh :
```
slapcat
```

- Tạo tài khoản người dùng trong OpenLDAP: Trước hết, bạn cần tạo một Organization Unit (OU) để lưu trữ thông tin người dùng và nhóm người dùng:
    ```
    vim users-ou.ldif
    ```
    + Thêm đoạn cấu hình sau, lưu ý thay đổi domain là domain của bạn:
    ```
    dn: ou=people,dc=pikab,dc=in
    objectClass: organizationalUnit
    objectClass: top
    ou: people

    dn: ou=groups,dc=pikab,dc=in
    objectClass: organizationalUnit
    objectClass: top
    ou: groups
    ```
     + Lưu và đóng file
- Điều chỉnh các điều khiển truy cập cơ sở dữ liệu SLAPD bằng cách tạo tệp sau:
    ```
    vim update-mdb-acl.ldif
    ```
    + Thêm đoạn cấu hình sau vào file:
    ```
    dn: olcDatabase={1}mdb,cn=config
    changetype: modify
    replace: olcAccess
    olcAccess: to attrs=userPassword,shadowLastChange,shadowExpire
      by self write
      by anonymous auth
      by dn.subtree="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" manage 
      by dn.exact="cn=readonly,ou=people,dc=pikab,dc=in" read 
      by * none
    olcAccess: to dn.exact="cn=readonly,ou=people,dc=pikab,dc=in" by dn.subtree="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" manage by * none
    olcAccess: to dn.subtree="dc=pikab,dc=in" by dn.subtree="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" manage
      by users read 
      by * none
    ```
    + Lưu và đóng file
- Để cập nhật cơ sở dữ liệu ACL với những file trên thực thi câu lệnh sau: 
    ```
    ldapadd -Y EXTERNAL -H ldapi:/// -f update-mdb-acl.ldif
    ```
    + Bạn sẽ nhận được thông tin như bên dưới: 
    ```
    SASL/EXTERNAL authentication started
    SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
    SASL SSF: 0
    modifying entry "olcDatabase={1}mdb,cn=config"
    ```
- Cập nhật thông tin OU bằng câu lệnh sau:
    ```
    ldapadd -Y EXTERNAL -H ldapi:/// -f users-ou.ldif
    ```
     + Bạn sẽ nhận được thông tin như bên dưới: 
    ```
    SASL/EXTERNAL authentication started
    SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
    SASL SSF: 0
    adding new entry "ou=people,dc=pikab,dc=in"
    adding new entry "ou=groups,dc=pikab,dc=in"
    ```
- Để tạo tài khoảng người dùng mới thực hiện tạo file như sau: 
    ```
    vim test.ldif
    ```
    + Thêm đoạn cấu hình sau vào file Lưu ý: thay đổi phù hợp thông tin với ou và domain của bạn.
    ```
    dn: uid=test,ou=people,dc=pikab,dc=in
    objectClass: inetOrgPerson
    objectClass: posixAccount
    objectClass: shadowAccount
    uid: test
    cn: test
    sn: test
    loginShell: /bin/bash
    uidNumber: 10000
    gidNumber: 10000
    homeDirectory: /home/test
    shadowMax: 60
    shadowMin: 1
    shadowWarning: 7
    shadowInactive: 7
    shadowLastChange: 0

    dn: cn=test,ou=groups,dc=pikab,dc=in
    objectClass: posixGroup
    cn: test
    gidNumber: 10000
    memberUid: test
    ```
    + Lưu và đóng file
- Thực thi lệnh sau để thêm người dùng vào cơ sở dữ liệu:
    ```
    ldapadd -Y EXTERNAL -H ldapi:/// -f test.ldif
    ```
    + Bạn sẽ nhập được thông tin như bên dưới khi đã thêm thành công
    ```
    SASL/EXTERNAL authentication started
    SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
    SASL SSF: 0
    adding new entry "uid=test,ou=people,dc=pikab,dc=in"
    adding new entry "cn=test,ou=groups,dc=pikab,dc=in
    ```
- Tiếp theo, bạn hãy sử dụng câu lệnh bên dưới để đặt mật khẩu cho người dùng mới thêm:
    ```
    ldappasswd -H ldapi:/// -Y EXTERNAL -S "uid=test,ou=people,dc=pikab,dc=in"
    ```
- Tạo Bind DN trên OpenLDAP
    + Để có thể truy vấn trên LDAP database, bạn cần phải tạo một Bind DN tương ứng. Thực hiện tạo mới Bind DN bằng câu lệnh sau:
    ```
    slappasswd
    ```
    + Sau khi nhập mật khẩu, bạn sẽ nhận được đoạn mật khẩu đã mã hoá như bên dưới:
    ```
    New password: 
    Re-enter new password: 
    {SSHA}DhjyJN5akaj2etaFKoyeAY8QMgSD/OTb
    ```
    + Tiếp theo, bạn hãy tạo một file :
    ```
    vim readonly-user.ldif
    ```
    + Thêm những dòng cấu hình sau vào file
        + Lưu ý: userPassword là đoạn mật khẩu đã mã hoá có được từ bước trên.
    ```
    dn: cn=readonly,ou=people,dc=pikab,dc=in
    objectClass: organizationalRole
    objectClass: simpleSecurityObject
    cn: readonly
    userPassword: {SSHA}DhjyJN5akaj2etaFKoyeAY8QMgSD/OTb
    description: Bind DN user for LDAP Operations
    ```
    + Lưu và đóng file
- Thêm người dùng BIND vào cơ sở dữ liệu với lệnh sau:
```
ldapadd -Y EXTERNAL -H ldapi:/// -f readonly-user.ldif
```
- Kiểm tra lại tài khoản trên có thể sử dụng để truy vấn được hay khôn như sau:
    ```
    ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config '(olcDatabase={1}mdb)' olcAccess
    ```
    + Bạn sẽ nhận được thông tin như bên dưới: 
    ```
    dn: olcDatabase={1}mdb,cn=config
    olcAccess: {0}to attrs=userPassword,shadowLastChange,shadowExpire by self writ
     e by anonymous auth by dn.subtree="gidNumber=0+uidNumber=0,cn=peercred,cn=ext
     ernal,cn=auth" manage  by dn.exact="cn=readonly,ou=people,dc=pikab,dc=in" 
     read  by * none
    olcAccess: {1}to dn.exact="cn=readonly,ou=people,dc=pikab,dc=in" by dn.subt
     ree="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" manage by * non
     e
    olcAccess: {2}to dn.subtree="dc=pikab,dc=in" by dn.subtree="gidNumber=0+uid
     Number=0,cn=peercred,cn=external,cn=auth" manage by users read  by * none
    ```
## Cài đặt và Cấu hình phpLDAPadmin
- Để tiện cho việc quản lý LDAP database, bạn hãy cài đặt phpLDAPadmin bằng lệnh như bên dưới:
```
apt-get install phpldapadmin -y
```
- Sau khi cài đặt xong, thực hiện chỉnh sửa file cấu hình tạo /etc/phpldapamin/config.php:
```
vim /etc/phpldapadmin/config.php
```
- Thay đổi nội dung file như bên dưới rồi lưu lại:
    - Lưu ý địa chỉ host, domain là đại chỉ host và domain của bạn:
    ```
    $servers->setValue('server','name','My LDAP Server');
    $servers->setValue('server','host','69.87.216.102');
    $servers->;setValue('server','base',array('dc=pikab,dc=in'));
    $servers->setValue('login','auth_type','session');
    $servers->setValue('login','bind_id','cn=admin,dc=pikab,dc=in');
    $servers->setValue('auto_number','min',array('uidNumber'=>10000,'gidNumber'=>10000));
    ```
    - Lưu và đóng file

- Cấu hình Apache cho Phpldapadmin
phpLDAPadmin có file cấu hình mặc định cho Apache tại đường dẫn /etc/apache2/conf-available/phpldapadmin.conf chú ý không chỉnh sửa. 

Tiếp theọc tắt website mặc định của Apache và khởi động lại Apache để cập nhật cấu hình:
```
a2dissite 000-default.conf
```
```
systemctl restart apache2
```


- Truy cập Phpldapadmin UI
    + Như vậy, bạn đã có thể truy cập vào phpLDAPadmin thông qua trình duyệt web với địa chỉ: http://your-server-ip/phpldapadmin:

    ![](https://i.imgur.com/SfC31Sz.png)

    + Khi bạn ấn vào nút login, một form đăng nhập sẽ hiện ra.
    + Thực hiện đăng nhập với Bind DN là người dùng admin hoặc người dùng readonly như bên trên là bạn có thể xem, quản lý, chỉnh sửa LDAP database.

## References 
* [How to Install and Configure OpenLDAP and phpLDAPadmin on Ubuntu 20.04](https://www.howtoforge.com/how-to-install-and-configure-openldap-phpldapadmin-on-ubuntu-2004/)
* [CÀI ĐẶT OPENLDAP VÀ PHPLDAPADMIN TRÊN UBUNTU 20.04](https://tel4vn.edu.vn/blog/install-openldap-and-phpldapadmin-ubuntu-20-04/)
