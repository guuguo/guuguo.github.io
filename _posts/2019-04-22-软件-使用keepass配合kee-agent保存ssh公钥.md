---
layout:     post
title:      使用keepass配合kee-agent保存ssh公钥
date:       2019-04-22
author:     guuguo
catalog:    false
tags:
    - ssh

---
# 使用keepass配合kee-agent保存ssh公钥

虽然KeePass通常用于存储凭据，但它也可用于存储SSH密钥以及在应用程序需要时*提供*这些SSH密钥。

在连接到远程服务器而不是用户名/密码时使用SSH密钥是个好主意。为每个连接的服务器生成密钥对也是一种很好的做法 - 包括执行远程`git`操作时。

但是，随着时间的推移，您需要管理和记住的密钥数量会增加。有多种方法可以解决这个问题，包括SSH `.config`文件。KeePass是一种解决方法，通过使用KeePass和KeeAgent插件，我们可以使用KeePass数据库作为我们密钥的容器，并在需要时提供服务。这具有SSH密钥与KeePass数据库同步的优点。

## 安装东西

### KeePass的

确保已安装[KeePass Professional Edition](http://keepass.info/download.html)。您可能需要考虑使用*便携*版，并将整个KeePass安装与您的`.kdbx`密码文件整个同步。例如使用onedrive，您可以在Onedrive 同步文件夹中安装KeePass，其中包括配置文件和插件文件夹。这样，您的设置和插件将跨机器运行，从而减少了所需的设置(跨机器不跨平台)。

### Git Bash

Git的Bash是不仅仅是`git`命令，因为大多数人使用它，它实际上是非常有用的和熟悉的工具，如集合`grep`，`vi`，`awk`，`cut`，但最重要的`ssh`和`scp`。查看**C:\Program Files\Git\usr\bin**以了解您可以使用的内容。

![混帐兵](https://github.com/mendhak/keepass-and-keeagent-setup/raw/master/assets/git-bin-folder.png)

在安装[Git Bash时](https://git-scm.com/downloads)，我建议使用Windows命令提示符中的Git选项，并且行结尾为“原样”。

### KeeAgent

安装[KeeAgent](http://lechnology.com/software/keeagent/#Download) - 将`KeeAgent.plgx`文件放在KeePass插件文件夹中很简单。

![plgx](https://github.com/mendhak/keepass-and-keeagent-setup/raw/master/assets/keeagent-install-plgx.png)

您需要重新打开KeePass才能显示该插件。

### 将密钥添加到远程Git帐户

SSH的一个常见用例是访问您的Github或Bitbucket帐户`ssh`而不是`http`。

![gitclone](https://github.com/mendhak/keepass-and-keeagent-setup/raw/master/assets/github-clone.png)

作为先决条件，[请将您的公钥添加](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/)到您的帐户。

![githubssh](https://github.com/mendhak/keepass-and-keeagent-setup/raw/master/assets/ssh-key-paste.png)

------

## 存放你的钥匙

继续使用Github示例，创建一个新条目来保存密钥。如果私钥上有密码，请在密码字段中输入密码。

![keeagent1](https://github.com/mendhak/keepass-and-keeagent-setup/raw/master/assets/keepass-ssh-key-1.png)

现在换钥匙。单击“ *高级”*选项卡，然后选择附加文件。

![keeagent2](https://github.com/mendhak/keepass-and-keeagent-setup/raw/master/assets/keepass-ssh-key-2.png)

找到远程服务器的SSH密钥对并附加它们

![keeagent3](https://github.com/mendhak/keepass-and-keeagent-setup/raw/master/assets/keepass-ssh-key-3.png)

### 使用KeeAgent加载密钥

单击KeeAgent选项卡。选中*允许KeeAgent使用此输入*选项。从“ *附件”*选项中，选择刚刚附加的私钥。

您应该看到“ *密钥信息”*部分填充了有关密钥的一些信息。

![keeagent4](https://github.com/mendhak/keepass-and-keeagent-setup/raw/master/assets/keepass-ssh-key-4.png)

此时，KeeAgent知道您的密钥但尚未加载密钥。要加载密钥，请重新打开KeePass数据库，或双击“ *SSH密钥状态”*列以将状态从“ *未加载”*更改为“ *已加载”*

![keeagent5](https://github.com/mendhak/keepass-and-keeagent-setup/raw/master/assets/keepass-ssh-key-5.png)

检查加载哪些键的另一种方法是使用*Tools* > *KeeAgent*

![keeagent6](https://github.com/mendhak/keepass-and-keeagent-setup/raw/master/assets/keepass-ssh-key-6.png)

### 告诉Git Bash使用KeeAgent

虽然KeeAgent现在已准备好提供密钥，但需要告知Git Bash。如果你现在打开Git Bash并尝试快速测试，你应该会收到一个错误。

> $ ssh -T [git@github.com](mailto:git@github.com)
> 权限被拒绝（publickey）。

返回KeePass，单击*工具* > *选项...*，然后单击*KeeAgent*选项卡。选择*显示通知...*，更重要的是选中*Cygwin / MSYS Integration*区域中的框。添加一个路径，如*C：\ Temp \ cyglockfile*和*C：\ Temp \ syslockfile*或任何您想要的任意文件名。这将创建套接字文件，这是一个Unix概念 - 它允许应用程序通过文件相互通信。在这种情况下，Git Bash将通过这两个套接字文件之一与KeePass进行通信。

![keeagent7](https://github.com/mendhak/keepass-and-keeagent-setup/raw/master/assets/keepass-ssh-key-7.png)

再次，关闭并重新打开KeePass，然后转到*C：\ Temp*或您指定的任何路径。你应该在那里看到你的套接字文件。

![keeagent8](https://github.com/mendhak/keepass-and-keeagent-setup/raw/master/assets/keepass-ssh-key-8.png)

使用文本编辑器，甚至`vi`在Git Bash中编辑/创建`~/.bashrc`文件。这将对应于*C：\ users \ username \ .bashrc*

```
vi ~/.bashrc
```

添加以下行 - 它将设置`SSH_AUTH_SOCK`环境变量，指向套接字文件。这就是Git Bash需要知道的。

```
export SSH_AUTH_SOCK="C:\Temp\cyglockfile"
```

关闭并重新打开Git Bash。然后再试一次。如果它有效，您应该看到来自Github的消息，以及使用密钥的通知。如果它不起作用，请再次尝试使用其他文件（syslockfile）。

![keeagent9](https://github.com/mendhak/keepass-and-keeagent-setup/raw/master/assets/keepass-ssh-key-9.png)

尝试一些`git`命令 - `git clone`（使用非http URL），`git fetch`和`git push`。在每种情况下，它都应该使用密钥并显示通知。

### 不要加载每个密钥

回到加载步骤，我们在*打开/解锁数据库*选项*时*将*Add键*留给*代理*。

![keeagent10](https://github.com/mendhak/keepass-and-keeagent-setup/raw/master/assets/keepass-ssh-key-10.png)

这告诉KeeAgent在打开KeePass数据库时加载此密钥。但是，如果您加载了大约5个或更多密钥，您的身份验证可能会失败。这是因为SSH代理通过尝试使用每个加载的密钥来工作，直到找到有效密钥。许多SSH服务器不喜欢这样，如果它看到大约5次或更多次尝试，将关闭连接。

您应该只检查上面的选项以获取频繁使用密钥，并且Git服务器密钥就是一个很好的例子。

对于偶尔使用密钥，您可以双击*SSH密钥状态*列，仅在您即将使用它时加载它们，甚至可以卸载其他一些*密钥*。

