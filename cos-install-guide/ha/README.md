COS HA安装部署文档
=================

## 1. 准备工作

### 1.1 COS镜像安装盘
ISO镜像文件 `cos_production_iso_image.iso`，大小约500M左右, 将ISO文件刻录为一张CD光盘或者USB启动盘.  
注意: 如果准备使用USB启动盘进行安装的话, 请将USB设备的卷标修改为`CDROM`


### 1.2 硬件设备
  - 1台24口千兆（及以上）交换机
  - 6台(及以上)服务器：(3台控制器做HA集群, 3台节点)
    - 8G内存
    - 双路酷睿64位CPU
    - 千兆全双工网卡
    - `2组`本地磁盘 (建议1组做`raid5`安装COS系统,另外1组做`raid10`存放docker镜像数据)

### 1.3 网路拓扑/IP地址池
需要一个单独的C段网络，如10.1.1.0/24，所有COS将部署在同一个LAN中  
此外还需要提前申请一个IP段, 如`10.1.1.10/24 - 10.1.1.200/24`, 用于安装完成后设置容器要使用的IP池.  

## 2. 角色划分说明
其中3个COS为中心控制器(HA集群模式), 其余均为COS节点.  
中心控制器负责统一的资源管理和调度,所有业务容器均运行在节点上.  
网络访问权限只要求`COS节点`可以访问`COS中心控制器`即可.

## 3. 安装部署
> 说明: 安装过程由字符安装界面引导，安装过程中可随时按键 `Ctrl + c` 退出安装过程。退出安装后，如果想要重新启动安装引导程序，按键 `Ctrl + d` 即可。

### 3.1 COS中心控制器HA集群安装
> 说明: 中心控制器HA需要安装3台.每个控制器的部署过程如下:

#### 3.1.1 中心控制器的安装
使用刻录好的ISO镜像安装盘引导服务器启动，如图:  
[![启动引导](pics/002.png)](pics/002.png)

启动完毕后，会自动弹出安装引导程序欢迎界面，回车确认，如图：  
[![启动引导](pics/003.png)](pics/003.png)

接下来进入COS角色选择界面，有2种角色分别是：  

   - **Controller**    安装中心控制器  
   - **Agent**         安装Agent节点  

首台安装，这里我们选择安装角色controller，上下键切换，空格键选中，回车确认，如图：
[![启动引导](pics/004.png)](pics/004.png)

接下来进入中心控制器的参数设定界面，这里需要设定两个参数：  

  - **HTTP   Port**：     中心控制器对外提供服务的HTTP端口，默认80
  - **Cluster  Size**：     COS集群中Etcd集群的初始大小，只能设定为1,3,5,7,9 五种数值，默认为`3`，**强烈建议保持不变**。  

这里我们保持默认值，回车确认，如图：  
[![启动引导](pics/005.png)](pics/005.png)

接下来设置是否启用中心控制器HA部署, 这里我们选择YES,如图:  
[![启动引导](pics/006-1.png)](pics/006-1.png)

接下来进入系统设定界面，这里需要设定主机名称`HostName`和默认账户`cos`的密码`Password`，这里我们假设我们设定主机名为`controller1`(其他控制器主机名以此类推设置为`controller2`,`controller3`)，cos账户密码为`cos`，回车确认，如图：  
[![启动引导](pics/006-3.png)](pics/006-3.png)

接下来进入网络配置界面，首先这个界面上会列出所有识别到的网卡设备。
已经连接网线的网卡为绿色标示<font color="green">[Linked]</font>，未检测到连接网线的网卡为红色标示<font color="red">[NoLink]</font>。  
这里我们只选择其中一块在COS集群中使用的网卡进行配置即可，其他网卡可待安装结束后再进行额外配置。上下键切换，空格键选中，回车确认，如图：  
[![启动引导](pics/007.png)](pics/007.png)

接下来安装程序会尝试用DHCP自动获取一个地址的配置，如图：  
[![启动引导](pics/008.png)](pics/008.png)

