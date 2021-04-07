---
title: "在Google Cloud Platform上运行Jupyter Notebook"
date: 2018-12-14T17:03:12+11:00
draft: false
categories: ["Cloud"]
---

# 在Google Cloud Platform上运行Jupyter Notebook

## 简介
本文取材自 [Amulya Aankul](https://towardsdatascience.com/@aankul.a) 发布在 [Medium](https://medium.com/) 的 [Running Jupyter Notebook on Google Cloud Platform in 15 min](https://towardsdatascience.com/running-jupyter-notebook-in-google-cloud-platform-in-15-min-61e16da34d52)，主要介绍如何在Google Cloud Platform上搭建服务器，并在服务器上安装和运行Jupyter Notebook。

## 服务器搭建

### 创建账号
首先在[Google Cloud Platform](https://cloud.google.com/)上创建一个账号。


### 创建新项目
点击左上角"Google Cloud Platform"右边的三个点，点击"NEW PROJECT"创建新项目。

![1](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Run-Jupyter-Notebook-on-GCP/1.png)

![2](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Run-Jupyter-Notebook-on-GCP/2.png)

### 创建虚拟机
进入刚才创建的项目，从左侧边栏点击 Compute Engine -> VM instances 进入虚拟机页面。点击Create创建一个新的虚拟机实例（VM instance）

![3](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Run-Jupyter-Notebook-on-GCP/3.png))

根据需求填写和选择 Name, Region, Zone, Machine Type和Boot Disk。在 Firewall 选项中选中 Allow HTTP traffic 和 Allow HTTPS traffic, 在下方的 Disks 选项卡中取消勾选 Delete boot disk when instance is deleted。最后点击 Create，虚拟机实例就创建好了。

![4](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Run-Jupyter-Notebook-on-GCP/4.png)

### 设置静态IP
默认情况下，外网IP是动态变化的，为了方便访问服务器，我们可以将其设置为静态的。

![5](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Run-Jupyter-Notebook-on-GCP/5.png)

从左侧边栏点击 VPC Network -> External IP Address，可以看到当前项目下的所有虚拟机，依次点击虚拟机实例对应的Type标签和Static标签，将外网IP设置为静态的。

![6](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Run-Jupyter-Notebook-on-GCP/6.png)

### 设置防火墙

从左侧边栏点击 VPC Network -> Firewall rules，点击上方的 CREATE FIREWALL RULE，创建一个新的防火墙规则。

![7](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Run-Jupyter-Notebook-on-GCP/7.png)

根据需求填写 Name，将 Targets 勾选为 All instances in the network，在 Source IP ranges 中填写 0.0.0.0/0，在 Protocols and ports 中勾选 tcp，填写一个端口范围，用于之后访问 Jupyter Notebook。

![8](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Run-Jupyter-Notebook-on-GCP/8.png)

### 连接虚拟机

回到VM instances，根据外网IP连接上刚才创建的虚拟机。可以直接从谷歌提供的web终端连接，也可以通过其他途径连接。Windows 下可以使用Putty，Linux 和 Unix 系统可以直接使用SSH连接。

![9](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Run-Jupyter-Notebook-on-GCP/9.png)

## 配置 Jupyter Notebook

### 安装 Jupyter Notebook

在终端中输入```wget http://repo.continuum.io/archive/Anaconda3-4.0.0-Linux-x86_64.sh```
获取 Anaconda 3 的安装文件

接下来输入```bash Anaconda3-4.0.0-Linux-x86_64.sh```
运行该文件，并根据屏幕提示安装 Anaconda 3。

安装好之后读取启动文件```source ~/.bashrc```
以使用 Anaconda 3

### 修改配置文件

创建 Jupyter Notebook 的配置文件```jupyter notebook --generate-config```

使用Vim或其他编辑器打开该配置文件```vi ~/.jupyter/jupyter_notebook_config.py```

在该文件中加入相应的设置
```
c = get_config()
c.NotebookApp.ip = '*'
c.NotebookApp.open_browser = False
c.NotebookApp.port = <Port Number>
```
在 \<Port Number> 处填写 Jupyter Notebook 使用的端口号，该端口号应该是在防火墙规则的端口范围之内的，否则将不能够通过外网IP和端口号访问 Jupyter Notebook。填写完之后使用```:wq```命令保存该文件。

![10](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Run-Jupyter-Notebook-on-GCP/10.png)


### 启动 Jupyter Notebook

最后，在终端中输入
```jupyter-notebook --no-browser --port=<Port Number>```
来启动 Jupyter Notebook，当然也可以使用
```nohup jupyter-notebook --no-browser --port=<Port Number> > jupyter.log &```
指令忽略挂起信号，让 Jupyter Notebook 一直在后台运行，并将控制台信息输出到 jupyter.log 文件中。

最后在浏览器中输入 IP 地址和端口号（例如156.73.83.51:4813）就能打开 Jupyter Notebook 了！

![11](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Run-Jupyter-Notebook-on-GCP/11.png)


## 参考资料

[Running Jupyter Notebook on Google Cloud Platform in 15 min](https://towardsdatascience.com/running-jupyter-notebook-in-google-cloud-platform-in-15-min-61e16da34d52)