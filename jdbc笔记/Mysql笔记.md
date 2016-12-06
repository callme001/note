
mysql开启远程访问：

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'passowrd' WITH GRANT OPTION;

flush privileges;
```