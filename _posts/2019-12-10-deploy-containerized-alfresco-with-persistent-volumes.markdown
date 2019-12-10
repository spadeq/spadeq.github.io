---
layout: post
title: 具有持久卷的 Alfresco 容器化部署及 AD 集成
date: 2019-12-10 14:46:00
categories:
  - Development
tags:
  - Alfresco
  - Docker
  - Container
---

本文介绍如何容器化部署 Alfresco 社区版，并且使用持久卷保存数据，同时介绍如何在容器化部署中配置 Windows 域的集成。

## 准备 docker-compose 文件

在已经安装好 Docker 和 docker-compose 的服务器上，执行：

```bash
git clone https://github.com/Alfresco/acs-community-deployment.git
cd acs-community-deployment
```

该目录下就有一个 `docker-compose.yml` 文件，本文的所有改动都基于这个文件。

## 加入持久化卷

首先在该文件末尾增加卷的申请。

```yaml
volumes:
  alf-repo-data:
    external: false
  postgres-data:
    external: false
  solr-data:
    external: false
```

注意，如果想自行通过 `docker volume create` 命令预先创建卷，则应该将上述 `external:` 改为 `true`。

然后分别对应找到以下三个容器的定义，加入卷的属性（其他行略去）：

```yaml
services:
  alfresco:
    volumes:
      - alf-repo-data:/usr/local/tomcat/alf_data

  postgres:
    volumes:
      - postgres-data:/var/lib/postgresql/data

  solr6:
    volumes:
      - solr-data:/opt/alfresco-search-services/data
```

如果不希望使用 docker volume，要用本地目录或者是集成 k8s 的 pv，可参考上述路径。

## AD 域认证集成

此部分配置应修改 alfresco 容器的环境变量，加入 Java 参数（其他行略去）：

```yaml
services:
  alfresco:
    environment:
      JAVA_OPTS: "
        -Dauthentication.chain=alfinst:alfrescoNtlm,ldap1:ldap-ad
        -Dntlm.authentication.sso.enabled=false
        -Dldap.authentication.allowGuestLogin=false 
        -Dldap.authentication.userNameFormat=%s@spadeq.com
        -Dldap.authentication.java.naming.provider.url=ldap://spadeq.com:389
        -Dldap.authentication.defaultAdministratorUserNames=Administrator
        -Dldap.synchronization.java.naming.security.principal=bind@spadeq.com
        -Dldap.synchronization.java.naming.security.credentials=passw0rd
        -Dldap.synchronization.groupSearchBase=\"ou=dev,dc=spadeq,dc=com\"
        -Dldap.synchronization.userSearchBase=\"ou=dev,dc=spadeq,dc=com\"
        -Dldap.synchronization.userIdAttributeName=sAMAccountName
      "
```

注意修改第 8、9 行的域名、第 10 行的管理员 id、第 11、12 行的只读用户名和密码、第 13、14 行的基础搜索 OU。

完成之后用 `docker-compose up -d` 命令更新服务。