# FastDFS Cluster
The steps following is used of installation of FastDFS Cluster (multi node single group) 

wget https://github.com/happyfish100/fastdfs/archive/master.zip  
wget https://github.com/happyfish100/libfastcommon/archive/master.zip  
wget https://github.com/happyfish100/fastdfs-nginx-module/archive/master.zip  
wget http://nginx.org/download/nginx-1.12.2.tar.gz  

### Install
cd libfastcommon  
./make.sh  
./make.sh install  
\#create symbolic link  
ln -s /usr/lib64/libfastcommon.so /usr/local/lib/libfastcommon.so  
ln -s /usr/lib64/libfastcommon.so /usr/lib/libfastcommon.so  
ln -s /usr/lib64/libfdfsclient.so /usr/local/lib/libfdfsclient.so  
ln -s /usr/lib64/libfdfsclient.so /usr/lib/libfdfsclient.so  
cd fastdfs  
./make.sh  
./make.sh install  

\#Edit Tracker  
vi /etc/fdfs/tracker.conf  

\#Start Tracker  
mkdir -p /data/fastdfs/tracker/  
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf  

\#Edit Storage  
vi /etc/fdfs/storage.conf  

\#Start Storage  
mkdir -p /data/fastdfs/storage/  
mkdir -p /data/fastdfs/data/  
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf  

\#Configure Nginx  
cp $fdfs_extract/conf/http.conf /etc/fdfs  
cp $fdfs_extract/conf/mime.types /etc/fdfs  

\#Install dependency  
yum install -y gcc gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel  
./configure \  
--prefix=/usr/local/nginx \  
--pid-path=/var/local/nginx/nginx.pid \  
--lock-path=/var/lock/nginx/nginx.lock \  
--error-log-path=/var/log/nginx/error.log \  
--http-log-path=/var/log/nginx/access.log \  
--with-http_gzip_static_module \  
--http-client-body-temp-path=/var/temp/nginx/client \  
--http-proxy-temp-path=/var/temp/nginx/proxy \  
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \  
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \  
--http-scgi-temp-path=/var/temp/nginx/scgi \  
--add-module=../fastdfs-nginx-module/src  
make  
make install  

\#copy mod_fastdfs.conf  
cp $extract_path/src/mod_fastdfs.conf /etc/fdfs/  
vi /etc/fdfs/mod_fastdfs.conf  

vi /usr/local/nginx/conf/nginx.conf  

\#Start nginx  
mkdir -p /data/fastdfs/nginx-module  
/usr/local/nginx/sbin/nginx  
  
### tracker.conf  
base_path=/data/fastdfs/tracker/  

### storage.conf
base_path=/data/fastdfs/storage/  
store_path0=/data/fastdfs/data/  
tracker_server=10.0.5.21:22122  
\#tracker_server=10.0.5.22:22122  
\#for tracker cluster use  

### mod_fastdfs.conf  
tracker_server=10.0.5.21:22122  
tracker_server=10.0.5.22:22122  
url_have_group_name = true  
base_path=/data/fastdfs/nginx-module  
store_path0=/data/fastdfs/data/  

### nginx.conf  
location /group1/M00 {  
    ngx_fastdfs_module;  
}  

### TODO  
\#Configure Keepalived to achieve auto switch over on two nodes