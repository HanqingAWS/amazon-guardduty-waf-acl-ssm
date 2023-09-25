# 如何通过GuardDuty和Systems+Manager自动化地进行恶意文件处置
自动化的威胁检测和响应，既可以减轻企业安全运营人员的运维工作投入、缩短响应时间，又可以满足网络安全等级保护MLPS和PCI DSS等安全框架的相关要求。在亚马逊云科技的云采用框架安全专业指导文件中，也明确提出了安全自动响应是一个关键的指导原则。

您可以通过使用亚马逊云科技的各种产品，构建您的云环境安全威胁的自动检测和响应的解决方案。例如，通过配置 事件Event来触发Lambda功能来自动响应GuardDuty检测到的可疑或恶意的行为。根据不同检测结果的Event类型，您可以配置不同的自动化响应动作，包括修改网络访问控制规则、终止EC2实例或撤销IAM 安全凭据等。

在本博客中，我们介绍如何使用亚马逊云科技的Systems Manager产品，自动化地响应（删除）被GuardDuty检测发现的恶意文件或病毒文件，以满足企业安全运营和合规的技术要求。

关于GuardDuty检测结果的自动响应的解决方案，亚马逊云科技已经有了一个对恶意IP地址自动拦截的方案，通过在VPC NACL和WAF ACL中自动添加规则的方式来实现，具体参考如下链接。本方案是针对GuardDuty检测出的恶意文件的自动响应的方案，通过SSM执行shell命令删除文件的方式来实现。为了能让客户同时使用这两个方案，本方案的代码（包括Cloud Formation和Lambda代码）都是基于前一个方案的代码框架来实现；客户只需要在Event Bridge中调整所需要自动响应的event类型，即可实现不同方案或两个方案同时使用的目的。

# 使用到的核心产品介绍
* 亚马逊云科技的GuardDuty产品是一项威胁检测服务，它持续监控您的 AWS 账户和工作负载的恶意活动，并提供详细的安全检测结果。Malware Protection恶意文件检测功能是GuardDuty产品下一项新发布的功能，帮助您检测EC2或容器实例上的恶意文件。这种检测活动不需要在EC2上部署Agent，通过扫描EBS的方式来实现。能检测的恶意文件主要包括trojans木马、worms蠕虫、crypto miners 加密货币挖矿程序、rootkits、bots僵尸程序等。
* GuardDuty产生findings检测结果，这些检测结果说明了潜在的安全问题，包括受影响的EC2或容器负载、或者云环境的安全凭据。所有的检测结果都应该被尽快出来，比如调查受影响的EC2实例中的恶意文件、并手动或自动地删除这些恶意文件。
* 亚马逊云科技的Systems Manager是一个端到端的云资源管理产品，包括亚马逊云科技的云资源、也包括混合云或多云环境的云资源。Run Command是Systems Manager产品的一个重要功能模块，使用此模块，您可以远程、安全地管理EC2实例的配置；包括自动地执行系统管理任务和执行一次性的配置变更。

# 方案整体
在本博客中，我们将向您暂时如何通过亚马逊云科技的GuardDuty和Systems Manager产品来自动响应GuardDuty实时检测的恶意文件。当GuardDuty检测到恶意文件（Malware）的存在时，会实时自动触发Lambda函数，进而通过Systems Manager在EC2上删除被发现的恶意文件；再删除文件之前，会对文件及其相关信息在S3和DDB中进行备份，便于在需要恢复被删文件的时候使用。
* 该方案的整体架构图如下所示：
![image](https://github.com/HanqingAWS/amazon-guardduty-waf-acl-ssm/blob/main/amazon-guardduty-waf-acl-ssm.jpg)

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.
