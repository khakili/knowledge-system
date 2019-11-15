# RocketMQ的安装
## 所需环境
- JDK 1.8+
- Maven 3.2.X
- Git（非必须）


## RocketMQ安装步骤

> 环境请自行配置

- 下载源码: `wget http://mirrors.tuna.tsinghua.edu.cn/apache/rocketmq/4.5.2/rocketmq-all-4.5.2-source-release.zip`
- 解压缩： `unzip rocketmq-all-4.5.2-source-release.zip`
- 编译源码: `mvn -Prelease-all -DskipTests clean package`

## 单机启动
- 进入源码编译好后的目录（rocketmq-all-4.5.2-source-release/distribution/target/rocketmq-4.5.2/rocketmq-4.5.2/bin）

