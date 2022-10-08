```shell
#!/bin/bash
if [ ! -d "/root/.pip" ]; then
  mkdir /root/.pip
fi

if [ ! -f "/root/.pip/pip.conf" ]; then
  echo  "[global]" >> /root/.pip/pip.conf
  echo  "index-url = http://xxx.org/pypi/simple" >> /root/.pip/pip.conf
  echo  "extra-index-url = http://xxx.org/sfpypi/simple" >> /root/.pip/pip.conf
  echo  "[install]" >> /root/.pip/pip.conf
  echo  "trusted-host = xxx.org" >> /root/.pip/pip.conf
fi


grep 'xxx.org' /etc/hosts

if [ $? -ne 0 ] ;then
    echo 'ip.ip.ip.ip  xxx.org'  >>  /etc/hosts
else
  echo 1;
fi

pip install coverage==5.5
pip install inflect==2.1.0
pip install Pygments==2.5.2
pip install MarkupSafe==1.1.1
pip install Jinja2==2.7.1
pip install diff-cover==2.6.1
```

