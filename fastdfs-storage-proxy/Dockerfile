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
	PROXY_PATH=/home/proxy \
    LUAJIT_VERSION=2.0.4 \
    OPENRESTY_VERSION=1.11.2.2 \
	UPSTREAM_CHECK_VERSION=0.3.0 \
    NGX_CACHE_PURGE_VERSION=2.3
	
#创建必要的目录
RUN mkdir -p ${PROXY_PATH}/download

#下载
#RUN wget "http://luajit.org/download/LuaJIT-${LUAJIT_VERSION}.tar.gz" -P ${PROXY_PATH}/download/luajit 
RUN wget "https://openresty.org/download/openresty-${OPENRESTY_VERSION}.tar.gz" -P ${PROXY_PATH}/download/openresty 
RUN wget "http://labs.frickle.com/files/ngx_cache_purge-${NGX_CACHE_PURGE_VERSION}.tar.gz" -P ${PROXY_PATH}/download/nginx 
RUN wget "https://github.com/yaoweibin/nginx_upstream_check_module/archive/v${UPSTREAM_CHECK_VERSION}.tar.gz" -P ${PROXY_PATH}/download/nginx 
RUN tar zxvf ${PROXY_PATH}/download/openresty/openresty-${OPENRESTY_VERSION}.tar.gz -C ${PROXY_PATH}/download/openresty  \
# tar zxvf ${PROXY_PATH}/download/luajit/LuaJIT-${LUAJIT_VERSION}.tar.gz -C ${PROXY_PATH}/download/luajit  \
 && tar zxvf ${PROXY_PATH}/download/nginx/ngx_cache_purge-${NGX_CACHE_PURGE_VERSION}.tar.gz -C ${PROXY_PATH}/download/nginx  \
 && tar zxvf ${PROXY_PATH}/download/nginx/v${UPSTREAM_CHECK_VERSION}.tar.gz -C ${PROXY_PATH}/download/nginx  

#安装LuaJIT 
#WORKDIR ${PROXY_PATH}/download/luajit/LuaJIT-${LUAJIT_VERSION}
#RUN make && make install

#安装openresty
WORKDIR ${PROXY_PATH}/download/openresty/openresty-${OPENRESTY_VERSION}
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
COPY nginx_config/nginx.conf /usr/local/openresty/nginx/conf/nginx.conf
COPY ["start","stop","${PROXY_PATH}/"]
RUN chmod +x ${PROXY_PATH}/start \
  && chmod +x ${PROXY_PATH}/stop \
  && ln -s ${PROXY_PATH}/start /usr/local/bin/start_proxy \ 
  && ln -s ${PROXY_PATH}/stop /usr/local/bin/stop_proxy

CMD ["bash","start_proxy"] 