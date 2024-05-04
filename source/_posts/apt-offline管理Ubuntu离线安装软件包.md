---
title: Ubuntu离线安装软件包
date: 2018-06-05 19:27:37
tags:
---

### 参考链接

- [apt-offline离线安装指南](http://wiki.ubuntu.org.cn/Apt-offline离线安装指南)
- https://blog.sleeplessbeastie.eu/2014/01/30/how-to-manage-packages-on-an-off-line-debian-system/
- http://manpages.ubuntu.com/manpages/precise/man8/apt-offline.8.html#contenttoc4
- https://www.debian.org/
- https://camicri.github.io/camicri-cube/#/

### 应用场景

- 需要升级/安装软件的电脑无网络。
- 软件要安装到多台电脑上，且软件较大，下载时间长。

### 方案

- 由于**apt-offline**的安装有依赖，离线安装会失败，所以需要借助**Camicri Cube**来完成**apt-offline**的安装。
- 不用**Camicri Cube**完成全部安装是因为它貌似不支持命令行，只有可视化界面操作。如果安装的包较多，就比较麻烦。而**apt-offline**支持命令行操作，一次性可以打包多个安装包，比较方便。

### Camicri Cube完成离线系统升级

- 在**离线**电脑上打开Camicri Cube，创建一个**project**，关闭后打包Cube目录下相应的project。
- 在**联网**电脑上，将打包文件解压到Cube目录下的project目录，然后用Cube打开这个project。
- 点击  **Cube -> Repository -> Download Repositories**  进行下载更新。
- 点击 ** Asterisk -> Upgradable** ， ** Cube -> Download -> Mark All Updates for Download **，  ** Cube -> Download -> Download All Marked Packages** ，就可以完成软件包的升级。
- 在上方的搜索框输入**apt-offline**和**vim**，点download进行下载。
- 在**联网**电脑的project打包后传到**离线**电脑上，**覆盖**原先的project，然后用Cube打开。
- ** Cube -> System -> Update Computer's Repositories** 更新Repository。
- **Cube -> Install -> Mark All Downloaded for Installation**  , **Cube -> Install -> Install All Marked Packages ** ，完成安装软件。

### apt-offline下载及安装离线软件包

- 假设要在**离线**电脑安装mysql-server，先在离线电脑运行以下命令来，完成安装包的行为和定义签名文件：

  ```shell
  apt-offline set debian-install.sig --install-packages mysql-server
  ```

- 将生成的**debian-install.sig**文件上传到**联网**电脑执行下载并打包：

  ```shell
  apt-offline get debian-install.sig --bundle debian-install.zip
  ```

- 将打包好的**debian-install.zip**传到**离线**电脑上，运行下面的命令来更新APT database:

  ```
  sudo apt-offline install debian-install.zip
  ```

- 最后运行apt-get来完成安装：

  ```
  sudo apt-get install mysql-server
  ```

  

