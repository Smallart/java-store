# 账号管理与ACL权限设定

Linux是如何辨别每一个使用者的？虽然我们账号与密码登录Linux，但是Linux并不会直接认识登录者的账号名称，他仅认识ID，而ID与账号的对应就是在`/etc/passwd`中。也就是说每个登入的用户至少都会取得两个ID。一个是用户ID（UID）、一个是群组ID（GID）。档案通过UID与GID来判别自己的拥有者和群组。每一个文件都会有所谓的所有者ID与拥有组ID，当我们要显示文件属性的需要时，系统会依据`/etc/passwd`与`/etc/group`的内容，找到UID/GID对应的账号与群组名称再显示出来。

用户登录过程：
* 先找寻 /etc/passwd里面是否有你输入的账号？如果没有则跳出，如果有的话则将该账号对应的UID与GID读出来，另外，该账号的家目录与Shell设定也一并读出。
* 再就是核对密码表，此时Linux会进入/etc/shadow里面查找对应的账号与UID，然后核对下你刚刚输入的密码与里头的密码是否相符
* 如果一切都OK，就进入Shell控管的阶段

也就是说，更用户账号管理有关的有两个非常重要的档案，一个是管理用户UID/GID的重要参数的/etc/passwd，一个则是专门管理密码相关资料的/etc/shadow。

* /etc/passwd 每一行都代表一个账号，有几行就代表有几个账号在你的系统中！不过需要特别留意的，里头很多账号本来就是系统正常运行所必须的，我们可以简称他为系统账号，例如bin,daemon,adn,nobody等等

```shell

head -n 4 /etc/passwd
root[账号名]:x[密码]:0[UID]:0[GID]:root[用户信息说明栏]:/root[主目录]:/bin/bash[Shell]

```

* 密码现在放在了 /etc/shodow 下
* UID：用户表示符
    * 0 表示系统管理员
    * 1-999 系统账号（系统上启动的网络服务或背景服务希望使用较小的权限去运作，因此不希望使用root的身份去执行这些服务，所以我们就得提供这些运行中程序的所有者账号才行，这些账号通常是不可登入的）
    * 1000-60000 给一般用户使用

* GID：与/etc/group有关


```shell

head -n 4 /etc/shadow
root[账号名称]:$6$wtbCCce/PxMeE5wm$KE2IfSJr.YLP7Rcai6oa/T7KFhO...[密码]:16559[最近更动密码的日期]:0[密码不可被更动的天数]:99999[密码需要重新更改的天数]:7[密码需要变更期限前的警告天数]:[密码过期后的账号宽限时间]:[账号失效日期]:[保留]

```

## 关于群组：有效与初始群组、groups，newgrp

与GID相关的两个文件`/etc/group`与`/etc/gshadow`。

```shell

head -n 1 /etc/group
root[组名]:x[组密码]:0[GID]:[此群组支持的账号名]

```

###　有效组与初始组

＊　每个用户在他的`/etc/passwd`里面的第四栏有个所谓的GID，那个GID就是所谓的初始群组。也就是当用户以登入系统，立刻就拥有了这个群组的相关全新。

对于非initial group的其他群组就不同了，我将dmtsai加入users这个群组中，由于users这个群组并非是dmtsai的初始群组，所以，我必须在/etc/group这个档案中，找到users那一行，并将dmtsai这个账号加入第四栏，这样dmtsai才能加入users这个群组。

但是当我要创建一个新的文件或者是新的目录，那么新文件的群组是什么？这就需要检查下当时的有效群组。

当登入某一个账号之后，如何知道我所支持的群组
```shell

groups
dmtsai wheel users

```

在输出的信息中，可知道此用户同时属于 dmtsai,wheel及users这三个组，而且，第一个输出的组即为有效组（effective group）。当创建一个新的文件，则该档案的群组便是dmtsai。

有效群组的切换 newgrp，newgrp只能切换的群组必须是你已经有支持的群组。

如何让一个账号加入不同的群组？加入一个群组有连个方式。一个通过系统管理员（root）利用usermod帮你加入，二、通过群组管理员以gpasswd帮你加入其所管理的群组。

newgrp底层与`/etc/gshadow`

```shell

head -n 1 /etc/gshadow
root[组名称]:[密码]:[组管理其的账号]:[加入该群组支持的所属账号]

```

以系统管理员的角度来说，这个gshadow最大功能就是建立群组管理员。群组管理员可以将某个账号加入自己管理的组中。不过由于目前类似sudo子类的工具，就很少使用这个东西了。

## 账号管理

* 新增与删除用户：useradd
* 相关配置文件 passwd,usermod,userdel

```shell

useradd xxx

# 建立一个已经存在的群组作为用户的初始群组，因为群组已经存在，所以在 /etc/group 里面就不会主动的建立与账号同名的群组。
useradd -u 1500 -g users vbird2
```

其中默认设定：
* 在 /etc/passwd 里面建立一行于账号相关的资料，包括建立UID/GID/家目录
* 在/etc/shadow 里面讲此账号的密码相关参数填入，但是尚未有密码
* 在/etc/group 里面加入一个与账号名称一摸一样的群组名称
* 在 /home 底下建立一个与账号同名的目录作为用户家目录，且权限为700
