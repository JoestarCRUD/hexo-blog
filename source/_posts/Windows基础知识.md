---
title: Windows基础知识
date: 2020-11-1 18:07:40
categories: 
- 操作系统
tags: 
 - 批处理
 - NTFS
---

# 批处理

## 变量相关

&nbsp;&nbsp;&nbsp;&nbsp;我们观察如下的批处理代码：

> @echo off  
> color 0a  
> title 小程序 v1.0  
> echo 菜单  
> echo 1.修改管理员密码  
> echo 2.定时关机  
> echo 3.退出本程序  
> set /p u=请输入用户名：  
> set /p p=请输入新密码：  
> net user %u% %p% >nul  
> pause

效果如下：

![图片](https://raw.githubusercontent.com/JoestarCRUD/MarkdownPhotos/master/%E6%89%B9%E5%A4%84%E7%90%86%E5%B0%8F%E7%A8%8B%E5%BA%8F%E4%BE%8B1.png)

对其中的批处理语句简单解释一下：

- set /p var(变量)=value(字符串)
  获取输入的字符，把它赋值给 value
- %u%
  提取变量 u 的值
- net user 用户名 密码
  修改账户的密码（全 Windows 平台支持）
- \>nul 或 1\>nul  
  屏蔽操作成功显示的信息，但是错误信息还是会显示。那么于是对应也有：
  - 2\>nul  
    屏蔽操作失败显示的消息，如果成功依旧会显示 - \>nul 2\>nul  
    全部都屏蔽，不管成功失败

## 选择和区块

&nbsp;&nbsp;&nbsp;&nbsp;我们观察如下的批处理代码：

> @echo off  
> color 0a  
> title 飞哥小程序 v1.0
>
> :menu  
> cls  
> echo ==========================  
> echo 菜单  
> echo 1、修改管理员密码  
> echo 2、定时关机  
> echo 3、退出本程序  
> echo ===========================  
> set /p num=您的选择是：  
> if "%num%"=="1" goto 1  
> if "%num%"=="2" goto 2  
> if "%num%"=="3" goto 3  
> echo 被闹，好好输！  
> pause  
> goto menu  
> :1  
> set /p u=请输入用户名:  
> set /p p=请输入新密码:  
> net user %u% %p% >nul  
> echo 您的密码已经设置成功!  
> pause  
> goto menu  
> :2  
> set /p time=请输入时间：  
> shutdown -s -t %time%  
> set /p x=是否取消（1：是，0：否）：  
> if "%x%"=="1" shutdown -a  
> goto menu  
> pause  
> :3  
> exit

![图片](https://raw.githubusercontent.com/JoestarCRUD/MarkdownPhotos/master/%E9%80%89%E6%8B%A9%E5%92%8C%E5%88%86%E5%9D%97%E4%BE%8B%E5%AD%902.png)

对其中的批处理语句简单解释一下：

- :menu
  menu 区块，这里其实就是把区块命名为 menu，仅仅作为标识，是后续 goto 语句跳转的锚点
- goto 区块名
  程序跳转到该区块
- shutdown -s -t 100
  定时关机.我们可以再看看其他常用的 shutdown 命令：
  > shutdown -a #取消关机  
  > shutdown -s #关机  
  > shutdown -f 　 #强行关闭应用程序  
  > shutdown -l 　 #注销当前用户  
  > shutdown -r 　 #关机并重启  
  > shutdown -s -t 时间　 #定时关机

# Windows 的用户和组

&nbsp;&nbsp;&nbsp;&nbsp;每个用户都会有自己的唯一安全标识，称为 SID，其中 Windows 的系统管理员 administrator 的 UID 时 500，其他用户一概从 1000 开始；Linux 的 root 用户的 UID 是 0，其他用户一概从 1000 开始。

Windows 上的账号和密码是存储在：C:\windows\system32\config\SAM 文件里的。

&nbsp;&nbsp;&nbsp;&nbsp;Windows 默认的内置账户有两种：

- 给人用的：
  - administrator 管理员账户
  - guest 来宾账户
- 计算机服务组件：
  - system 系统账户，权限至高无上
  - local services 本地服务账户 普通权限
  - network services 网络服务账户 普通权限

与账户相关的 Windows Dos 命令

> net user  
> net user 账户名 字符串 #修改密码  
> net user 账户名 #查看账户信息  
> net user 账户名 密码 /add #新建账户  
> net user 账户名 密码 /del #删除账户  
> net user 账户名 /active:yes/no #激活或禁用账户

## 组

&nbsp;&nbsp;&nbsp;&nbsp;组的作用是为了简化赋权的流程，赋权既可以对用户进行也可以对组进行，通过将用户批量加入组的过程，可以实现对用户的批量赋权。Windows 自带一批内置组，内置组的权限已经被系统赋予了：

> administrators #管理员组  
> guests #来宾组  
> users #普通用户组  
> network #网络配置组  
> print #打印机组  
> Remote Desktop #远程桌面组

与组有关的 Windows Dose 命令

> net localgroup #查看组  
> net localgroup 组名 #查看组  
> net localgroup 组名 /add #创建组  
> net localgroup 组名 用户名 /add #添加组成员  
> net localgroup 组名 用户名 /del #删除组成员  
> net localgroup 组名 /del #删除组

# NTFS 安全权限

&nbsp;&nbsp;&nbsp;&nbsp;NTFS（New Technology File System），是 Microsoft 公司开发的专用文件系统，从 Windows NT 3.1 开始成为 Windows NT 家族的标准文件系统。NTFS 取代 FAT（文件分配表）和 HPFS（高性能文件系统）并进行一系列改进，例如增强对元数据的支持，使用更高级的数据结构以提升性能、可靠性和磁盘空间利用率，并附带一系列增强功能，如访问控制列表（ACL）和文件系统日志。而在此之上的 NTFS 安全权限则是能主要实现以下三种功能：

1. 通过设置 NTFS，实现不同用户访问不同的对象（文件或文件夹）的权限
2. 分配正确的访问权限后，用户才能访问不同的对象
3. 设置权限防止资源被篡改被删除

## 修改 NTFS 权限

&nbsp;&nbsp;&nbsp;&nbsp;在介绍修改权限之前，我们先简单了解一下 NTFS 的特点，这是权限得以修改的前提和基础。

1. NTFS 提高了磁盘读写的性能
2. 可靠性更佳，它提供了文件加密系统，提供了访问控制列表 ACL(Access Controll List)用以设置权限
3. 增加了磁盘利用率。例如在分割压缩磁盘空间时，可以选择磁盘配额
4. 相较 FAT32，支持单个文件大于 4G

### 1. 取消 NTFS 继承

&nbsp;&nbsp;&nbsp;&nbsp;取消后可以修改权限列表，在 Windows10 中：文件夹右键->安全选项卡->高级->点击禁用继承

### 2. 文件和文件夹的权限

<table border="1">
  <tr>
    <td rowspan="2" align="center">文件/文件夹权限</td>
    <td colspan="2" align="center">权限内容</td>
  </tr>
  <tr>
    <td align="center">文件</td>
    <td align="center">文件夹</td>
  </tr>
  <tr>
    <td>完全控制</td>
    <td>拥有读取、写入、删除文件、及特殊权限</td>
    <td>拥有对文件夹及文件的读取、写入、删除文件、及特殊权限</td>
  </tr>
  <tr>
    <td>修改</td>
    <td>拥有读取、写入、删除文件的权限</td>
    <td>拥有对文件夹及文件的读取、写入、删除文件的权限</td>
  </tr>
  <tr>
    <td>读取与执行</td>
    <td>拥有读取、及执行文件的权限</td>
    <td>拥有对文件夹及文件读取、及执行文件的权限</td>
  </tr>
  <tr>
    <td>读取</td>
    <td>拥有读取的权限</td>
    <td>拥有对文件夹中的文件下载、读取的权限</td>
  </tr>
  <tr>
    <td>写入</td>
    <td>拥有修改文件内容的权限</td>
    <td>拥有在文件夹中创建新文件的权限</td>
  </tr>
  <tr>
    <td>特殊权限</td>
    <td>控制文件权限列表的权限</td>
    <td>控制文件夹权限列表的权限</td>
  </tr>
</table>  
<br/>  

### 3. 权限累加
当用户属于多个组时，权限时可以累加的  
### 4. 拒绝最大
当用户权限累加时，如遇到拒绝权限，拒绝最大
### 5. 文件赋值剪切对权限也有影响
文件复制后，跨分区移动，文件的权限会被目标文件覆盖
文件同分区移动，文件的权限不会被目标文件覆盖