接下来我们开始对选中的网卡进行配置，可以在DHCP获取的地址配置基础上进行修改，也可以全部重新编辑设定：  
参数说明如下：  

  - **IP/Mask**:      IP地址和掩码，必须是CIDR格式，比如192.168.159.129/24  
  - **Gateway**:      网关地址，必须是IP地址格式，比如 192.168.159.2  
  - **DnsMaster**:    DNS地址，比如是IP地址格式，比如 192.168.159.2  

检查确认填写无误后，直接回车确认，如图：
[![启动引导](pics/009.png)](pics/009.png)

接下进入安装磁盘选择界面，这里会列出安装程序能够识别出的磁盘设备，按上下键切换设备，空格键选中设备。回车确认，如图：  
[![启动引导](pics/010.png)](pics/010.png)

此时安装引导程序进入最后确认界面  
注意：**确认安装的话所选择安装的磁盘设备上所有数据都会被清空**，确认后直接回车，如图：  
[![启动引导](pics/011.png)](pics/011.png)

安装进度，如图：  
[![启动引导](pics/012.png)](pics/012.png)

安装进度结束后，会弹出Cloud Config的配置确认界面，可以上下键查看配置信息，也可以直接回车，如图：  
[![启动引导](pics/013.png)](pics/013.png)

安装结束，回车确认重启，如图：  
[![启动引导](pics/014.png)](pics/014.png)

> **至此, 第一台控制器安装完毕, 此时是无法访问控制器的管理面板的, 请按照如上步骤继续安装其他两台控制器.**

#### 3.1.2 中心控制器的HA配置
假设3台控制器都安装完毕, 主机名和IP地址分别是:  

  - controller1 - 192.168.122.170/24
  - controller2 - 192.168.122.171/24
  - controller3 - 192.168.122.172/24

##### 3.1.2.1 单独启动mongodb
SSH登录`所有三台`控制器主机. 停止所有组件,并单独启动mongodb:
```bash
$ sudo cspherectl stop
$ sudo cspherectl start mongodb
```

##### 3.1.2.2 配置mongodb replset集群
在第一台控制器`controller1`机器上执行:

```bash
连接本地mongodb:
$ mongo

设置集群参数:  (根据实际情况修改IP地址)
默认情况下, 第一个IP地址就是集群运行起来后的**主**中心控制器:
> rs.initiate({ _id: "rs0", version: 1, members: [ { _id: 0, host: "192.168.122.170:27017"}, { _id: 1, host: "192.168.122.171:27017"}, { _id: 2, host: "192.168.122.172:27017"}  ] });
{ "ok" : 1 }

查看集群状态: (默认第一个IP是主节点`PRIMARY`, 另外两个IP是`SECONDARY`, `health`状态均为1)
> rs0:PRIMARY> rs.status();

退出
> rs0:PRIMARY> exit
bye
```

至此, mongodb repl集群配置完毕.

##### 3.1.2.3 启动其他组件
在`所有三台`控制器主机上执行:
```bash
$ sudo cspherectl  start etcd
$ sudo cspherectl  start docker
$ sudo cspherectl  start monitor // 负责监控mongo集群的切换状态并管理其他组件的启动停止
$ sudo systemctl enable csphere-monitor  // 开机激活 monitor 服务
```
> 说明: **不要手工执行启动** `prometheus` `controller` `agent` 这三个组件了,这三个组件的启动停止由 `monitor` 根据 `monogodb` 的角色状态负责启动停止.

#### 3.1.3 

登录**主**中心控制器，直接用浏览器访问控制器的IP地址，如图  
[![启动引导](pics/015.png)](pics/015.png)

创建一个初始的账号：  
[![启动引导](pics/016.png)](pics/016.png)

登录进入中心控制器的管理后台，默认的license授权无法安装COS节点, 首先需要填入一个**企业版**的license。如图:  
[![启动引导](pics/016-1.png)](pics/016-1.png)

