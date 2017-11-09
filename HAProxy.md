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
```
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
   bind *:80  
   stats uri /haproxy?stats  

   acl Web_URL hdr_dom(host) -i web.example.com  
   acl Api_URL hdr_dom(host) -i api.example.com  
   acl File_URL hdr_dom(host) -i file.example.com  

   use_backend Web if Web_URL  
   use_backend Api if Api_URL  
   use_backend File if File_URL  

backend Web  
#  balance roundrobin  
   mode http  
   server web01 10.0.0.11:8080  
   server web02 10.0.0.12:8080  

backend Api  
#  balance roundrobin  
   mode http  
   reqrep ^([^\ :]\*)\ /(.\*)     \1\ /api/\2  
   /#this is to forward incoming address and append it, ex: api.example.com ->  10.0.0.11:8080/api/  
   server api01 10.0.0.11:8080  
   server api02 10.0.0.12:8080  

backend File  
#  balance roundrobin  
   mode http  
   server web-file 10.0.0.21:8080  

listen stats  
   bind *:8181  
   stats enable  
   stats uri /  
   stats realm Haproxy\ Statistics  
   stats auth admin:password  #don't forget to change password  
```