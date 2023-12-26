---
title: 2023.3.6 - 2023.3.10
date: 2023-03-11 09:46:39
password: 1062597351
tags:
---

> - 工作相关：Upstream data extracts (sftp 配置)
> - 博客更新：[Java Language](../../../../2022/05/08/java-language/)

近期做的一个需求是通过 SFTP 获取上游数据，分为 SSH 连接配置、数据的获取、解析及落库这几步。

### SSH 连接配置

SFTP (SSH File Transfer Protocol) 是 SSH 文件传输协议，或者说安全文件传输协议 (Secure File Transfer Protocol)，使用 SSH 协议进行身份验证并建立安全连接，通过安全的连接来在两终端之间传输文件，可以遍历本地和远程系统上的文件系统。

需要保证用户只能使用sftp命令，不能ssh到机器进行操作，相关接配置步骤如下（[参考](https://linuxstory.org/how-to-use-sftp-to-securely-transfer-files-with-a-remote-server/)）：

1）由本地客户端（或者远程服务器）生成密钥对，将私钥放置于本地，公钥安装到远程，本地即可利用私钥完成认证并登录到远程

```bash
ssh-keygen -t rsa -C "xxx" -f ~/.ssh/mykeys/local_key
# -t rsa 指定要创建的秘钥类型为 RSA
# -C "xxx" 指定追加到公钥文件末尾以便于识别的注释（通常以电子邮件地址用作注释），不指定则默认为 user_name@local_name
# -f .ssh/mykeys/local_key 指定私钥文件的文件名，同名.pub公钥文件会在相同目录下生成（该目录必须存在），不指定则默认为 ~/.ssh/id_rsa
```

2）如果是本地生成的密钥对，则需将公钥安装到远程目标服务器（如果是远程生成的密钥对，则需下载私钥到本地）

```bash
# 查看本地生成的公钥
cat ~/.ssh/mykeys/local_key.pub

# 将公钥拷贝到远程
ssh-copy-id -i ~/.ssh/mykeys/local_key.pub login_name@remote_address
# -i 指定公钥文件，不指定则默认为 ~/.ssh/id_rsa.pub
```

3）将私钥上传到本地服务器，由于项目是运行在 ECS 上的，每次部署都会令上一次的部署的所有配置信息失效，所以需要用到 K8S 的 Secret 对象来保存私钥信息，因为之前不知道 K8S 的这个功能，导致这里卡了很久，碰巧问到有了经验的 devops 一下就解决了，私钥 JSON 配置如下

```JSON
"objects": [
    {
        "objects": [
            {
                "spec": {
                    "contains": [
                        {
                            "command": [
                                "sh",
                                "-c",
                                ".../run_autosys_jobs.sh \"${CMD_ARG1}\" \"${CMD_ARG2}\""
                            ],
                            "volumeMounts": [
                                {
                                    "name": "secret-volume",
                                    "mountPath": "/etc/secret-volume",
                                    "readOnly": true
                                }
                            ]
                        }
                    ],
                    "volumes": [
                        {
                            "name": "secret-volume",
                            "secret": {
                                "secretName": "${SFTP_SECRET}"
                            }
                        }
                    ]
                }
            }
        ]
    }
],
"parameters": [
    {
        "name": "SFTP_SECRET",
        "required": false
    }
]
```

4）测试是否能建立 SSH 连接并打开 SFTP 会话，成功后提示符会变为 SFTP 提示符

```bash
sftp -K ~/.ssh/mykeys/local_key login_name@remote_address
# -K 指定私钥文件，不指定则默认为 ~/.ssh/id_rsa
```

### 写在最后

- 跨组沟通最大的障碍是我自己都搞不清楚自己有哪些问题需要沟通，问题不明确是由于相关背景知识贫瘠（首先是技术知识，其次是业务知识以及对公司内部的各种操作网站不熟悉），技术和业务知识很影响自己对需求的理解，无法及时做出准确有效的判断
- 有个贪心且低效的习惯是，在接触一个新东西的时候，整体和细节全都想要，有先后但没有侧重，总是希望把整体和细节都输入完毕之后再去做输出类的工作，从短时间输出上看，效率比别人低很多，虽然我个人更偏向取长期效率，但短期效率低也是个问题
- 明明是不涉及技术难度的事情却做的很慢，这是因为自己接触过的东西太少，但凡来个事对我来说都是新事物，与此同时接收新事物的速度又不够快