点击`基本设置` `生成COS验证码`，比如这里我们获取到的4位安装码是8070。**不要关闭验证码的界面**。接下来我们要使用这个验证码来安装其他的Agent节点。  
[![启动引导](pics/017.png)](pics/017.png)
[![启动引导](pics/018.png)](pics/018.png)


> 至此, 主控中心的HA集群部署完毕, 下面介绍节点的部署.

### 3.2 COS节点安装
说明:  
> **COS节点至少要安装`CluterSize`台, 也就是中心控制器安装过程中设定的`Etcd集群初始大小`。**  
> **也就是说假如中心控制器安装过程中, `ClusterSize`设置为默认值`3`, 则COS节点主机至少要安装`3`台才能完成集群的首次初始化, 所有的COS节点才能够正常启动。**  
> **在全部安装完成之前，无法从中心控制器页面上看到任何Agent节点的加入。 建议多台Agent并行安装以节省时间。**  

下面将重点描述Agent和中心控制器不同的安装步骤，相同的步骤将会简略描述。  

欢迎界面：  
[![启动引导](pics/019.png)](pics/019.png)

角色选择界面： 注意：此处我们要选择安装角色为 **`agent`**  
[![启动引导](pics/020.png)](pics/020.png)

选择角色为agnet后，紧接着进入Agent 配置界面，这里有两个参数需要配置，说明如下：  

  - **Controller**：            控制器集群的地址和端口，格式必须是 `地址`:`端口`, 多个控制器地址`,`分隔，并且当前的**主控制器必须写在第一个**, 我们这里填:`192.168.122.170:80,192.168.122.171:80,192.168.122.172:80`  
  - **InstallCode**：           Agent安装码，格式必须是4位数字，这里我们填中心控制器界面上生成的安装码：8070  
 
[![启动引导](pics/021-1.png)](pics/021-1.png)


系统配置界面，为了方便识别，我们配置主机名为`node1`，其他Agent主机名为`node2`, `node3` 以此类推：  
[![启动引导](pics/022.png)](pics/022.png)

选择网卡界面，如图：  
[![启动引导](pics/023.png)](pics/023.png)

接下来设定COS节点上的Docker要使用的网络模型，所有COS节点应该使用相同的网络模型，可选择的配置项有：  

  - **Bridge**:    Docker默认的Bridge网络模型，**适合大部分的部署环境**，建议使用  
  - **Ipvlan**:    Ipvlan网络模型，只在当部署环境是**vSphere虚拟化**的环境时选择使用  

[![启动引导](pics/024.png)](pics/024.png)


接下来是节点的网卡地址设置：  
**注意**：  

  - 如果上一步选择的网络模型是Ipvlan网络模型，则各个节点在填写IP地址的时候，需要注意各个节点的IP地址**不要连续**  
  - 如果是Bridge网络模型，则无此限制。  

举例说明：  
> 3台节点，都使用了Ipvlan网络模型，则第一台Agent使用IP地址：192.168.159.101/24，那个第二台Agent的IP地址应该为： 192.168.159.103/24，第三台Agent的IP地址应该为： 192.168.159.105/24  
> 如果3台节点都使用了Bridge网络模型，则节点的IP地址可随意设定，没有限制。  

[![启动引导](pics/025.png)](pics/025.png)

磁盘设备选择界面：  
[![启动引导](pics/026.png)](pics/026.png)

确认选择：  
[![启动引导](pics/027.png)](pics/027.png)

安装进度：  
[![启动引导](pics/028.png)](pics/028.png)

[![启动引导](pics/029.png)](pics/029.png)

安装完毕，安装程序会提示是否重启？回车确认重启：  
[![启动引导](pics/030.png)](pics/030.png)

等节点首次启动后，安装结束。至此，首个节点安装完毕。  

**注意**
> 此时，在控制器页面上是看不到刚安装完毕的首个节点的,因为此时首个节点在等待其他两个节点加入集群, 所以此时不要通过SSH登录节点进行重启操作.

**重复上面步骤，继续完成剩余2个节点的安装** 

在**全部节点**完成安装之后，在中心控制器的页面上才可以看到所有安装完毕的节点主机。  
此时可以在COS安装码界面点击“结束安装”：  
[![启动引导](pics/031.png)](pics/031.png)

