title: 用 Dayone 写日报的正确姿势
date: 2017-11-16 12:00:00
categories: 折腾
tags: [效率,工具]
description: 不是工具不好用，实在是人太懒呐
---

写日报是个好习惯，管理日记的 Dayone 是个好软件，但是状态栏小工具实在不好用，而且 Dayone 不提供模板，每次写日报的时候都得自己手动写格式，实在是非常烦人。而且每天的日报光有我自己 bb，没啥别的实锤证明我自己干了些什么事， 实在是非常的不大丈夫。

就在我因为懒得开 Dayone 而中断了好多天写日报的时候，我突然发现，Dayone 居然提供 CLI 工具：[ CLI for Dayone2 ](http://help.dayoneapp.com/day-one-2-0/command-line-interface-cli)

良心啊！这可给我高兴坏了，虽然 CLI 提供功能很少（只有**新增**…… 没有修改查询删除啥的），但是已经够了~

工具实现了三个功能：

- 可以用 Vim 直接在命令行里写日报啦
- 通过使用 Vim 的模板功能，终于可以通过模板新建日记惹（md 格式）
- 可以自动在日报结尾添加今天的 Git 提交记录作为一天工作的实锤啦


工具设置起来稍微有些麻烦，分几个步骤：

首先需要安装 Dayone2 的 CLI：

```bash
sudo /Applications/Day\ One.app/Contents/Resources/install_cli.sh
```

然后需要在 .vimrc 里面设置某种文件打开时的模板，把这段话写到 .vimrc 文件的末尾：

```bash
autocmd BufNewFile $YOUR_TMPLATE_FILE_NAME 0r $PATH_TO_YOUR_TEMPLATE_FILE | autocmd! BufNewFile
```

这段代码的意思是在你使用 Vim 打开符合 `YOUR_TMPLATE_FILE_NAME` 文件名的文件的时候会自动往里写入给定文件的内容。就是通过这个方法来完成 Vim 模板的设定的~ 你可以自己设定你自己需要的模板~~

然后接下来是打印当天 Git 提交记录的脚本：

```bash
#!/usr/bin/env sh
 
# receive param as query date
query=$1
 
# make date string
if [ -n "$query" ]; then
  date=$query
else
  date=`date +%Y-%m-%d`
fi
 
# set git workspace
workspaces=$PATH_TO_YOUR_WORKSPACE
names=$YOUR_NAME
 
# recurrsively find git log.
for workspace in $workspaces
do
  # get all dirs
  repos=`ls $workspace/`
 
  for repoPath in $repos
  do
    cd $workspace/$repoPath
 
    for name in $names
    do
      logs=`git log --after "$date 00:00" --before "$date 23:59" --oneline --no-merges --author $name`
      if [ -n "$logs" ]; then
        echo "Found logs @ $repoPath, commits list below:\n$logs\n"
      fi
    done
  done
done
```

注意你需要在这个脚本里定义你的 workspace 和你的用户名哈~

接下来就是把上面的功能合在一起的 shell 脚本，注意这里我们把上面获取 Git log 的脚本命名为 `list_git.sh` 放到同一个目录下：

```bash
#!/usr/bin/bash
 
# ask for emotion
echo "今天感觉怎么样？"
read ANS
 
# this file name will trigger vim to create a new file with given template
vim $YOUR_TMPLATE_FILE_NAME
 
# get daily content
CONTENT=`cat $YOUR_TMPLATE_FILE_NAME`
 
# get title
DATE=`date '+%Y/%m/%d 周%a'`
TITLE="**工作日报 - $DATE**"
 
# get daily git logs
dir=`dirname $0`
GITLOG=`sh $dir/list_git.sh`
 
# save content to dayone2
FINAL_CONTENT="$TITLE\n> 心情：$ANS \n\n$CONTENT \n"
if test -n "$GITLOG"; then
  FINAL_CONTENT="$FINAL_CONTENT \n**Daily Git log** \n \`\`\`\n$GITLOG\n\`\`\`\n"
fi
echo "$FINAL_CONTENT" | dayone2 new
 
# clean tmp file
rm $YOUR_TMPLATE_FILE_NAME
```

注意这里列出来的两个脚本中均需要你自己替换 `$YOUR_TMPLATE_FILE_NAME` 为你自己设定的模板文件名哦，不然 vim 不会加载模板的~

然后就 OK 惹…… 来看看运行效果✧(≖ ◡ ≖✿)：

dr 是我的 alias，脚本一开始会让你输入今天的心情…………

![](http://ojanerta1.bkt.clouddn.com/2017-11-16-%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-11-15%2021.03.53.png)

然后会进入 Vim 编辑一下………………

![](http://ojanerta1.bkt.clouddn.com/2017-11-16-%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-11-15%2021.05.29.png)

保存，然后日报就进 Dayone 啦：

![](http://ojanerta1.bkt.clouddn.com/2017-11-16-image2017-11-15%2021_8_50.png)

嗯！希望能治好我总忘写日报的病吧………………

科科



