# HAProxy
The steps following is used of installation of HAProxy

### Install
yum install gcc pcre-static pcre-devel wget -y  
wget http://www.haproxy.org/download/1.7/src/haproxy-1.7.9.tar.gz
tar -xvf haproxy-1.7.9.tar.gz  
cd haproxy-1.7.9  
make TARGET=linux2628  
make install  
mkdir -p /etc/haproxy  
mkdir -p /var/lib/haproxy  
touch /var/lib/haproxy/stats  
ln -s /usr/local/sbin/haproxy /usr/sbin/haproxy  
cp examples/haproxy.init /etc/init.d/haproxy  
chmod +x /etc/init.d/haproxy  
chkconfig haproxy on  
haproxy -v  
vi /etc/haproxy/haproxy.cfg  
service haproxy start  

### haproxy.cfg
global
   log /dev/log local0
   log /dev/log local1 notice
   chroot /var/lib/haproxy
   stats timeout 30s
   user haproxy
   group haproxy
   daemon

defaults
   log global
   mode http
   option httplog
   option dontlognull
   timeout connect 5000
   timeout client 50000
   timeout server 50000

frontend http_front
   bind 0.0.0.0:80
   stats uri /haproxy?stats
   default_backend http_back

backend http_back
   balance roundrobin
   server web01 10.0.3.20:8080 check
   server web02 10.0.3.21:8080 check

listen stats
   bind *:8181
   stats enable
   stats uri /
   stats realm Haproxy\ Statistics
   stats auth admin:password