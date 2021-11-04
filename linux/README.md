# 说明
最近在wsl2子系统下折腾spark和tidb，执行的过程中也遇到了一些问题。

## 执行sudo echo时报Permission denied
参考这篇文章[避免’sudo echo x >’ 時’Permission denied’](https://www.itread01.com/content/1546379105.html)

**注意**
```
# 切换root
sudo -s
# 切换指定用户名
sudo su huxb
```

## 执行spark-shell时报metastore_db cannot be created
使用sudo命令提权。。。```sudo bin/spark-shell```
