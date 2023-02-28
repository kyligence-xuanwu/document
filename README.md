云原生数据底座
============
### 云原生数据底座是什么？
继 Hadoop 之后，云原生的 Kubernetes 技术上将形成新一代大数据底座，以 Spark 为统一的计算引擎，以湖仓为统一的存储核心，大幅简化数据栈复杂度，上建低代码、低门槛的数据应用，是未来的大势所趋。
经过以金融生产场景的初步测评，云原生数据底座可以让：
- ***整体 IT 资源利用率提升 20-50%***，受益于底层的计算统一和存储统一，资源能在企业更大范围内灵活调配
- ***整体计算性能提升约 20%***，受益于存算分离架构，计算任务能更有弹性的分布到更多计算节点高速完成
- ***系统运维成本大幅降低***，受益于理论上无限的扩容能力，和整合多系统到统一底座的能力
- 实现***国产化、自主可控***的技术换代
### Kyligence 云原生数据底座能做什么？
Kyligence 云原生数据底座简称 KYLIGENCE XUANWU，它是一个开源套件，旨在提供一个快速体验和验证新技术的平台，推动国产化云原生技术的交流和发展。
- ***快速安装***，一行命令完成 Kubernetes 上的安装，30 分钟开始体验云原生的数据能力
- ***一键运行 TPC-H 基准测试***，云原生的 SparkSQL 相比 Hadoop 提速约 20%
- ***一键体验弹性伸缩***，快速响应业务变化，精准应对潮汐峰谷
- ***在低代码平台上体验 AI 机器学习***，从数据准备、模型训练、到模型发布的全链路
### 后续展望
- 下一代 Native Spark Engine 核心升级，性能全面提升约 2.5 倍，预计能整体节省 IT 成本约 40%！
- 自建 Serverless 计算集群，实现企业内的无限扩容 和 pay-as-you-go

![alt text](/asset/kyligence_xuanwu.png "Kyligence 云原生数据底座")

Quick Start
============
### 1. 快速部署
#### 1.1 部署 Xuanwu Manager 步骤
Xuanwu Manager 提供对资源、作业和服务的管理与观测能力，它是数据服务运行的基础。
##### 1.1.1 获取安装脚本
```Bash
wget --no-check-certificate https://s3.cn-north-1.amazonaws.com.cn/public.kyligence.io/xuanwu/release/alpha20221230/install_opensource.sh
```
##### 1.1.2 执行安装脚本
```Bash
bash install_opensource.sh
```
正确部署后，使用浏览器即可访问 Xuanwu Manager 各项功能，目前支持两种身份登录：
- 租户管理员：xuanwu/ADMIN/Xuanwu@319（默认）
- 平台管理员：ADMIN/KYLIN（默认）


#### 1.2 部署数据服务步骤
##### 1.2.1 获取安装脚本
```Bash
# 获取安装包并解压
wget --no-check-certificate https://s3.cn-north-1.amazonaws.com.cn/public.kyligence.io/xuanwu-services/release/alpha20221230_opensource.tar.gz
```
##### 1.2.2 配置对象存储
请修改 ```xuanwu-1.0-alpha-part-1/values.yaml```
```YAML
# 对象存储
storage:
  type: obs # 华为填写 obs，阿里填写 oss
  bucket: https://mybucket.obs.cn-east-3.myhuaweicloud.com # bucket 地址（obs 为例）
  accessKey: yourAccessKey
  secretKey: yourSecretKey
```
##### 1.2.3 配置 RSS
为提高存算分离架构下计算引擎执行稳定性，推荐[部署 RSS 服务](./manual/rss_installation_guide.md "部署 RSS 服务")，请将 RSS Master 地址填入文件 ```xuanwu-1.0-alpha-part-1/values.yaml``` ，示例如下：
```YAML
kyuubi:
  spark:
    rss: # RSS 地址
      - addresses:
        - ip: 10.1.2.146
        ports:
        - port: 9097
          protocol: TCP
      - addresses:
        - ip: 10.1.2.147
        ports:
        - port: 9097
          protocol: TCP
      - addresses:
        - ip: 10.1.2.148
        ports:
        - port: 9097
          protocol: TCP
```
##### 1.2.4 执行安装脚本
在安装包目录下找到 ```install.sh``` 并执行

### 2. 快速使用
#### 2.1 一行命令，生成 TPC-DS 数据集
进入安装包根目录，执行
```Bash
kubectl apply -f ./tpcds-setup.yaml
```
查看数据集生成进度，执行 ```kubectl get job tpcds-setup -n xuanwu``` ，当 COMPLETIONS 显示 1/1 表示完成
```
NAME        COMPLETIONS   DURATION   AGE
tpcds-setup   1/1           33m        48m
```
#### 2.2 一行命令，运行 TPC-DS 基准测试
进入安装包根目录，执行
```Bash
kubectl apply -f ./tpcds-run.yaml
```
查看数据集生成进度，执行 ```kubectl get job tpcds-run -n xuanwu``` ，当 COMPLETIONS 显示 1/1 表示完成
```
NAME        COMPLETIONS   DURATION   AGE
tpcds-run   1/1           33m        48m
```
查看测试结果，执行 ```kubectl logs job.batch/tpcds-run -n xuanwu``` 

### 3. 部署 Byzer
为在低代码平台上体验 AI 机器学习，从数据准备、模型训练、到模型发布的全链路，通过 Xuanwu Manager 的 Web GUI 可以快速[部署 Byzer](./manual/byzer_installation_guide.md "部署 Byzer") 


***

### 欢迎加入讨论组

![alt text](/asset/xuanwu_group_qrcode.png "玄武（XUANWU）讨论组")


