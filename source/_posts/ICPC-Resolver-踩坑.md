---
title: ICPC Resolver 踩坑
date: 2019-05-12 22:09:44
tags:
---

## 应用场景

从 DOMjudge 系统中导出数据，使用 ICPC Tools/Resolver 滚榜。

DOMjudge 版本：7.0.1。

Resolver 版本：2.0.1798。如果使用 DOMjudge 评测，建议使用 2.1 及以上版本。

## 数据操作

1. 搜索 [ICPC Tools](<https://icpc.baylor.edu/icpctools/>)，下载 ICPC Resolver.rar。

2. 运行 award.sh，通过 REST 导入 event feed（一场比赛的所有信息流）。

   ```shell
   URL: http://59.77.134.102/domjudge/api/contests/5
   USER: amdin
   Password: *******
   ```

   点击 `save` 保存为 "events.xml"。

3. 目前版本（2.0.1798）的 Resolver 存在 bug，需要**手动**修订 events.xml 文件：

   1. 第 1、2 行重复 `<contest>`，删除其中一行。

   2. `<problem>` 中的生成 `<id>` 从 0 开始，改成从 1 开始（否则导致部分提交不合法）。

   3. 删除信息不全的队伍，必要信息有：

      ```xml
      <team>
        <id>221801437</id>
        <name>Teamaaa</name>
        <university>福州大学</university>
        <university-short-name>福州大学</university-short-name>
        <region>Participants</region>
      </team>
      ```

      `university-short-name` 为滚榜中显示的学校名称，故这里写学校全称。

   4. 末尾添加 `finalized` 信息：

      ```xml
      <finalized>
        <last-gold>1</last-gold>
        <last-silver>2</last-silver>
        <last-bronze>3</last-bronze>
        <timestamp>1557574214.130</timestamp>
      </finalized>
      </contest>
      ```

      `timestamp` 可设置成任意值。
   
   可使用该代码 [**icpc_resolver_revise_events_xml**](<https://github.com/McGinn7/myscript/blob/master/icpc_resolver_revise_events_xml.py>) 修订 events.xml。
   
4. 使用 award.sh 打开**处理后**的 events.xml，设置金银铜奖人数，然后导出新的 xml 文件，并重新修订 events.xml 文件。

   由于 award.sh 金银铜每组上限 10 个，共 30 个。不符合中国国情，故需要在 events.xml 中手动修改 `last-gold` 等字段。若 `last-gold=3, last-silver=10`，则表示设置金奖 3 个，银奖 7 个。

   生成的 `<award>` 的 citation 可设置成 “金奖”、“银奖” 中文显示。

5. 调用以下指令开始滚榜：

   ```shell
   resolver.sh <Path to CDP>
   ```

6. CDP（[Contest Data Package](<https://clics.ecs.baylor.edu/index.php?title=CDP>)）是提供榜单需要的数据目录，其中包括：

   1. config 目录，根据官方 wiki 设置即可，**必要**文件包括：
      1. contest.yaml：设置比赛标题、时长和封榜时间。
      2. problemset.yaml：设置题目 id，题目名称。
      3. groups.tsv, teams.tsv：从 DOMjudge 导出。
      4. <del>userdata.tsv</del>：官方 wiki 提示必须，实际上**似乎可去除**。
   2. events.xml：比赛信息；
   3. images/logo/team_id.png：学校图标，每个学校的在 events.xml 的第一支队伍 id，id 不包含前导 0； 
   4. images/team/team_id.jpg：队伍照片，若出现 Out Of Memory 问题，则限制队伍照片的大小或者加大 resolver.sh 中 -Xmx 参数。

7. 榜单目前并不支持队伍、学校的中文显示，需要使用压缩工具（如 Bandizip）打开 "resolver/lib/presentContest.jar"，使用支持中文的字体替换 "font/HELV.PFB" 即可。 