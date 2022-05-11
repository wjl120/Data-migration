# 使用DataSync 同步GCS数据到Amazon S3

# 背景

客户需要将GCP的Cloud Storage 的存量和增量数据和同步到Amazon S3做数据分析,但没有一个好用的数据迁移工具,rclone需要写脚本和定期人工检查，工作量大，且无法保证质量，因此想测试一下Datasync。
AWS DataSync 能够从与 S3 兼容的对象存储传输文件。 Google Cloud (GCP) 使用其互操作性 XML API 支持其 Google Cloud Storage (GCS) 产品的 S3 API。 本文档中的过程将指导您完成将数据从 GCS 存储桶传输到 AWS 上的 S3 存储桶的过程。

# 需求

* 全量数据的迁移;
* 每天在规定时间内保证增量数据的迁移完成；
* 可检查和监控任务的成功或失败；
* 保证数据的正确性：增量数据有可能是新文件 ，原旧文件可能已经删除，也有可能是同名文件 ，但是会更新数据；这两种情况都需要处理；
* 按时完成数据的同步；

# 架构

[Image: image.png]AWS DataSync 使用 S3 互操作性 XML API 连接到您的 GCS 存储桶。 它使用 HMAC AccessKey 和 SecretKey 进行身份验证，并同步任务中定义的数据。 在 AWS 区域中，AWS DataSync 代理部署为 Amazon EC2 实例，代理使用 VPC Endpoint 连接到 AWS DataSync 服务以在同一子网内移动数据，从而避免任何跨 AZ 或跨区域费用 在 AWS 内。 代理从 GCS 读取数据，将其发送到 AWS DataSync 服务，然后将数据推送到 S3 存储桶。

## DataSync的组件介绍:



* **代理（agent）**– 用于从自我管理位置读取数据或将数据写入到自我管理位置的虚拟机(VM)。 在同一账户中的 AWS 存储服务之间传输时不需要代理。

* **位置(Location)**– 数据传输中使用的任何源或目标位置（例如，Amazon S3、Amazon EFS、Amazon FSx for Windows File Server、NFS、SMB 或自我管理的对象存储）。

* **任务(Task)**– 由源位置和目标位置以及定义数据传输方式的配置组成。 任务总是将数据从源传输到目标。 配置可以包括任务计划、带宽限制等选项。任务是数据传输的完整定义。

* **任务执行(Task Execution)**– 任务的单独运行，包括开始时间、结束时间、写入的字节数和状态等信息。

[Image: image.png]
# 前提条件

* AWS CLI  
* Create HMAC keys from a GCP account that has access to the GCS bucket to transfer data from. You will  use these credentials to configure an Object Storage location for DataSync.



# 过程

## GCP

1. GCP中创建Service account

[Image: image.png]
1. 为CLoud Storage创建服务账号的密钥


[Image: image.png][Image: image.png]选择之前创建的Service account:
[Image: image.png]
1. 注意保留生成的密钥：

访问密钥：GOOG1EC3BLJN4DC3KRXDW7B2TMLK23K3T2NZBTSC6QZAXWGBZNOVOL75I
密钥：KoWx1l+oFQ2XCvWUK47fEisCRumqhW2sbB/
[Image: image.png]
## 

1. 创建VPC endpoint:


[Image: image.png][Image: image.png]
[Image: image.png]
[Image: image.png]注意，安全组的端口要记得对互联网开放80和22，同时对安全组自己要开放：
[Image: image.png][Image: image.png]
1. 安装agent所需要的EC2:

在AWS的EC2上或具备aws cli执行的环境中，执行如下命令（注意替换红字）：

aws ssm get-parameter —name /aws/service/datasync/ami —region $region

得到如下内容，重点关注ami-id：

```
aws ssm get-parameter --name /aws/service/datasync/ami --region us-east-1                              

{
    "Parameter": {
        "Name": "/aws/service/datasync/ami",
        "Type": "String",
        "Value": "ami-id",
        "Version": 6,
        "LastModifiedDate": 1569946277.996,
        "ARN": "arn:aws:ssm:us-east-1::parameter/aws/service/datasync/ami"
    }
}
```

替换`source-file-system-region为实际的region代码，并`用上面的实际红字来替换下面命令中的和`ami-id`：

```
https://console.aws.amazon.com/ec2/v2/home?region=source-file-system-region#LaunchInstanceWizard:ami=ami-id
```

[Image: image.png]
记得不要选小于M5.2xlarge的机型。
[Image: image.png][Image: image]注意安全组的选择：
要22和80端口可以让公网访问，同时endpoint 和你的ec2的安全组最好配置为一个，这样减少配置，里面把本安全组自己配置进去。
[Image: image.png]
在DataSync中创建agent：
[Image: image.png][Image: image.png][Image: image.png]
激活agent, 得到下面这样的key,然后创建agent:
[Image: image.png]



1. 创建源location：


[Image: image.png]
[Image: image.png]创建destination location:y
[Image: image.png]


1. 创建task

[Image: image.png]配置源location:
[Image: image.png]配置目标location:
[Image: image.png]
1. 设置其它配置

[Image: image.png][Image: image.png][Image: image.png]

1. 最后创建并启动任务

[Image: image.png]


1. 监控任务执行

[Image: image.png]
1. 结果

[Image: image.png]
## 结论

Datasync可以满足客户对于同步存量数据和增量数据从GCS到S3的功能，并有如下优势：

* 数据加密和验证 

您的所有数据在传输过程中都使用传输层安全性 (TLS) 进行加密。 DataSync 支持对 S3 存储桶使用默认加密、静态数据的 Amazon EFS 文件系统加密以及静态和传输中数据的 Amazon FSx for Windows File Server 加密。 AWS DataSync 可确保您的数据完好无损地到达。对于每次传输，服务都会在传输中和静止时执行完整性检查。这些检查确保写入目标的数据与从源读取的数据相匹配，从而验证一致性。 

* 数据传输调度 

AWS DataSync 带有内置的调度机制，使您能够定期执行数据传输任务以检测更改从源存储系统复制到目标。您可以使用 AWS DataSync 控制台或 AWS 命令行界面 (CLI) 安排您的任务，而无需编写脚本来管理重复传输。任务计划自动按照您配置的计划运行任务，控制台中直接提供每小时、每天或每周选项。 

* 使用 Amazon CloudWatch 和 AWS CloudTrail 进行监控和审计 

使用 Amazon CloudWatch，您可以监控当前正在进行的任何 DataSync 传输的状态，并检查以前数据传输的历史记录。使用 CloudWatch 指标，您可以查看已复制的文件数量和数据量。您可以查阅 CloudWatch Logs 以了解有关在给定时间传输的单个文件的信息，以及 DataSync 执行的完整性验证的结果。这简化了监控、报告和故障排除，并使您能够及时向利益相关者提供更新。此外，可以在您的传输任务完成时触发 CloudWatch Events，从而实现依赖工作流的自动化。出于审计目的，您可以查阅 AWS CloudTrail，它记录了 DataSync 执行的所有操作。 

* 即用即付定价 

您只需为服务复制的数据付费，按每 GB 统一的费率付费，无需软件许可、合同、维护费用、开发周期或所需硬件。与手动构建、操作和优化您自己的高性能脚本传输相比，这提供了更低的总拥有成本。它还提供比购买和运行商业传输工具更低的总成本。 

* 自动基础架构管理 

AWS DataSync 消除了您在编写、优化和管理自己的复制脚本或部署和调整重量级商业传输工具时面临的许多基础设施和管理挑战。 DataSync 带有内置的监控和重试机制，以及对用于传输数据的网络带宽部分的精细控制。
