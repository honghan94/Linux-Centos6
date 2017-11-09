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
    script "/home/keepalived/check.sh"
    interval 2
}

vrrp_instance VI_1 {
    interface eth0
    state MASTER
    priority 200

    virtual_router_id 50
    unicast_src_ip 10.0.0.11
    unicast_peer {
        10.0.0.12
    }

    authentication {
        auth_type PASS
        auth_pass password
    }

    track_script {
        chk_haproxy
    }

    virtual_ipaddress {
        10.0.0.10/24
    }
}
```

### Server01 script
```
vrrp_script chk_haproxy {
    script "/home/keepalived/check.sh"
    interval 2
}

vrrp_instance VI_1 {
    interface eth0
    state BACKUP
    priority 100

    virtual_router_id 50
    unicast_src_ip 10.0.0.12
    unicast_peer {
        10.0.0.11
    }

    authentication {
        auth_type PASS
        auth_pass password
    }

    track_script {
        chk_haproxy
    }

    virtual_ipaddress {
        10.0.0.10/24
    }
}
```
