
# 1 搭建私服

>[!note] 概述
>私服就是私有的放置jar包的服务器，由于放置公司有知识产权，不可泄漏的jar包

私服搭建工具的下载地址
https://help.sonatype.com/en/download.html

![[Pasted image 20260513161300.png|560]]

进入该工具的目录，在此目录打开cmd(一定是cmd,powshell不行)
![[Pasted image 20260513161341.png|508]]

执行
```cmd
 nexus.exe /run nexus
```

出现版本号即表示私服启动成功
![[Pasted image 20260513161547.png|596]]

在浏览器打开localhost:8081即可进入私服配置，注册好账户密码即可


# 2 把jar包发布到私服


## 2.1 配置私服

创建仓库
![[Pasted image 20260513210020.png]]

![[Pasted image 20260513210053.png|697]]

加入仓库组
![[Pasted image 20260513210131.png|362]]

maven setting 配置私服仓库
id 为仓库的名字
![[Pasted image 20260513205755.png]]

maven settings配置仓库组
![[Pasted image 20260513210233.png|506]]

聚合工程父工程配置私服仓库

![[Pasted image 20260513210425.png|562]]

install 之后使用deploy 发布包
![[Pasted image 20260513210455.png]]

结果
![[Pasted image 20260513210551.png]]


# 3 给私服配置阿里云代理服务器

![[Pasted image 20260513211837.png|599]]

![[Pasted image 20260513211955.png|591]]


然后把这个仓库加入公共组即可，并且把代理仓库顺序调到最前面生效