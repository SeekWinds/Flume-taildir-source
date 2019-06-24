# Flume-taildir-source
修改Flume taildir source源码,使其支持递归读取指定目录下的文件变化
之前的Flume在使用Flume taildir作为source的时候,只能监听文件夹下面的指定后缀的文件的变化,但是如果文件夹下面有递归文件夹,就不能监听递归文件夹中指定后缀的文件的变化.
例如:
test目录下只有a.txt一个文件,我们在flume的配置文件配置监听,在不修改源码的情况下,只会监听a.txt文件的变化
[hadoop@hadoop001 /opt/scripts/producer/test] 10.25.13.21 
$ ll
total 0
-rw-rw-r--. 1 hryadm hryadm 0 Jun 24 09:59 a.txt
但是如果出现了
