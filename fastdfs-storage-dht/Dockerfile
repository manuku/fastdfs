#base on docker centos 
FROM centos:7

#AUTHOR
MAINTAINER imlzw <imlzw@imlzw.com>

#安装wget
RUN yum install -y wget

#更换yum源 
RUN mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup \
	&& wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo \
	&& yum clean all \
	&& yum makecache 
	
#安装工具集
RUN yum install -y zlib zlib-devel pcre pcre-devel gcc gcc-c++ openssl openssl-devel libevent libevent-devel perl unzip net-tools git

	
#配置环境变量
ENV CENTOS_VERSION=7 \
	USER_ROOT=/home/imlzw \
	FASTDFS_PATH=/home/imlzw/fastdfs \
    FASTDFS_STORAGE_PATH=/home/imlzw/fastdfs/storage \
    FASTDFS_STORAGE_DATA_PATH=/home/imlzw/fastdfs/storage/data \
    FASTDHT_PATH=/home/imlzw/fastdfs/fastdht \
    NGINX_VERSION=1.11.7 \
    BERKELEY_DB_VERSION=6.2.23

#创建必要的目录
RUN mkdir -p ${FASTDFS_PATH}/download \
 && mkdir -p ${FASTDFS_STORAGE_PATH} \
 && mkdir -p ${FASTDFS_PATH}/client \
 && mkdir -p ${FASTDFS_PATH}/mod \
 && mkdir -p ${FASTDHT_PATH}

#下载
RUN wget https://github.com/happyfish100/libfastcommon/archive/master.zip -P ${FASTDFS_PATH}/download/libfastcommon
RUN wget https://github.com/happyfish100/fastdfs/archive/master.zip -P ${FASTDFS_PATH}/download/fastdfs
RUN wget https://github.com/happyfish100/fastdfs-nginx-module/archive/master.zip -P ${FASTDFS_PATH}/download/nginx_module 
RUN wget https://github.com/happyfish100/fastdht/archive/master.zip -P ${FASTDFS_PATH}/download/fastdht 
RUN wget "http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz" -P ${FASTDFS_PATH}/download/nginx 
RUN wget "http://download.oracle.com/berkeley-db/db-${BERKELEY_DB_VERSION}.tar.gz" -P ${FASTDFS_PATH}/download 
RUN unzip ${FASTDFS_PATH}/download/libfastcommon/master.zip -d ${FASTDFS_PATH}/download/libfastcommon \
 && unzip ${FASTDFS_PATH}/download/fastdfs/master.zip -d ${FASTDFS_PATH}/download/fastdfs \
 && unzip ${FASTDFS_PATH}/download/nginx_module/master.zip -d ${FASTDFS_PATH}/download/nginx_module \
 && unzip ${FASTDFS_PATH}/download/fastdht/master.zip -d ${FASTDFS_PATH}/download/fastdht \
 && tar zxvf ${FASTDFS_PATH}/download/nginx/nginx-${NGINX_VERSION}.tar.gz -C ${FASTDFS_PATH}/download/nginx \
 && tar zxvf ${FASTDFS_PATH}/download/db-${BERKELEY_DB_VERSION}.tar.gz -C ${FASTDFS_PATH}/download

#安装libfastcommon
WORKDIR ${FASTDFS_PATH}/download/libfastcommon/libfastcommon-master
RUN ["/bin/bash", "-c", "./make.sh && ./make.sh install"]

#安装fastdfs
WORKDIR ${FASTDFS_PATH}/download/fastdfs/fastdfs-master
RUN ["/bin/bash", "-c", "./make.sh && ./make.sh install && cp conf/* /etc/fdfs/"] 
 

#安装nginx
WORKDIR ${FASTDFS_PATH}/download/nginx/nginx-${NGINX_VERSION}
RUN ./configure --add-module=${FASTDFS_PATH}/download/nginx_module/fastdfs-nginx-module-master/src \
 && make \
 && make install \
 && cp ${FASTDFS_PATH}/download/nginx_module/fastdfs-nginx-module-master/src/mod_fastdfs.conf /etc/fdfs/
 
 
#安装berkeley db
WORKDIR ${FASTDFS_PATH}/download/db-${BERKELEY_DB_VERSION}/build_unix
RUN ../dist/configure --prefix=/usr/local/db-${BERKELEY_DB_VERSION} \
 && make \
 && make install

#安装fastdht
WORKDIR ${FASTDFS_PATH}/download/fastdht/fastdht-master
RUN sed -i "s?CFLAGS='-Wall -D_FILE_OFFSET_BITS=64 -D_GNU_SOURCE'?CFLAGS='-Wall -D_FILE_OFFSET_BITS=64 -D_GNU_SOURCE -I/usr/local/db-${BERKELEY_DB_VERSION}/include/ -L/usr/local/db-${BERKELEY_DB_VERSION}/lib/'?" make.sh 
RUN ["/bin/bash", "-c", "./make.sh && ./make.sh install"] 
RUN cp /usr/local/db-${BERKELEY_DB_VERSION}/lib/libdb-${BERKELEY_DB_VERSION:0:3}.so /usr/lib64

#添加初始化配置
RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
COPY fdht_config/* /etc/fdht/
COPY fdfs_config/* /etc/fdfs/
COPY fdfs_config/* /etc/fdfs/
COPY nginx_config/nginx.conf /usr/local/nginx/conf/nginx.conf
COPY ["start","stop","/home/imlzw/"]
RUN chmod +x /home/imlzw/start \
  && chmod +x /home/imlzw/stop \
  && ln -s /home/imlzw/start /usr/local/bin/start_fdfs \ 
  && ln -s /home/imlzw/stop /usr/local/bin/stop_fdfs

EXPOSE 23000 11411 80

CMD ["bash","start_fdfs"] 

