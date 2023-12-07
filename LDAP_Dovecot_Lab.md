## Tích hợp LDAP vs Dovecot (Lab)
### Dựng LDAP với domain muốn tích hợp
https://github.com/DinhHa1011/LDAP/blob/main/Conf_LDAP.md
### Cấu hình tích hợp LDAP với Dovecot
- Tạo file kết nối tới database LDAP
```
vim /etc/dovecot-ldap.conf.ext
```
```
hosts = ip LDAP
tls = no
debug_level = stats
dn = cn=admin,dc=hadt,dc=space
dnpass = 'HaNoi2021'
base = ou=people,dc=hadt,dc=space
user_filter = (&(objectClass=posixAccount)(uid=%u))
iterate_filter = (objectClass=*)
pass_filter = (&(objectClass=posixAccount)(uid=%u))
```
- Trong file
``` 
vim /etc/dovecot/conf.d/10-auth.conf
```
comment `!include auth-mail.conf.ext` sau đó thêm dòng dưới đây
```
!include auth-ldap.conf.ext
```
- Sửa cấu hình auth sang LDAP
```
vim /etc/dovecot/conf.d/auth-ldap.conf.ext
```
```
passdb {
  driver = ldap
  args = /etc/dovecot/dovecot-ldap.conf.ext
}
userdb {
  driver = ldap
  args = /etc/dovecot/dovecot-ldap.conf.ext
}
```
- restart dovecot để áp dụng cấu hình
