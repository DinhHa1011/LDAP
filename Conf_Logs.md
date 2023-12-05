# Logging Slapd
# Table of contents

- [Logging Slapd](#logging-slapd)
  - [OpenLDAP](#openldap)
  - [Rsyslog](#rsyslog)
  - [Test](#test)
  - [Note](#note)
  - [References](#references)
## OpenLDAP
- Tạo file chỉnh sửa cấu hình log của slapd
```
vim add_slapdlog.ldif
```
- Thêm các dòng cấu hình sau
```
dn: cn=config
changeType: modify
replace: olcLogLevel
olcLogLevel: stats
```
- Áp dụng cấu hình trên vào database LDAP bằng lệnh
```
ldapmodify -Y external -H ldapi:/// -f add_slapdlog.ldif
```
- Khởi động lại OpenLDAP 
```
systemctl force-reload slapd
```
## Rsyslog
- Tạo file cấu hình 
```
vim /etc/rsyslog.d/10-slapd.conf
```
- Thêm các dòng cấu hình sau
```
$template slapdtmpl,"[%$DAY%-%$MONTH%-%$YEAR% %timegenerated:12:19:date-rfc3339%] %app-name% %syslogseverity-text% %msg%\n"
local4.*    /var/log/slapd.log;slapdtmpl
```
- Khởi động lại Rsyslog để áp dụng 
```
service rsyslog restart
```
## Test 
- Chạy câu lệnh test: Chú ý domain là domain server ldap của bạn 
```
ldapsearch -Y external -H ldapi:/// -b dc=example,dc=com
```
- Kiểm tra file log
``` 
tail -f /var/log/slapd.log
```

## References 
* [The slapd Configuration File](https://www.openldap.org/doc/admin24/slapdconfig.html)
* [Enable the production of Openldap Log file](https://tutoriels.meddeb.net/openldap-tutorial-log/)
