# Keepalived

The steps following is used of installation of Keepalived (Active - Passive)

### Install
```
wget http://www.keepalived.org/software/keepalived-1.3.5.tar.gz
tar -xvf keepalived-1.3.5.tar.gz
mv keepalived-1.3.5 keepalived
cd keepalived
yum -y groupinstall 'Development Tools'
yum -y install openssl-devel
./configure
make
make install
cp /usr/local/etc/init/keepalived.conf /etc/init/
cp /usr/local/etc/sysconfig/keepalived /etc/syconfig/
mkdir -p /etc/keepalived
vi /etc/keepalived/keepalived.conf
vi /home/keepalived/check.sh
chmod +x /home/keepalived/check.sh
start keepalived
```

### Server01 script
```
vrrp_script chk_haproxy {
    script "/home/keepalived/check.sh" #script to check is the monitored process still running
    interval 2
}

vrrp_instance VI_1 {
    interface eth0
    state MASTER
    priority 200 #higher will be master

    virtual_router_id 50
    unicast_src_ip 10.0.0.11 #the server ip
    unicast_peer {
        10.0.0.12 #other server's ip
    }

    authentication {
        auth_type PASS
        auth_pass password
    }

    track_script {
        chk_haproxy
    }

    virtual_ipaddress {
        10.0.0.10/24 #the virtual ip, make sure it is same on both server and use an unused ip
    }
}
```

### Server02 script
```
vrrp_script chk_haproxy {
    script "/home/keepalived/check.sh"
    interval 2
}

#script to check is the monitored process still running
#  fall f      # If script returns non-zero f times in succession, enter FAULT state
#  rise r      # If script returns zero r times in succession, exit FAULT state
#  timeout t   # Wait up to t seconds for script before assuming non-zero exit code
#  weight w    # Reduce priority by w on fall
#since im not using the feature above, the script will automatically start the service if it is down

vrrp_instance VI_1 {
    interface eth0
    state BACKUP
    priority 100  #lower will be slave

    virtual_router_id 50
    unicast_src_ip 10.0.0.12 #the server ip
    unicast_peer {
        10.0.0.11 #other server's ip
    }

    authentication {
        auth_type PASS
        auth_pass password
    }

    track_script {
        chk_haproxy
    }

    virtual_ipaddress {
        10.0.0.10/24 #the virtual ip
    }
}
```
### Simple script check.sh
```
#!/bin/bash

#sum the amount of process using the name 'nginx'
#if the sum is zero, start the service
A="$(ps -C nginx --no-header | wc -l)" 
if [[ $A -eq 0 ]]
then
/usr/local/nginx/sbin/nginx
fi
exit
```