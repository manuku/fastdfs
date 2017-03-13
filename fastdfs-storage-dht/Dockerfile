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
    LIB_FAST_COMMON_VERSION=1.0.35 \
    FASTDFS_VERSION=5.09 \
    NGINX_VERSION=1.11.7 

#创建必要的目录
RUN mkdir -p ${FASTDFS_PATH}/download \
 && mkdir -p ${FASTDFS_STORAGE_PATH} \
 && mkdir -p ${FASTDFS_PATH}/client \
 && mkdir -p ${FASTDFS_PATH}/mod 

#下载
RUN wget "https://github.com/happyfish100/libfastcommon/archive/V${LIB_FAST_COMMON_VERSION}.tar.gz" -P ${FASTDFS_PATH}/download/libfastcommon
RUN wget "https://github.com/happyfish100/fastdfs/archive/V${FASTDFS_VERSION}.tar.gz" -P ${FASTDFS_PATH}/download/fastdfs
RUN wget https://github.com/happyfish100/fastdfs-nginx-module/archive/master.zip -P ${FASTDFS_PATH}/download/nginx_module 
RUN wget "http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz" -P ${FASTDFS_PATH}/download/nginx 
RUN tar zxvf ${FASTDFS_PATH}/download/libfastcommon/V${LIB_FAST_COMMON_VERSION}.tar.gz -C ${FASTDFS_PATH}/download/libfastcommon \
 && tar zxvf ${FASTDFS_PATH}/download/fastdfs/V${FASTDFS_VERSION}.tar.gz -C ${FASTDFS_PATH}/download/fastdfs \
 && unzip ${FASTDFS_PATH}/download/nginx_module/master.zip -d ${FASTDFS_PATH}/download/nginx_module \
 && tar zxvf ${FASTDFS_PATH}/download/nginx/nginx-${NGINX_VERSION}.tar.gz -C ${FASTDFS_PATH}/download/nginx 

#安装libfastcommon
WORKDIR ${FASTDFS_PATH}/download/libfastcommon/libfastcommon-${LIB_FAST_COMMON_VERSION}
RUN ["/bin/bash", "-c", "./make.sh && ./make.sh install"]

#安装fastdfs
WORKDIR ${FASTDFS_PATH}/download/fastdfs/fastdfs-${FASTDFS_VERSION}
RUN ["/bin/bash", "-c", "./make.sh && ./make.sh install && cp conf/* /etc/fdfs/"] 

#安装nginx
WORKDIR ${FASTDFS_PATH}/download/nginx/nginx-${NGINX_VERSION}
RUN ./configure --add-module=${FASTDFS_PATH}/download/nginx_module/fastdfs-nginx-module-master/src \
 && make \
 && make install \
 && cp ${FASTDFS_PATH}/download/nginx_module/fastdfs-nginx-module-master/src/mod_fastdfs.conf /etc/fdfs/
 
#添加初始化配置
RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
COPY fdht_config/* /etc/fdht/
COPY fdfs_config/* /etc/fdfs/
COPY nginx_config/nginx.conf /usr/local/nginx/conf/nginx.conf
COPY ["start","stop","/home/imlzw/"]
RUN chmod +x /home/imlzw/start \
  && chmod +x /home/imlzw/stop \
  && ln -s /home/imlzw/start /usr/local/bin/start_fdfs \ 
  && ln -s /home/imlzw/stop /usr/local/bin/stop_fdfs

CMD ["bash","start_fdfs"] 

