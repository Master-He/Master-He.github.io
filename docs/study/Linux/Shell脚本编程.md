```shell
test -n "$(ip route| grep 200.200.0.0/22)";echo $?
```



```shell
if [ -n "$(ip route| grep 200.200.0.0/22)" ]
then
    ip route del 200.200.0.0/22;
    echo "del route 200.200.0.0/22"
fi
```



查看/etc目录下哪个文件的内容包含"system_set_cfg.ini"

egrep -rn system_set_cfg.ini /etc