[![启动引导](pics/032.png)](pics/032.png)

至此，整个COS集群安装完毕。  


### 3.3 建立镜像仓库

登录到中心控制器上来建立我们的镜像仓库，操作步骤如下  
`镜像仓库` -> `新建镜像仓库`，选择中心控制器主机来创建我们的镜像仓库，并填写中心控制器主机的域名或IP地址，其他配置保持默认，如图：  
[![启动引导](pics/034.png)](pics/034.png)
[![启动引导](pics/036.png)](pics/036.png)
此时我们就在中心控制器上创建了镜像仓库，之后就可以在此仓库中 Push/Pull Docker镜像  

### 3.4 设定容器IP地址分配范围

SSH登录任意一台节点主机上执行：  
假如我们设置docker容器的IP分配池为: 192.168.122.200/24 - 192.168.122.250/24,   
执行:  
```bash
# net-plugin ip-range --ip-start=192.168.122.200/24  --ip-end=192.168.122.250/24
```
说明:
  
  - 如果有多个不连续的IP分配池,可以多次执行上面的命令进行设置.
  - IP分配池的IP地址应该和集群主机的IP地址可以进行通讯,否则无法启动容器
  - IP分配池的起始IP地址掩码应该和集群主机的掩码一致, 否则无法启动容器

如果节点安装时候选择网络模型选择是`ipvlan`，还需执行如下命令：
```bash
复制粘贴执行即可
# net-plugin ip-range --ip-start=172.17.0.1/16  --ip-end=172.17.0.254/16
```
否则docker无法正常启动


### 3.5 转移Docker数据到XFS分区

为了正常使用容器的磁盘空间配额功能，在所有节点主机上要求docker容器数据必须存放在`xfs`文件系统上。  

这里假设节点主机上给容器预留的第二组硬盘块设备为 `/dev/sdb`  
操作步骤如下：  

  - 新建一个数据分区`/dev/sdb1`，格式化为`xfs`，并使用`prjquota`参数挂载：  
```bash
# fdisk /dev/sdb        # 建立一个分区，假设为 /dev/sdb1
# mkfs.xfs  /dev/sdb1   # 格式化为xfs分区 
# mkdir /xfs            # 创建挂载点为 /xfs
# cat > /etc/systemd/system/xfs.mount
[Unit]
Description = docker data xfs mount

[Mount]
What = /dev/sdb1
Where = /xfs
Type = xfs
Options = prjquota

[Install]
WantedBy = multi-user.target

# systemctl  enable xfs.mount    // 激活：启动后自动挂载
# systemctl  start xfs.mount     // 挂载到/xfs
# df -T /xfs                     // 确认已挂载
# mount |grep "/dev/sdb1"        // 确认挂载参数prjquota
```

  - 停止docker daemon并转移数据到xfs分区  
```bash
# cspherectl  stop docker 
# mv /var/lib/docker/ /xfs/docker 
# ln -sv /xfs/docker/ /var/lib/docker
```

  - 重新启动docker daemon
```bash
# cspherectl  start docker
```

## 4. 使用

### 4.1 使用BuildPack

  - 准备镜像  
