title: 使用 Things 3 和桌面的特别姿势
date: 2017-11-24 18:04:23
categories: 技术
tags: 折腾
description: 来自四川山区贫困的小程为了生存改造了桌面
---

Things 3 是我最近一直在用的 GTD 软件，简洁、高效、交互体验好，某宝售价五元的 Things 3，可以说比我两块五买的 Omni* 系列好用不要太多。Things 可以按 Area 区分 Task，每个 Area 下可以建立对应的 Project，每个 Task 下可以建立 SubTask，三级任务分类足以覆盖我绝大多数情况下的事务记录。并且展示方式也很棒，可以按时间线展示、按未来待办展示等等……（哎打广告的先停一停啊…… 总之就是很棒就对了！

但是使用中依然存在痛点：为了看自己的「待办清单」，我每次都会经历以下步骤：

- 打开 alfred
- 输入 things
- 找到不知道从主屏还是副屏中弹出来的 Things 窗口
- 移动鼠标到窗口上
- 点击对应任务组的分类
- 查看待办清单

这么多步骤对于我来讲实在是很容易浇灭使用 GTD 软件的心，于是就容易形成这样的恶性循环：

> 懒得用 GTD 软件 → 生活没有规划 → 人生走向低谷 → 越来越贫困……

可以说后果非常严重了…… 那么该怎么做呢？来自四川的贫困的小程陷入了思索：

> 看 Things 内容很繁琐 → 有没有什么便捷的展示信息的方案 → 命令行 → 一行命令就可以展示所有待办任务岂不快哉？

