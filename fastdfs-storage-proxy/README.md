# fastdfs服务集合
fastdfs的服务集合，包含tracker与storage服务，集成阿里云的yum.repo,使安装速度更加快速！

# 使用简介
当前docker在docker for windows 上运行正常。
- 先启动两台fastdfs-tracker（生成ip:172.17.0.2,ip:172.17.0.3的两台tracker）
- 再启动两台fastdfs-storage-dht服务(生成ip:172.17.0.4，ip:172.17.0.5)，storage集成了fastdht实现文件排重功能。
- 测试上传

# 启动命令
```java
# 172.17.0.2
docker run -dit -p 181:80 -p 22122:22122 --name tracker1 imlzw/fastdfs-tracker
# 172.17.0.3
docker run -dit -p 182:80 -p 22123:22122 --name tracker2 imlzw/fastdfs-tracker
# 172.17.0.4
docker run -dit -p 281:80 -p 23000:23000 --name u_group1_storage1 --link tracker1:tracker1 --link tracker2:tracker2 imlzw/fastdfs-storage-dht
# 172.17.0.5
docker run -dit -p 282:80 -p 23001:23000 --name u_group1_storage2 --link tracker1:tracker1 --link tracker2:tracker2 imlzw/fastdfs-storage-dht
# 172.17.0.5
docker run -dit -p 282:80 -p 23001:23000 --name u_group1_storage2 --link tracker1:tracker1 --link tracker2:tracker2 imlzw/fastdfs-storage-dht
```
如果生成ip不是上面的话，运行可能会出错，你需要自行修改所有相关文件夹下的配置文件（/etc/fdfs/*,/etc/fdht/*），手动停止与手动启动服务

# 自启动与手动启动
所有容器都已经集成自启动服务的功能，并且提供启动服务与停止服务的命令：start_fdfs与stop_fdfs
