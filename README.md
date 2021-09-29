# oVirtInstallDocument
简单概述oVirt在CentOS7.9上面的部署方式


### 虚拟化平台——oVirt安装文档（适用于CentOS 7.9、oVirt 4.3）（待更新）

***

+ 采用“engine+hosts”的计算模式
+ 本文完全基于Enterprise Linux搭建
+ 官方文档：https://ovirt.org/documentation/virtual_machine_management_guide/index.html
+ 有错误请在issues发布 
+ 本文适合稍微有一些Linux操作系统使用经验的人阅读
+ 在以下步骤出现了报错，如果本文档尚未列出，也可以发布issues，然后会将其列入报错列表/注意事项

#### 准备材料

***

+ 已经写入完成的CentOS 7镜像的U盘或者其他移动存储设备
+ 一台配置较低的服务器（用于安装管理端，即oVirt-engine）
  + 最低配置：
    + CPU：双核心x64架构CPU
    + 内存：4GB RAM
    + 硬盘：本地可读写磁盘25GB
    + 网络适配器：带宽1Gbps
+ 一台配置足够高的服务器（用于安装为oVirt-engine的节点，即oVirt-node）
+ 以上2台服务器必须连网，且拥有固定的IP，如没有公网IP则需要在同一网域下
+ 一个域名（因为oVirt >= 4.0的版本官网添加了“全限定域名”，即Fully Qualified Domain Name FQDN），如果在局域网使用无需备案

#### 安装管理端

***

+ 在配置较低的服务器上安装软件源（repo）并关闭SELinux

  ```shell
  yum update
  yum install https://resources.ovirt.org/pub/yum-repo/ovirt-release43.rpm
  setenforce 0
  ```

  注：setenforce 0 为临时关闭SELinux，永久关闭需要将/etc/sysconfig/selinux的SELINUX=enforcing改为SELINUX=disabled，然后重启生效

+ 安装ovirt-engine软件包

  ```shell
  yum install ovirt-engine
  ```

  注意：如果有依赖问题就尝试解决，在卸载冲突的软件包的时候请不要直接使用参数“-y”，一定要看清这个软件包被什么所依赖，一味的盲目移除软件包会导致大部分正在使用的软件包被卸载。

+ 设置ovirt-engine

  ```shell
  engine-setup
  ```

  说明：这里面的配置按具体情况来设置，看不懂就去翻译

  注意：在设置之前请确保你的域名可以ping到对应的主机

+ 以上设置成功之后在浏览器输入 https://<你的域名>:443/ovirt-engine/ 或者 http://<你的域名>:80/ovirt-engine/

+ 后话：在设置过程中如果出现“ [ ERROR ] ”请仔细检查报错

+ 设置成功会出现：[ INFO  ] Execution of setup completed successfully

+ 注意：如果因为防火墙导致不能访问，请执行如下命令：

  ```shell
  firewall-cmd --permanent --add-port=<端口号>/<协议> --zone=<作用域，默认public>
  firewall-cmd --reload
  ```

  



#### 安装计算节点

***

+ oVirt-Node有多种安装方式，本文采用基于现有的Enterprise Linux安装。

+ 有关依赖的问题（如果尚未安装过oVirt 4.2版本可以跳过该步骤）：

  + oVirt-Node 4.2会有一个非常严重的依赖问题，即vdsm与sos的依赖问题，该问题在4.3版本被修复，故采用了4.3版本（目前只测试了4.2版本）

    + 如果之前安装了4.2版本又不想卸载的处理办法（按顺序执行）：

      + 添加版本锁定：

        ```shell
        yum install yum-plugin-versionlock
        yum versionlock sos
        ```

      + 在sos的版本被锁定的情况下，进行yum更新，这将会把新的oVirt的存储库安装，会替换4.2版本

        ```shell
        yum update
        ```

      + 禁用或者删除oVirt 4.2版本的存储库

        ```shell
        # 禁用
        yum-config-manager --disbale ovirt-4.2
        yum-config-manager --disbale ovirt-4.2-extra
        yum clean all
        
        #删除
        yum remove ovirt-release42
        cd /etc/yum.repo.d/
        rm ovirt-4.2*
        yum clean
        yum update
        ```

      + （可选）解锁sos包的版本锁定

        ```shell
        yum versionlock delete sos
        ```

+ 正式开始安装（如果已经更新到4.3版本需跳过该步骤）

  + 安装存储库（版本推荐与oVirt-engine版本一致）

    ```shell
    yum install https://resources.ovirt.org/pub/yum-repo/ovirt-release43.rpm
    yum update
    ```

  + 然后去oVirt-engine中新建主机

    + 需要注意的一点，主机的“Hostname”是你要作为计算节点的计算机的IP地址或域名
    + 密码则是主机的root密码
    + 电源管理配置（PMS，Power Management System）可以不用配置，除非你有那个模块

  + 在oVirt-engine的主机中，你新建的主机的状态为“Up”即表示安装成功（但是这可能需要一些时间）



#### 配置说明（简要说明）

***

+ 都是web操作，也没什么可以讲的，这里主要写了关于命令操作的方式

+ 用户操作

  + 创建用户

    ```shell
    # 添加一个用户
    ovirt-aaa-jdbc-tool user add <用户名> --attribute=firstName=<名> --attribute=lastName=<姓>
    # 为上面的用户添加一个密码和密码的有效期
    ovirt-aaa-jdbc-tool user password-reset <用户名> --password-valid-to="YYYY-MM-DD HH:mm:ss-zone"
    ```

  + 删除用户

    ```shell
    ovirt-aaa-jdbc-tool user delete <用户名>
    ```

    

#### 错误列表/注意事项

***

+ 尚无