说干就干，Things 3 官方不提供 CLI 的话，可以看看有没有屌大的自己写，果然被我一搜就搜到了：[AlexanderWillner/things.sh](https://github.com/AlexanderWillner/things.sh)

原来 Things 3 的 DB 是基于 sqlite 开发的，而且没加密……所以只要你愿意你自己写个命令行程序去增删改查都行……但是考虑到我使用 Things 的痛点其实不在增删改上，而且自己乱写 SQL 操作数据库风险太大经不起折腾……所以这个脚本虽然只提供了对于各种内容的查找功能，但是已经足够甚至超出了我所需要的适用范围，加上许多格式不符合我的需求，于是直接上手改吧：

```bash
#!/bin/bash

set -o errexit
# set -o nounset

# default options
limitBy="20"
waitingTag="Waiting for"
orderBy="creationDate"
area=""
project=""

readonly PROGNAME=$(basename $0)
readonly DEFAULT_DB=~/Library/Containers/com.culturedcode.ThingsMac/Data/Library/Application\ Support/Cultured\ Code/Things/Things.sqlite3
readonly THINGSDB=${DB:-$DEFAULT_DB}

# table names
readonly TASKTABLE="TMTask"
readonly AREATABLE="TMArea"
readonly TAGTABLE="TMTag"

# is trashed
readonly ISNOTTRASHED="trashed = 0"
readonly ISTRASHED="trashed = 1"

# status
readonly ISOPEN="status = 0"
readonly ISCANCELLED="status = 2"
readonly ISCOMPLETED="status = 3"

# start
readonly ISNOTSTARTED="start = 0"
readonly ISSTARTED="start = 1"
readonly ISPOSTPONED="start = 2"

# type
readonly ISTASK="type = 0"
readonly ISPROJECT="type = 1"
readonly ISHEADING="type = 2"

# merged queries
readonly IS_UNCOMPLETE_VALID_TASK="type = 0 AND status = 0 AND trashed = 0"
readonly IS_UNCOMPLETE_VALID_PROJECT="type = 1 AND status = 0 AND trashed = 0"

usage() {
  cat <<-EOF
usage: ${PROGNAME} <OPTIONS> [COMMAND]

List to do items from your Things database given a focus area.

COMMAND:
  area  (show tasks in given area)

OPTIONS:
  -a|--area <area>        List tasks in area
EOF
}

# utils for displaying text
function printLineWithLevel() {
  # set \n as separator
  OLD_IFS=$IFS
  IFS=$'\n'

  array=($1)
  level=$2
  indicator=$3
  for data in ${array[@]}
  do
    line=""
    for k in $( seq 1 ${level} )
    do
      line="$indicator$line"
    done
    line="$line $data"
    echo "$line"
  done

  # fallback separator
  IFS=$OLD_IFS
}

# utils for handling SQL result
function printSQLResult() {
  # set \n as separator
  OLD_IFS=$IFS
  IFS=$'\n'

  array=($1)
  level=$2
  if [ -n ${array} ]; then
    for item in ${array[@]}
    do
      # print task name
      printLineWithLevel "$item" $level " "
    done
  fi

  # fallback separator
  IFS=$OLD_IFS
}

t() {
  printSQLResult "`listAreas`" 1
}

listAreas() {
  sqlite3 "$THINGSDB" <<-SQL
SELECT title
FROM ${AREATABLE};
SQL
}

listProjectsInGivenArea() {
  sqlite3 "$THINGSDB" <<-SQL
SELECT title
FROM ${TASKTABLE}
WHERE ${IS_UNCOMPLETE_VALID_PROJECT} AND area=(select uuid from ${AREATABLE} where title="${area}");
SQL
}

listTaskInGivenArea() {
  sqlite3 "$THINGSDB" <<-SQL
SELECT title
FROM ${TASKTABLE}
WHERE ${IS_UNCOMPLETE_VALID_TASK} AND area=(select uuid from ${AREATABLE} where title="${area}") and project is NULL;
SQL
}

listTaskInGivenProject() {
  sqlite3 "$THINGSDB" <<-SQL
SELECT title
FROM ${TASKTABLE}
WHERE ${IS_UNCOMPLETE_VALID_TASK} AND project=(select uuid from ${TASKTABLE} where title="${project}") and area is NULL;
SQL
}

area() {
  # print inbox items
  printLineWithLevel "Inbox" 1 " "
  printSQLResult "`inbox`" 5

  # try to get areas list
  if [[ -z ${area} ]]; then
    # display all areas
    areas=`listAreas`
  else
    # display given areas
    areas=${area}
  fi

  # loop to display all area tasks in given area list
  areaArray=(${areas})
  for item in ${areaArray[@]}
  do
    # list area name
    printLineWithLevel "$item" 1 " "

    # list tasks directly belongs to this area
    area=${item}
    tasks=`listTaskInGivenArea`
    if [[ -n $tasks ]]; then
      # has non-project tasks
      printLineWithLevel "其他" 3 " "
      printSQLResult "`listTaskInGivenArea`" 5
    fi

    # get all projects in this area
    TMP_IFS=$IFS
    IFS=$'\n'
    projects=`listProjectsInGivenArea`
    projects=(${projects})
    IFS=$TEM_IFS
    if [[ -n ${projects} ]]; then
      # has projects under area
      for sproject in ${projects[@]}
      do
        # print project name
        printLineWithLevel "$sproject" 3 " "

        # find all task in project
        project=${sproject}
        printSQLResult "`listTaskInGivenProject`" 5
      done
    fi
  done
}

require_sqlite3() {
  command -v sqlite3 > /dev/null 2>&1 || {
    echo >&2 "ERROR: SQLite3 is required but could not be found."
    exit 1
  }
}

require_db() {
  test -r "$THINGSDB" -a -f "$THINGSDB" || {
    echo >&2 "ERROR: Things database not found at $THINGSDB."
    exit 2
  }
}

require_sqlite3
require_db

while [[ $# -gt 1 ]]; do
  key="$1"
  case ${key} in
    -a|--area) area="$2";shift ;;
    *) ;;
  esac
  shift
done

command=${1:-}

if [[ -n ${command} ]]; then
  case $1 in
    area) area ;;
    *) usage ;;
  esac
else
  usage;
fi
```

使用效果就像这样：

![命令行调用结果](http://ojanerta1.bkt.clouddn.com/2017-11-24-1.png)

OK，事情是不是到这里就结束了呢？因为懒惰而异常贫穷的小程表示还不够：

> 我还是得先唤起命令行窗口 → 输入命令 → 查看结果…… 还是麻烦

有没有更方便的做法？小程看着面前的 Dell 27' 4K 显示器，觉得这么大个显示器，好像平时也就利用了中间这么一小部分啊…… **两侧的空白**能用来做什么呢？小程陷入了沉思……

**桥豆麻袋！**说到这个「两侧的空白」啊，必须先提一下小程最近在使用窗口管理时的策略：

> 4K 27' 显示器是主屏幕，对于绝大多数情况下，只会在屏幕内摆放一个窗口，这个窗口会占据屏幕宽度的百分之 70，占据屏幕高度的百分之 90

就像这样：（通过快捷键让所有窗口 resize 并居中）

![日常状况](http://ojanerta1.bkt.clouddn.com/2017-11-24-2.png)

在有些时候会采取更 focus 的策略，会通过快捷键让窗口全屏：

![Focus 状况](http://ojanerta1.bkt.clouddn.com/2017-11-24-3.png)

在需要查看当前窗口的时候会通过快捷键展示当前显示器中有哪些窗口：

![List 状况](http://ojanerta1.bkt.clouddn.com/2017-11-24-5.png)

唔…… 以上的窗口管理是通过 [Phoenix](https://github.com/kasper/phoenix) 完成的，这款神器呢是以前公司的架构师 [DDD](https://blog.alswl.com/about/) 安利给我的~~（当然也安利给了很多人）~~，这里我就再继续安利出去好了~ 具体介绍请直戳链接围观，简单来讲就是个可编程的（js）、利用键盘快捷键对窗口进行管理的软件~ 如果真的感兴趣请戳以下链接围观：[windows management for hacker](https://blog.alswl.com/2016/04/windows-management-for-hacker/)

收回来一下，其实上面就讲了一件事：**我平时的工作窗口其实还有两侧的空白没有被利用起来。**

那我能利用这两块空白做点什么呢？

想到这里，我又想起来曾经在 Windows 7 环境下用过的「便签」小工具…… 其实这玩意在 Mac 上也有，不过被放在 Dashboard 上了，想用还得去另一块窗口，忒不方便，而且**贼 jb 丑**啊……

于是想到这里，贫困的小程不禁想到：**有没有办法在空白屏幕上写字啊**？

会这么想也是有原因的，以前在 Windows 时代我想很多人都折腾过 [Rainmeter](https://www.rainmeter.net/) 之类的桌面美化软件，在桌面上画个画模拟个仪表盘还不是个分分钟的事情…… 那 Mac 上该咋整咧？

Google 了一下，很快就发现了这么个有趣的小玩意：[GeekTool](https://www.tynsoe.org/v2/geektool/)

他能干什么呢…… 简单来讲哦，这个小工具可以让你在桌面上定义一个区域，显示一段脚本的输出结果、一幅图、一个网页以及一个日志文件的输出，当然这些东西都是不可交互的。就像这样：

![GeekTool](http://ojanerta1.bkt.clouddn.com/2017-11-24-6.png)

可以看到这里显示了我的一行命令「date」的执行结果……

这…… 就很他娘的有趣了！

结合之前的 Things CLI…… 于是空出了左边屏幕，填入脚本：

Bazinga！

![最终效果](http://ojanerta1.bkt.clouddn.com/2017-11-24-7.jpeg)

可以看到屏幕左边空白显示了我 Things 里面所有待办的事项，右边则显示了我一个固定的 Note.md 里面的内容，我有时候会通过命令行快速记录一些东西，都会记录在这里。

这个脚本会每五秒执行一次，所以基本上你在 Things 里面操作了待办事项之后可以瞬间显示在这里~~

呼……bb 这么多，至此，我使用 Things 3 的时候遇到的痛点就都解决啦~ 并且还开发出了一个新的信息展示区域~ 开心~

这个小工具我已经使用了超过一周了，目前运行良好~ 几乎没有什么性能开销，你唯一需要做的就是：空出一块屏幕，定义一个脚本，搞定你的输出，然后 enjoy the DESKTOP！
