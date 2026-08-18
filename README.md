# 1 项目简介
## 1.1 声明
本项目开发和运维团队为**中国工程物理研究院成都科学技术发展中心第四研究室**，是基于[**OpenSCOW**](https://github.com/PKUHPC/OpenSCOW)和ABHPC单层RDMA网络无盘超算集群开发的用户与管理门户系统，**同时也适配于采用Slurm调度系统的集群**，旨在将该框架广泛应用于科研、教育和工程行业。

按照木兰宽松协议v2要求，本项目更名为**ASCOW**且不开源，以保障**OpenSCOW**的商标权，但发布的公共镜像允许任意用户下载并免费使用，本项目的基础配置文件和**OpenSCOW**完全兼容(可参考[**OpenSCOW的安装与配置**](https://pkuhpc.github.io/OpenSCOW/docs/deploy))，增加的部分字段可参考更新日志说明。

本系统完全沿用OpenSCOW框架，感谢**北京大学OpenSCOW开发团队**的无私贡献，向他们致敬！

> 本系统推荐使用[**ASCOW-slurm适配器**](https://github.com/abhpc/abhpc-scow-slurm-adapter)，可避免交互式作业中文乱码和GPU资源分配等问题，还可实现基于`Environment Module`的模块调用统计。

## 1.2 技术合作和研讨
本项目欢迎任意形式的技术合作和研讨，可联系四室主任刘晓毅(邮件：[xyliu#mechx.ac.cn](mailto:xyliu@mechx.ac.cn))和陈洁高级工程师(邮件：[jchen#mechx.ac.cn](mailto:jchen@mechx.ac.cn))。

# 2 版本说明和使用方法

## 2.1 历史版本说明
**20250901版本起，ND和SAND功能均在通用版本中实现，因此这两个版本不再更新，后续只更新通用版本。**

> latest-ND：禁用下载版本(已合并到通用版本)

> latest-SAND: 单点登录且禁用下载版本(已合并到通用版本)

## 2.2 使用方法
首先拉取镜像：
```bash
docker pull xyliucd/ascow:latest
```
使用`scow-cli`生成docker-compose.yml文件：
```bash
./cli generate
```
替换为`ascow`镜像:
```bash
sed -i s "@mirrors.pku.edu.cn/pkuhpc-icode/scow:v1.6.4@xyliucd/ascow:latest@g" docker-compose.yml
```
启动容器：
```bash
docker compose up -d
```

# 3 更新日志(按日期倒序排列)
## 20260818
- [功能优化]显示上传进度与速度
- [功能优化]支持作业计费精度热生效

## 20260816
- [bug修复]http单点登录强制登录提示。
- [功能优化]批量创建用户
- [功能优化]机时价格允许在线设置小数位数（0-4）
- [功能优化]创建用户时重名识别
- [功能优化]信息提醒侧边栏一键已读
- [功能优化]支持在线配置单点登录

## 20260731
- [bug修复]http单点登录强制登录失败。
- [bug修复]http无法信任设备。
- [bug修复]单点登录被踢没有提示

## 20260727
- [bug修复]操作日志中的ip会被解析为容器ip

## 20260723
- [功能优化]冻结费用
- [Bug修复]计量方式删详情查看
- [功能优化]提醒通知加导出excel功能
- [功能优化]系统信息批量删除
- [布局优化]修改操作列图标
- [功能优化]用户名限制只能小写

## 20260717
- [功能优化]增加CPU和GPU分项计费策略
- [功能优化]增加平台安全提示
- [安全]禁止通过网页直接访问文件和文件上传
- [安全]设置文件上传后缀黑名单
- [安全]按账号限制登录失败（含非法账号）
- [安全]前端安全-隐藏敏感运行配置
- [安全]允许用户信任设备（信任设备不再要求2FA认证），用户查看信任设备列表并删除信任设备。
- [安全]登录安全增强-2FA+图片验证码+登录限制

  **注意**：启用2FA时，要修改docker-compose.yml，增加mis-server:environment:MIS_CONFIG_ENCRYPT_KEY=请替换为固定的高强度随机字符串
- [安全]运行httos协议

  **注意**：需要配置docker-compose.yml
  ```
  gateway:
    restart: always
    environment:
    # 添加
      - HTTPS_ENABLED=true
      - HTTPS_PORT=443
      - HTTPS_CERT_PATH=/etc/scow/ssl/fullchain.pem
      - HTTPS_KEY_PATH=/etc/scow/ssl/privkey.pem
      - HTTPS_REDIRECT_HTTP_TO_HTTPS=true
      - HTTPS_HSTS_ENABLED=false
      - HTTPS_HSTS_MAX_AGE=31536000
      - ALLOWED_SERVER_NAME=mx.yinhe596.cn
      - DEFAULT_SERVER_BLOCK=server {\n    listen 80 default_server;\n    return 444;\n  }\n\n  server {\n    listen 443 ssl default_server;\n    ssl_certificate /etc/scow/ssl/fullchain.pem;\n    ssl_certificate_key /etc/scow/ssl/privkey.pem;\n    return 444;\n  }
    ports:
    # 修改端口号为443
      - '88:443'
    volumes:
    # 添加ssl证书路径
      - ./ssl:/etc/scow/ssl:ro
    
  auth:
    environment:
      - PUBLIC_ORIGIN=https://mx.yinhe596.cn:20388 # 实际访问地址

  portal-web:
    environment:
      - PROTOCOL=https

  mis-web:
    environment:
      - PROTOCOL=https
  ```

  然后重启
  docker compose up -d --force-recreate gateway auth portal-web mis-web

- [Bug修复]修复信息中心查看更多跳转(因隐藏配置出现的bug)
- [Bug修复]修复管理员修改用户信息修改被相同邮箱阻断
- [Bug修复]邮箱验证通过后旧邮箱才失效
- [功能优化]管理员可查看邮箱验证状态
- [功能优化]增加站内信和邮件通知统计曲线
- [功能优化]增加找回密码功能

## 20260622
- [功能优化]资源使用监控，允许管理员查看作业负载
- [功能优化]新建应用时添加临时邮箱接收提醒
- [功能优化]在线配置发送邮箱
- [Bug修复]未设置邮箱时，发送验证邮箱失败提醒

## 20260422
- [功能优化]新增站内信系统

## 20260419 
- [Bug修复]删除用户以后同步封锁状态,已经删除的用户会卡死封锁同步
- [功能优化]限制作业名称为：_-数字、大小写字母、中文
- [Bug修复]删除账户和用户时同时删除数据库里账户用户关系

## 20260323
- **[重要]** 增加用户、账户和租户删除功能。**需要更新adapter**。
- [Bug修复]设置目录白名单后，作业默认跳转到根目录。
- [功能优化]大于3%用量用户太多时，显示溢出

## 20260126
- **[重要]** 修复OpenSCOW修改邮箱渗透漏洞：任意登录用户可绕过权限修改其他用户邮箱。
  
## 20260122
- 修复账号管理员查看计算资源统计部分接口报错
  
## 20260121
- 仪表盘显示CPU、GPU和节点数情况：
  
<img width="1200" alt="image" src="https://github.com/user-attachments/assets/5bf0f7a1-7d0f-4e99-a857-cb74e50f4ba6" />

## 20260115
- [Pro]通过设置用户目录白名单，修复任意文件读取漏洞，避免用户选择任意文件点击下载，拼接`../../../etc/passwd`，便能读取到`passwd`。
  
<img width="800" alt="image" src="https://github.com/user-attachments/assets/3c9c441c-6af0-47af-a617-cc3a2d388c1e" />

## 20260110
- 增加版本号；
- 修改系统默认logo。
  
## 20260109
- [Pro]文件管理器地址栏新增`最近的作业目录`选项（**需要同步更新adapter**）：
  
  <img width="1200" alt="image" src="https://github.com/user-attachments/assets/1ce370a3-1184-4e31-a81e-465a0a6d60bb" />
  
## 20251230
- 修改平台和语言切换的显示模式：
  
  <img width="200" alt="image" src="https://github.com/user-attachments/assets/378676cf-80dc-4216-9b60-67e457677e20" />

## 20251212
- 在`应用`的`yml`文件(`config/apps/*.yml`)的`visible`字段增加`ban_accounts`和`ban_users`字段：
```yml
visible:
  public: no                   # 如果public值为yes，则所有用户可见
  allow_accounts: caep,mechx   # 如果public为no，allow_accounts中账户下的所有用户可见APP
  allow_users: user1,uesr2     # 如果public为no，allow_users中的用户可见APP
  ban_accounts: xxx1,xxx2      # 如果public为yes，ban_accounts中账户下的所有用户不可见APP
  ban_users: xxx1,xxx2         # 如果public为yes，ban_users中的用户不可见APP
```

## 20251211
- 应用：增加`最长运行时间`单位`小时`和`天`。

  <img width="300" alt="image" src="https://github.com/user-attachments/assets/21974b1f-31da-439a-9682-6f1b9faa8e5a" />

## 20251210
- 修复bug：文件选择器选择中间位置，前面会丢失反斜杠。
## 20251201
- 去掉夜间模式。
- 在`mis.yml`中修正错误提示：
```yml
createUser:
  builtin:
    userIdPattern:
      regex: "^[a-z][a-z0-9_]{2,39}$"
      errorMessage:
        i18n:
          default: "用户名只能包含小写字母、数字和下划线，且必须以小写字母开头"
          zh_cn: "用户名只能包含小写字母、数字和下划线，且必须以小写字母开头"
          en: "Username can only contain lowercase letters, numbers and underscores, and must start with a lowercase letter"
```

## 20251114
- 优化`作业->提交作业`页面布局，新增保存为模板前的命名：

<img width="1200" alt="image" src="https://github.com/user-attachments/assets/a7969c72-52e7-43e7-b5d3-ca0e9104c971" />


## 20251110
- 调整了`作业—>提交作业`页面的布局。
## 20251109
- 优化管理员界面的作业显示列表，给用户名和账户加上中文备注，方便识别用户和单位。

## 20251105
- 在`文件目录选择框`中新增`上传文件`和`新文件`按钮：
  
<img width="400" alt="image" src="https://github.com/user-attachments/assets/c645602e-3dfd-4bc3-b197-8f86472c26d7" />

## 20251104
- 优化`用户列表`显示
## 20251101
- 优化`账户列表`显示
## 20251031
- 增加应用搜索栏。
- 账户备注变为可编辑。
## 20251027
- 增加用户单位信息，需要在`auth.yml`中添加单位信息：
```yml
  ldap:
  attrs:
    uid: uid
    name: cn
    mail: mail
    affiliation: description #新加的
```
- 修正了修改文件时，改变文件权限的Bug
## 20251020
- 修复导入用户管理界面显示姓名为`userID`的问题，正常应显示为LDAP中的中文名。
## 20251011
- 美化界面。
## 20251009
- 优化NGINX的websockets超时参数，避免VNC经常断线的问题。
## 20250928
- 优化图标显示，增加最大作业时长统计。
- 优化筛选时间。

## 20250924
- 修正了不正确显示用户登录IP的Bug。
- 优化了应用同步页面，如下所示：

<img width="800" alt="image" src="https://github.com/user-attachments/assets/853dcc08-db16-46fe-9939-0d90c46c8db2" />

## 20250923
- 优化应用加载页面在APP数量较多时载入缓慢的问题，采用`redis`缓存机制。

## 20250919
- 新增图表统计，可导出数据做详细分析。
  
  <img width="1200" alt="image" src="https://github.com/user-attachments/assets/37d19120-ee8e-486d-b6b6-4267d57beece" />

- 优化`portal-web`的并发度，实现基于`pm2`的`Nginx`负载均衡。

## 20250917
- 优化`portal-web`和`portal-server`的并发度，全部采用`pm2 cluster`来实现高并发度。
## 20250916
- 修改最大导出条目为50000条(原来设计为10000条)
- 应用图标大小默认修改为小
- 优化计算资源统计表显示

## 20250914
- 新增计算资源统计板块，可统计不同module核时情况：
  
<img width="600" alt="image" src="https://github.com/user-attachments/assets/9a8b0f5f-f627-42f2-847c-49869d0f42f2" />

## 20250910
- 限制前后缀为`slm.*`、`*.slm`、`*.sh`、`*.bash`、`*.slurm`、`slurm.*`六种文件可提交，其他文件不可提交。

## 20250909
- 新增已完成作业应用模块的csv导出功能，以统计不同软件模块的机时占比。

## 20250906
- 保存用户作业调取`Environment Module`记录，便于后期做软件使用分析。如用户主节点未配置`Environment Module`记录器，则记录值为`unknown`。需要配合最新的[**abhpc-scow-slurm-adapter**](https://github.com/abhpc/abhpc-scow-slurm-adapter)使用，参见[效果图](https://github.com/abhpc/abhpc-scow-slurm-adapter/blob/main/image/README/1757250643455.png)：

<img width="1200" src="https://raw.githubusercontent.com/abhpc/abhpc-scow-slurm-adapter/main/image/README/1757250643455.png" alt="Image Description" />

## 20250901
- ND和SAND版本合并到通用版本中来。其中，单点登录在`auth.yml`文件中新增以下字段实现单点登录：
```yml
singleSession: on  # on为开通单点登录，off为关闭单点登录
```
通过在`portal.yml`文件中增加以下字段实现文件下载权限控制：
```yml
# 是否允许文件下载
file:
  download:
    enable: false
    #enableAccounts: account1,account2,account3
    #enableUsers: user1,user2,user3
    # enable为true的情况下，禁止哪些账户或用户下载
    #disableAccounts: account1,account2,account3
    #disableUsers: user1,user2
```


## 20250829
- 忽略`config/apps/*.yml`的错误配置，避免某个交互式应用配置出错时，整个集群报500错误。
- 新增交互式应用的图标大小调整。

## 20250827
- 在交互式应用的`yml`文件(`config/apps/*.yml`)中增加`visible`字段，使交互式应用可设置为公开或部分账户、用户可见。
```yml
visible:
  public: no                   # 如果public值为yes，则所有用户可见
  allow_accounts: caep,mechx   # 如果public为no，allow_accounts中账户下的所有用户可见APP
  allow_users: user1,uesr2     # 如果public为no，allow_users中的用户可见APP
```
## 20250826
- 增加了Portal的pm2并发性能，解决交互式应用在线用户数大于20时系统崩溃的问题。
- 修改文件和文件夹的图标，以方便区分文件夹和文件。
- 文件选择器默认路径设置为用户家目录，而不是根目录/。
- ND版本：禁用文件下载，适用于特殊教学或培训场景。
- SAND版本：增加用户单点登录功能，禁用同一账户在不同电脑上登录，且禁止用户下载文件。
- VNC单点登录和分辨率优化：禁用VNC多点登录，初始分辨率设置为3840x2160以避免部分程序窗口无法放大。
- 交互式应用新增文件和目录选择器，以支持基于后台的非交互式应用，例如：
```yml
attributes:
  - type: file
    name: input
    label: 输入文件
    required: true
    placeholder: "比如：/home/user/fds/test.fds"
```
- 新增目录链接识别，可在交互式应用中直接链接到用户算例目录。
- 优化资源选择，默认CPU和GPU核数独立设置，以方便用户选择资源。
