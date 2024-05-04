---
title: DOMjudge 配置
date: 2018-10-04 23:29:36
tags:
- domjudge
---

## Domserver 部署

### PHP timezone

1. php.ini 文件位置

   - CentOS/RedHat/Fedora = /etc/php.ini
   - Ubuntu/Debian/LinuxMint = /etc/php5/apache2/php.ini

2. 选择时区，通常定位为 "Asia/Shanghai"

   - [PHP: List of Supported Timezones](https://php.net/manual/en/timezones.php)

3. 编辑 php.ini 文件

   ```php
   date.timezone = "Asia/Shanghai"
   ```

4. 重启 Apache Service。

   ```shell
   sudo service apache2 restart
   ```

### MySQL maximum connections

原先编辑 *\etc\mysql\my.cnf* 添加 **max_connections = 1000**，然后重启 apache2 即可。但是设置后一直为 214，因为该值受限于 *table_open_ache* 和 *open_files_limit*。

下面的方法在 Ubuntu 16.04 和 MySQL 5.7.23 版本实验成功：

1. 运行下面的命令，设置 open_files_limit

   ```shell
   systemctl edit mysql
   ```

2. 输入以下内容

   ```shell
   [Service]
   LimitNOFILE=8000
   ```

3. 重启服务

   ```shell
   systemctl daemon-reload
   systemctl restart mysql
   ```

## 数据导入

### 测试数据导入

在 Problems 页面下可以编辑 Problem name, Time limit 这些信息，测试数据（Testcases）在页面上**只能**单点添加和修改，并且不能删除。

批量导入测试数据步骤：

1. 先添加题目，填入"Problem name"、"Time limit" 和 "Memory limit" 等信息，其余的保存默认即可，但是暂不添加 "Testcases"。

2. 点击该题目的导出按钮，下载得到一个压缩包。解压后的文件夹内容如下：

   ```
   // 在Windows使用 tree/f 生成该目录文本信息
   │  domjudge-problem.ini
   │  problem.yaml
   │
   ├─data
   │  ├─sample
   │  └─secret
   │
   └─problem_statement
           problem.pdf
   ```

3. 目录 *sample* 和 *secret* 可自行创建，两个目录存放样例数据和隐藏数据（即测试数据）。

4. 将数据 "0.in"，“0.ans” 放入对应的文件夹下，重新压缩成新的压缩包。

5. 在题目的编辑页面，通过 *Upload problem archive* 上传新压缩包。注意 *Contest* 选项选择为 *Do not add / update contest data*，否则可能会上传失败。

### 队伍账号导入

- [Domjudge队伍导入 - 参考链接](https://www.domjudge.org/pipermail/domjudge-devel/2015-September/001753.html)

- 需要在 home > import / export 页面下，导入 *teams.tsv* 和 *accounts.tsv* 这两个文件。在编辑这两个文件时，需要使用 **UTF-8** 格式，否则会上传失败或导致乱码。

- **teams.tsv**

   该文件用于描述队伍信息，包含一版本行，接着每个队伍占用一行，每行包括用制表符（tab）分隔的字段。

   首行为版本行，格式如下：

   <table style="border-collapse:collapse;border-spacing:0;border-color:#ccc" class="tg"><tr><th style="font-family:Arial, sans-serif;font-size:14px;font-weight:bold;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#ffffff;background-color:#fd6864;text-align:left">Field</th><th style="font-family:Arial, sans-serif;font-size:14px;font-weight:bold;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#ffffff;background-color:#fd6864;text-align:left">Description</th><th style="font-family:Arial, sans-serif;font-size:14px;font-weight:bold;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#ffffff;background-color:#fd6864;text-align:left">Example</th><th style="font-family:Arial, sans-serif;font-size:14px;font-weight:bold;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#ffffff;background-color:#fd6864;text-align:left">Type</th></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">1</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">Label</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">teams</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">fixed string (always same value)</td></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">2</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">Version number</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">1</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">integer</td></tr></table>

   队伍描述行，格式如下：

   <table style="border-collapse:collapse;border-spacing:0;border-color:#ccc" class="tg"><tr><th style="font-family:Arial, sans-serif;font-size:14px;font-weight:bold;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#ffffff;background-color:#fd6864;text-align:left">Field</th><th style="font-family:Arial, sans-serif;font-size:14px;font-weight:bold;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#ffffff;background-color:#fd6864;text-align:left">Description</th><th style="font-family:Arial, sans-serif;font-size:14px;font-weight:bold;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#ffffff;background-color:#fd6864;text-align:left">Example</th><th style="font-family:Arial, sans-serif;font-size:14px;font-weight:bold;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#ffffff;background-color:#fd6864;text-align:left">Type</th></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left">1</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left">Team Number</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left">22</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left">integer</td></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left">2</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left">External ID</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left">24314</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left">integer</td></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">3</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">Group ID</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">3</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">integer</td></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">4</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">Team name</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">Hoos</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">string</td></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">5</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">Institution name</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">Fuzhou University</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">string</td></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">6</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">Institution short name</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">FZU</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">string</td></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">7</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">Country Code</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">CHN</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">string ISO 3166-1 alpha-3</td></tr>
<tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">8</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">Affiliation External ID</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">Fuzhou University</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:inherit;color:#333;background-color:#fff;text-align:left;vertical-align:top">string</td></tr>
   </table>
   
   `Group ID` 对应 Categories 中的 ID，表示队伍的角色，如女队、打星队等。

- **accounts.tsv**

   该文件用于描述账号信息，同样包含版本行和账号行，每个账号占用一行。

   首行版本行的格式如下：

   <table style="border-collapse:collapse;border-spacing:0;border-color:#ccc" class="tg"><tr><th style="font-family:Arial, sans-serif;font-size:14px;font-weight:bold;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#ffffff;background-color:#fd6864;text-align:left">Field</th><th style="font-family:Arial, sans-serif;font-size:14px;font-weight:bold;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#ffffff;background-color:#fd6864;text-align:left">Description</th><th style="font-family:Arial, sans-serif;font-size:14px;font-weight:bold;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#ffffff;background-color:#fd6864;text-align:left">Example</th><th style="font-family:Arial, sans-serif;font-size:14px;font-weight:bold;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#ffffff;background-color:#fd6864;text-align:left">Type</th></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">1</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">Label</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">accounts</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">fixed string (always same value)</td></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">2</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">Version number</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">1</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">integer</td></tr></table>

   账号描述行的格式如下：

   <table style="border-collapse:collapse;border-spacing:0;border-color:#ccc" class="tg"><tr><th style="font-family:Arial, sans-serif;font-size:14px;font-weight:bold;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#ffffff;background-color:#fd6864;text-align:left">Field</th><th style="font-family:Arial, sans-serif;font-size:14px;font-weight:bold;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#ffffff;background-color:#fd6864;text-align:left">Description</th><th style="font-family:Arial, sans-serif;font-size:14px;font-weight:bold;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#ffffff;background-color:#fd6864;text-align:left">Example</th><th style="font-family:Arial, sans-serif;font-size:14px;font-weight:bold;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#ffffff;background-color:#fd6864;text-align:left">Type</th></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">1</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">Account Type</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">team</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">string</td></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">2</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">Full Name</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">wtf</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">string</td></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">3</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">Username</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">team099</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">string</td></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">4</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">Password</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">B!5MWJiy</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:0px;overflow:hidden;word-break:normal;border-top-width:1px;border-bottom-width:1px;border-color:#ccc;color:#333;background-color:#fff;text-align:left">string</td></tr></table>

   Account Type 取值为：team, judge, admin, analyst。这里要导入队伍账号，所以该字段在这固定为 team。

   **注意：**需要设置 `Username` 的格式才能将账号和队伍关联起来，规则为：`Username` 的整数部分需要和 **team.tsv** 中的 `Team Number` 一致。比如一支队伍 `Team Number` 为 99，则 `Username`  可以设置为 team-099。

- 导入样例

   team.tsv 文件

   ```
   teams	1
   8	team008	3	三核战队	福州大学	FZDX	CHN	福州大学
   18	team018	3	挂机不队	福州大学	FZDX	CHN	福州大学
   31	team031	3	这都是什么鬼	福州大学	FZDX	CHN	福州大学
   ```

   accounts.tsv 文件

   ```
   accounts	1
   team	三核战队	team008	T3yRt3
   team	挂机不队	team018	86MFyB
   team	这都是什么鬼	team031	RTJr6e
   ```


## 评测机

### Unprivileged user and group

```shell
// 下面这条命令是必须运行的
useradd -d /nonexistent -U -M -s /bin/false domjudge-run
// X=1~4，X通常等同于CPU核心数
useradd -d /nonexistent -U -M -s /bin/false domjudge-run-X 
```

### Linux Control Groups

- 每次**重启**都需要运行 ***judgehost/bin/create_cgroups***，否则提交会编译错误。

## 问题集锦

### 1. 比赛正常需要几台机器？

个人认为正常应该至少需要 4 台机器，配置较高的作为主服务器，即 DOMserver，提供比赛的 web 页面；一台打印服务器，也安装 DOMserver，但开放 print 页面，达到比赛与打印分开，减少宕机对选手的影响；两台评测机，即 Judgehost，这样如果一台宕机，也有另一台继续评测，而不是完全中断评测，并且修复后可随时上线新的评测机。

由于安装的机器较多且安装包大，建议使用 apt-offline 打包需要的安装包，节省安装时间。

### 2. 气球状态页面返回 500 错误

正式比赛开始时，contest 只能留有一个比赛。（个人不知道原因）。

### 3. 测试数据上传失败

可能需要修改的几个文件：

1. /etc/apache2/conf-avaliable/domjudge.conf

   ```php
   <IfModule mod_php7.c>
   php_value max_file_uploads    101
   php_value upload_max_filesize 128M
   php_value post_max_size       128M
   php_value memory_limit        512M
   </IfModule>
   ```

2. /etc/mysql/my.cnf

   ```cnf
   [mysqld]
   max_connections = 10000
   max_allowed_packet = 512M
   innodb_log_file_size = 512M
   ```

### 4. 如何让 DOMJudge 支持多台打印机？

修改 domserver/webapp/src/DOMJudgeBundle/Utils/Printing.php 中 `cmd` 指令：

```php
$cmd = "enscript -C " . $highlight
    . " -d " . $printername 		// 指定打印机名
    . " -b " . escapeshellarg($header)
    . " -a 0-10 "
    . " -f Courier9 "
//    . " -p $tmp "
    . escapeshellarg($filename) . " 2>&1";
```

去除 `-p $tmp` ，添加 `-d` 参数，指定打印机打印。多台打印机则让打印机名轮转即可。

