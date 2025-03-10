---
outline: deep
lastUpdated: 2025-01-18T20:25:00+08:00
prev: false
next: false
---

# 命令脚本

这是本服在0.2.3更新后新增的特有功能，用来让玩家能够执行一连串命令，就像shell脚本那样，并且支持在离线后仍然能继续执行。

当玩家在线时，脚本输出的信息直接输出到玩家的对话框上。当玩家离线后，脚本输出的信息被临时存储起来，当玩家重新进入时将全部输出这些信息。不过服务器重启后这些存储的信息会丢失。

## 脚本元信息

一个脚本含有以下元信息：
1. 脚本名称，这个同一时刻在服务器内只能存在唯一的脚本名称，即同一名称的脚本不能同时执行。
2. 执行者，为执行脚本的玩家的展示名字。
3. 启动时间，遵循[ISO 8601 时间格式](https://www.iso.org/iso-8601-date-and-time-format.html)。
4. 阶段，由脚本的内容定义，表示脚本目前执行到了哪个阶段，默认为 **Start**。

## 命令格式

本来打算以 `script` 作为命令前缀，但考虑到和地毯模组的脚本也用了这个，就换了 `run`。
- `/run do` 当玩家手持[成书](https://zh.minecraft.wiki/w/%E6%88%90%E4%B9%A6)时，输入该指令后将启动脚本，自动执行成书里的内容。脚本名称为书名。
- `/run do <名称>` 当玩家手持[书与笔](https://zh.minecraft.wiki/w/%E4%B9%A6%E4%B8%8E%E7%AC%94)时，输入该指令后将启动脚本，自动执行书与笔中的内容，脚本名称为命令中的名称参数。
- `/run list` 列出当前服务器执行的所有脚本，按照[脚本元信息](#脚本元信息)的格式打印。
- `/run cancel <名称>` 取消正在执行的对应名称的脚本。

::: tip
成书或书与笔中，多页内容会被自动视为一整页，按顺序拼接成许多行。
:::

::: warning :warning: 注意
脚本取消后，当前正在执行的任务并不会被取消，只是脚本里接下来的命令将不再执行。比如你若在脚本内召唤了假人，他们不会因为你取消脚本而停止当前执行的动作并下线。
:::

## 脚本格式
### 脚本元素

首先你需要知道，脚本是一行一行读取的。些元素占据至少完整的一行，不能一行多个元素。此外任何语句的前后空格将会被自动忽略。

- **空行** 会被忽略。
- **命令** 即普通的游戏命令。脚本执行这些命令，和玩家手动输入这些命令的效果是完全一样的，可以省略斜杠。比如 `/me 你好`, `player Steve spawn`。
- **等待** 元命令，格式为 `@wait <时间>`，即让脚本在此处暂停一段时间。时间参数的格式为 *数字 + 单位*，其中数字是正整数，单位为 `t/s/m/h/d`，分别对于1游戏刻/秒/分/时/天（不是严格意义对于现实的时间，事实上后面四个单位会被按比例换算为游戏刻）。比如 `@wait 3s` 表示暂停60游戏刻。
- **循环** 元命令，格式为 `@for <次数>`，从此处开始一直到循环终止符中间的这段内容将会循环执行，次数为正整数。注意循环不能嵌套。
- **循环终止** 元命令，格式为 `@end`，标记循环终止的位置。
- **阶段** 标记，格式为 `#<阶段名>`。每当脚本执行这些位置，就会将相应地修改当前阶段，脚本的阶段可以用 `/run list` 命令查看（灰色字体部分），这样可以清晰的看到当前脚本执行到什么地步了。

::: details 一个简单的例子
```
#开始
/me 生成假人
/player Steve spawn at 1 2 3
@wait 10t

#假人执行动作
@for 3
    player Steve jump continous
    @wait 1m
    player Steve move forward
    @wait 5s
    player Steve stop
    @wait 5t
@end

me 结束 
```
:::

### 玩家名展开

在解析上述元素之前，服务器首先会自动展开脚本中涉及的玩家名变量，用以方便给各个玩家使用而避免冲突。

具体地说，命令中用 `$<数字>` 标记的地方都会被自动展开为玩家名。其中 `$0` 表示脚本执行者的名字，`$n` (n是1~9的整数) 会被自动展开成服务器随机一个空闲假人的名称，整个脚本中同一数字对应的假人名是不变的。不过名称是在一开始脚本启动的时候就全部展开的，这意味着如果刚启动时 Steve 是空闲的假人，而过了一分钟后他被其他人召唤了，对于当前脚本来说 Steve 还是空闲的，此时脚本再召唤 Steve 就会失败。

举个例子，假如 Alex 是目前服里空闲的假人，那么 `/player $1 spawn` 在执行过程中就会变成 `/player Alex spawn`。

如果你的命令内确实需要 `$` 字符而不想让其被展开为玩家名，请使用 `\$`替代。

### 命令树

它可以让命令拼接在一起，方便编写。注意它只处理纯粹的命令，不包括元命令和阶段标记。

以左花括号 `{` 结尾的命令被视为不完整的命令，后面的命令会被自动拼接到前面的命令上。没有这个结尾的命令将被视为最后一段子句。右花括号 `}` 单独成一行用来标记一个层级的闭合。可以嵌套。听起来可能很抽象，我们看一段例子。

```
player {
    Steve {
        spawn at 1 2 3
        @wait 1s
        move forword
    }
    Alex {
        spawn at 4 5 6
        @wait 1s
        move backword
    }
    @wait 1m
    Steve kill
    Alex kill
}
```

将会被解析为

```
player Steve spawn at 1 2 3
@wait 1s
player Steve move forword
player Alex spawn at 4 5 6
@wait 1s
player Alex move backword
@wait 1m
player Steve kill
player Alex kill
```

::: tip
- 因为语句首尾部分的空格会被自动省掉，因此上面的例子中 `{` 符号左边的空格不能省掉，否则会被认成 `playerStevespawn at 4 5 6`云云。
- 除了 `player` 语句前可以加斜杠，其它都不行。
:::

## 综合案例

```
# 一个新的开始
me 我（$0）要开始执行这个脚本了

/player {
    $1 {
        spawn at 11 22 33 facing 45.0 135.0 in minecraft:overworld
        @wait 1s
        tag "攻击"
        attack continuous
    }
    $2 {
        spawn at -1 -2 -3 facing -45.0 -135.0 in minecraft:overworld
        @wait 1s
        tag "使用"
        use interval 10
    }
    @wait 1m

# 下一阶段
@for 10
    $1 {
        stop
        move forward
        @wait 3s
        dropStack
        move backword
        @wait 3s
        stop
        attack continuous
    }
    $2 {
        stop
        move left
        @wait 3s
        dropStack
        move right
        @wait 3s
        stop
        use interval 10
    }
    @wait 1m
@end
}

# 圆满完成
player {
    $1 kill
    $2 kill
}
me bye ~
```

这个任务实际上就是召唤了两个假人，一个不断攻击一个不断右键。他们每隔一分钟左右就会移动到一个定点位置把背包里的物品全丢出来，总共执行10次。

::: warning :warning: 注意
地毯的假人在召唤时采用了一条单独的线程，召唤的命令并不会和其它命令按顺序执行，因此召唤假人后需要手动设置间隔一秒钟才能执行其它命令。
:::