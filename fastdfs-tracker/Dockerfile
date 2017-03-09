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
    FASTDFS_TRACKER_PATH=/home/imlzw/fastdfs/tracker \
    LIB_FAST_COMMON_VERSION=1.0.7 \
    FASTDFS_VERSION=5.05 \
    LUAJIT_VERSION=2.0.4 \
    OPENRESTY_VERSION=1.11.2.2 \
	UPSTREAM_CHECK_VERSION=0.3.0 \
    NGX_CACHE_PURGE_VERSION=2.3
	
#创建必要的目录
RUN mkdir -p ${FASTDFS_PATH}/download \
 && mkdir -p ${FASTDFS_TRACKER_PATH} 

#下载
RUN wget "https://github.com/happyfish100/libfastcommon/archive/V${LIB_FAST_COMMON_VERSION}.tar.gz" -P ${FASTDFS_PATH}/download/libfastcommon
RUN wget "https://github.com/happyfish100/fastdfs/archive/V${FASTDFS_VERSION}.tar.gz" -P ${FASTDFS_PATH}/download/fastdfs
#RUN wget "http://luajit.org/download/LuaJIT-${LUAJIT_VERSION}.tar.gz" -P ${FASTDFS_PATH}/download/luajit 
RUN wget "https://openresty.org/download/openresty-${OPENRESTY_VERSION}.tar.gz" -P ${FASTDFS_PATH}/download/openresty 
RUN wget "http://labs.frickle.com/files/ngx_cache_purge-${NGX_CACHE_PURGE_VERSION}.tar.gz" -P ${FASTDFS_PATH}/download/nginx 
RUN wget "https://github.com/yaoweibin/nginx_upstream_check_module/archive/v${UPSTREAM_CHECK_VERSION}.tar.gz" -P ${FASTDFS_PATH}/download/nginx 
RUN tar zxvf ${FASTDFS_PATH}/download/libfastcommon/V${LIB_FAST_COMMON_VERSION}.tar.gz -C ${FASTDFS_PATH}/download/libfastcommon \
 && tar zxvf ${FASTDFS_PATH}/download/fastdfs/V${FASTDFS_VERSION}.tar.gz -C ${FASTDFS_PATH}/download/fastdfs \
# && tar zxvf ${FASTDFS_PATH}/download/luajit/LuaJIT-${LUAJIT_VERSION}.tar.gz -C ${FASTDFS_PATH}/download/luajit  \
 && tar zxvf ${FASTDFS_PATH}/download/openresty/openresty-${OPENRESTY_VERSION}.tar.gz -C ${FASTDFS_PATH}/download/openresty  \
 && tar zxvf ${FASTDFS_PATH}/download/nginx/ngx_cache_purge-${NGX_CACHE_PURGE_VERSION}.tar.gz -C ${FASTDFS_PATH}/download/nginx  \
 && tar zxvf ${FASTDFS_PATH}/download/nginx/v${UPSTREAM_CHECK_VERSION}.tar.gz -C ${FASTDFS_PATH}/download/nginx  

#安装libfastcommon
WORKDIR ${FASTDFS_PATH}/download/libfastcommon/libfastcommon-${LIB_FAST_COMMON_VERSION}
RUN ["/bin/bash", "-c", "./make.sh && ./make.sh install"]

#安装fastdfs
WORKDIR ${FASTDFS_PATH}/download/fastdfs/fastdfs-${FASTDFS_VERSION}
RUN ["/bin/bash", "-c", "./make.sh && ./make.sh install && cp conf/* /etc/fdfs/"] 

#安装LuaJIT 
#WORKDIR ${FASTDFS_PATH}/download/luajit/LuaJIT-${LUAJIT_VERSION}
#RUN make && make install

#安装openresty
WORKDIR ${FASTDFS_PATH}/download/openresty/openresty-${OPENRESTY_VERSION}
RUN ./configure --prefix=/usr/local/openresty \
	--with-debug \
	--with-http_realip_module  \
	--with-pcre  \
	--with-luajit \
	--add-module=../../nginx/ngx_cache_purge-${NGX_CACHE_PURGE_VERSION} \
	--add-module=../../nginx/nginx_upstream_check_module-${UPSTREAM_CHECK_VERSION} -j2 \
 && make -j2 \
 && make install
 
#添加初始化配置
RUN mkdir -p /var/cache/nginx/proxy_cache/tmp 
RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
COPY fdfs_config/* /etc/fdfs/
COPY nginx_config/nginx.conf /usr/local/openresty/nginx/conf/nginx.conf
COPY ["start","stop","/home/imlzw/"]
RUN chmod +x /home/imlzw/start \
  && chmod +x /home/imlzw/stop \
  && ln -s /home/imlzw/start /usr/local/bin/start_fdfs \ 
  && ln -s /home/imlzw/stop /usr/local/bin/stop_fdfs

EXPOSE 22122

CMD ["bash","start_fdfs"] 