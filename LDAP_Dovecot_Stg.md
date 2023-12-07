## Tích hợp LDAP vs Dovecot (Staging)
### Dựng LDAP với domain muốn tích hợp 
https://github.com/DinhHa1011/LDAP/blob/main/Conf_LDAP.md
### Cấu hình tích hợp LDAP & Dovecot
- Tạo file kết nối tới database ldap
```
vim /etc/dovecot/dovecot-ldap.conf
```
```
hosts = ip ldap
tls = no
debug_level = stats
dn = cn=admin,dc=pikab,dc=in
dnpass = 'HaNoi2021'
base = ou=people,dc=pikab,dc=in
user_filter = (&(objectClass=posixAccount)(uid=%n))
iterate_filter = (objectClass=*)
pass_filter = (&(objectClass=posixAccount)(uid=%n))
```
```
vim /etc/dovecot/dovecot-ldap-fallback.conf
```
```
hosts = localhost
tls = no
debug_level = stats
dn = cn=admin,dc=pikab,dc=in
dnpass = 'HaNoi2021'
base = ou=people,dc=pikab,dc=in
user_filter = (&(objectClass=posixAccount)(uid=%n))
iterate_filter = (objectClass=*)
pass_filter = (&(objectClass=posixAccount)(uid=%n))
```
- Sửa cấu hình authen từ mysql sang ldap
```
vim /etc/dovecot/dovecot.conf
```
```
userdb {
    args = /etc/dovecot/dovecot-ldap.conf
    driver = ldap
}
passdb {
    args = /etc/dovecot/dovecot-ldap.conf
    driver = ldap
}
userdb {
    args = /etc/dovecot/dovecot-ldap-fallback.conf
    driver = ldap
}
passdb {
    args = /etc/dovecot/dovecot-ldap-fallback.conf
    driver = ldap
}
```
- restart dovecot để áp dụng cấu hình