从[希云微镜像仓库](http://csphere.cn/hub)中下载 buildpack要使用的两个镜像：   
`index.csphere.cn/csphere-buildpack/builder`  
`index.csphere.cn/csphere-buildpack/runner`  
并推送到本地私有镜像仓库，命名为：  
`csphere-buildpack/builder`  
`csphere-buildpack/runner`  

  - 设定buildpack默认所使用的镜像仓库  
中心控制器`设置`页面，`基本设置`，在`buildpack的默认镜像仓库` 下拉列表中选择包含有`csphere-buildpack/builder`和`csphere-buildpack/runner`镜像的私有仓库。  
> 注意：应该有向该镜像仓库push的权限，以保证在构建完成后可以向此仓库push最终的产品镜像。

  - 选定一台节点主机作为buildpack构建运行的主机  
主机页面，选择一台节点主机，增加一个主机标签：  
> buildpack = true  
buildpack运行的时候将会在该主机上进行产品镜像的构建。  
有条件的话，应尽可能选择一台不运行业务的节点作为buildpack的主机

  - 设定buildpack中可引用的公共模板 （非必须）  
到应用模板页面，点击某个模板，修改设置，勾选为公共模板，  
在创建buildpack的时候就可以直接导入这个模板中定义的服务来使用了

  - 应用实例页面，点击`新建应用`来创建buildpack的应用。

### 4.2 导入常用的应用集群模板
希云提供了一系列已经制作好的经过测试的应用集群模板，可直接导入管理平台的应用模板使用，用户可以很方便的快速建立常见的应用集群。  

到 [此处项目](https://github.com/nicescale/cstore) 页面下载每个目录下的tar包，并在管理页面上，应用模板，导入模板，依次选择下载好的tar包文件进行模板的导入。
[![启动引导](pics/037.png)](pics/037.png)

导入前为导入模板指定名称，并选择镜像仓库  
> 所有模板中需要的镜像都可以在希云微镜像仓库中获取到。  
[![启动引导](pics/038.png)](pics/038.png)


## 5. FAQ

### 5.1 问题反馈

在问题反馈前, 请协助在有问题的主机上执行命令:
```bash
# cspherectl snap
/tmp/agent-575fa544e0323a02a9018fd6.tar.gz
```
请将问题描述和对应的文件一并提供给我方. 

### 5.1 跟踪查看各组件日志

  - `主控中心`上各组件日志:  
```liquid
mongodb日志
# tail -f /data/logs/mongodb.log

prometheus日志
# journalctl -u csphere-prometheus -f

etcd日志
# journalctl -u csphere-etcd2-controller -f

docker日志
# journalctl -u csphere-docker-controller -f

agent日志
# journalctl -u csphere-agent -f

主控日志
# journalctl -u csphere-controller -f
```

  - `节点`上各组件日志:  
```liquid
etcd日志
# journalctl  -u csphere-etcd2-agent -f

skydns日志
# journalctl  -u csphere-skydns -f

docker-ipam日志
# journalctl  -u csphere-dockeripam -f

docker日志
# journalctl  -u csphere-docker-agent -f

agent日志
# journalctl  -u csphere-agent -f
```

更多请查看journal帮助 `journalctl  --help`

### 5.2 USB安装报错: CDROM Device Not Ready
安装引导程序报错：
> CDROM Device Not Ready

请确保USB设备的卷标为`CDROM`, 如果还不能解决问题, 请刻录CD更换为光驱设备引导安装.

### 5.3 安装失败: ERROR on Installing COS To Device
安装引导程序报错：
> ERROR on Installing COS To Device /dev/xxx

这种情况多见于一些老型号的物理服务器，是因为硬盘分区表不能通过`blockdev --rereadpt`命令进行热更新，必须重启才可以。  
解决办法：  
重启服务器，使分区表更新生效，再重新开始一次安装即可。  

### 5.4 节点上可以创建容器,但无法启动容器
节点docker日志完整报错：  
>  Plugin Error: IpamDriver.RequestAddress, {"Error":"Key not found in store"}"]}

原因是由于安装完成后没有设定容器的IP池范围，或者容器IP池范围设定有问题. 
请参考[设定容器IP地址分配范围](#34-设定容器IP地址分配范围)重新进行设置

### 5.5 节点上自定义容器网关/DNS地址(>=v1.5)
服务器使用多网卡配置的情况时, `csphere-prepare.bash` 自动生成的docker/agent配置如果不适用, 可以自定义设置.  
  
修改文件 `/var/lib/coreos-install/user_data`, 修改如下两个配置项(默认为空):
```liquid
COS_CUSTOM_DOCKERGW=   # 自定义容器要使用的网关地址
COS_CUSTOM_DOCKERDNS=  # 自定义容器要使用的DNS地址
```
重启系统:
```bash
reboot
```


