# fastdfs服务集合
fastdfs的服务集合，包含tracker与storage服务，集成阿里云的yum.repo,使国内安装速度更加快速！

## 使用简介
当前docker在docker for windows 上运行正常。
根据docker-compose.yml中的编排，可直接运行fastdfs中的服务
- 首先通过命令docker build分别构建各服务镜像
- 在fastdf根目录执行命令：docker-compose up

## 代码拉取方式
```sh
git clone <url>
git submodule init
git submodule update
```
