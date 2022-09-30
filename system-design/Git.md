# Git

## Git基础命令

### 初始化

Git仓库，使用`git init`命令。

添加文件到Git仓库，分两步：

1. 使用命令`git add <file> `，注意，可反复多次使用，添加多个文件；
2. 使用命令`git commit -m <message>  `，完成。

### 版本回退

- `HEAD`指向的版本就是当前版本，因此，Git允许我们在版本的历史之间穿梭，使用命令`git reset --hard commit_id`。

- 穿梭前，用`git log`可以查看提交历史，以便确定要回退到哪个版本。

- 要重返未来，用`git reflog`查看命令历史，以便确定要回到未来的哪个版本。

  场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令`git checkout -- file`。

  场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令`git reset HEAD <file> `
  ，就回到了场景1，第二步按场景1操作。

  场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考[版本回退](https://www.liaoxuefeng.com/wiki/896043488029600/897013573512192)
  一节，不过前提是没有推送到远程库。

    - git checkout . && git clean -df 可以放弃所有修改、新增、删除文件

### 分支

- 查看分支：`git branch`
- 创建分支：`git branch <name> `
- 切换分支：`git checkout <name> `或者`git switch <name>`
- 创建+切换分支：`git checkout -b <name> `或者`git switch -c <name> `
- 合并某分支到当前分支：`git merge <name> `
- 删除分支：`git branch -d <name> `

### 标签

- 命令`git tag <tagname> `用于新建一个标签，默认为`HEAD`，也可以指定一个commit id；
- 命令`git tag -a <tagname> -m "blablabla..."`可以指定标签信息；
- 命令`git tag`可以查看所有标签。
- 命令`git push origin <tagname> `可以推送一个本地标签；
- 命令`git push origin --tags`可以推送全部未推送过的本地标签；
- 命令`git tag -d <tagname>` 可以删除一个本地标签；
- 命令`git push origin :refs/tags/<tagname>`可以删除一个远程标签

### 远程仓库

要关联一个远程库，使用命令`git remote add origin git@server-name:path/repo-name.git`；

关联后，使用命令`git push -u origin master`第一次推送master分支的所有内容；

此后，每次本地提交后，只要有必要，就可以使用命令`git push origin master`推送最新修改；

- 查看远程库信息，使用`git remote -v`；
- 本地新建的分支如果不推送到远程，对其他人就是不可见的；
- 从本地推送分支，使用`git push origin branch-name`，如果推送失败，先用`git pull`抓取远程的新提交；
- 在本地创建和远程分支对应的分支，使用`git checkout -b branch-name origin/branch-name`，本地和远程分支的名称最好一致；
- 建立本地分支和远程分支的关联，使用`git branch --set-upstream branch-name origin/branch-name`；
- 从远程抓取分支，使用`git pull`，如果有冲突，要先处理冲突。

## git服务器自动部署

> [使用git钩子自动部署服务]([https://x-front-team.github.io/2016/09/03/%E4%BD%BF%E7%94%A8git%E9%92%A9%E5%AD%90%E8%87%AA%E5%8A%A8%E9%83%A8%E7%BD%B2%E6%9C%8D%E5%8A%A1/](https://x-front-team.github.io/2016/09/03/使用git钩子自动部署服务/))
>
> [Git 实现 Laravel 项目的自动化部署](https://learnku.com/articles/33689)
>
> [WebHook](https://learnku.com/articles/34320)

### 裸仓库方式

这也是比较推荐的方式，不太有副作用，尤其需要多人共享时。
那什么是裸仓库呢？
就是没有工作目录的仓库, 不能执行有关git的任何命令，只记录历史提交信息，只能通过clone它的仓库对它进行push操作对它进行更新。裸仓库一般用来做多人代码交换的中转站，它只有这么些目录：
`branches config description HEAD hooks info objects refs`
接下来，仅以一个简单的node服务为例，演示整个过程。
ps: 本地和服务器都要安装git和node，否则没法玩~

#### 1.本地创建仓库并开发

终端cd到你喜欢的一个新目录(这里以githook-test为例)，然后：

```shell
git init
```

创建index.js,使用你喜欢的编辑器，输入以下代码(摘自node官网，改了下host)：

```js
const http = require('http');

const hostname = '0.0.0.0';
const port = 3000;

const server = http.createServer((req, res) => {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'text/plain');
    res.end('Hello World\n');
});

server.listen(port, hostname, () => {
    console.log(`Server running at http://${hostname}:${port}/`);
});
```

本地跑一下试试看：

```shell
node index.js
```

然后用浏览器访问`http://locaohost:3000/`,看到了`Hello World`。
嗯，很好。
提交代码：

```shell
git add .
git commit -m 'init'
```

#### 2.创建远程裸仓库

ssh登陆到服务器，cd到你喜欢的目录(比如`~/git_projects`),创建一个新仓库目录：

```shell
mkdir githook-test.git && cd $_
```

裸仓库一般都命名为xxx.git,然后初始化git：

```shell
git init --bare
```

注意，这里的bare参数很重要，就是创建裸仓库的意思

#### 3.推送代码到服务器

配置本地仓库的remote，比如你的服务器host为vps.xx.com,登陆用户为git，刚刚创建的远程仓库路径为~
/git_projects/githook-test.git，那么就在本地仓库执行：

```shell
git remote add origin git@vps.xx.com:~/git_projects/githook-test.git
```

最好配置服务器免密登陆，把你的公钥塞到服务器的~/.ssh/authorized_keys文件中即可。
第一次推送：

```shell
git push -u origin master
```

如果提示信息的最后两行是：

```shell
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
```

就说明推送成功了。

#### 4.创建远程非裸仓库

由于裸仓库没有工作目录，没法执行真正的代码，所以需要另一个目录(非裸仓库)
拉取实际代码，通过clone的方式拉取即可，cd到你喜欢的目录，比如`~/www`,执行：

```shell
git clone ~/git_projects/githook-test.git
```

#### 5.在服务器上启动服务

由于服务器的服务进程需要在后台执行，我们借助[pm2](http://pm2.keymetrics.io/)这个神器帮我们管理node服务.首先全局安装pm2：

```shell
npm install pm2 -g
```

然后cd到真正运行服务的仓库:

```shell
cd ~/www/githook-test
```

启动服务：

```shell
pm2 start index.js --name githook-test
```

name参数用来给当前的服务起一个名字标识，用于后续给指定的服务执行操作，比如重启，停止等.
打开浏览器，访问这个服务，比如，你的服务器host是vps.xx.com，那么就访问`http://vps.xx.com:3000`.
如果也看到了`Hello World`，就说明服务器上的服务已经跑起来了~

#### 6.配置git钩子

*这里才是重点~*
git钩子可以理解为一个触发器，当仓库发生指定事件，可以触发执行指定的shell脚本。我们这里需要的是post-receive钩子。
在服务器的仓库中创建钩子文件：

```shell
vi ~/git_projects/githook-test.git/hooks/post-receive
```

粘贴如下脚本：

```shell
#!/bin/sh
unset GIT_DIR
cd ~/www/githook-test
echo 'Pulling code:'
git pull origin master
echo 'Restarting server:'
pm2 restart githook-test
echo 'Deployment Done'
exit 0
```

保存并退出
其中，`unset GIT_DIR`
是必须的，和Makefile一样，在每一行，目录都是固定的，需要解除环境变量GIT_DIR环境变量才能自由移动当前位置，这东西坑了我好一会儿~
感谢https://argcv.com/articles/2078.c 这篇文章~
现在这个钩子只是个普通文件，需要给它加上可执行权限：

```shell
chmod +x ~/git_projects/githook-test.git/hooks/post-receive
```

#### 7.测试自动部署

修改本地的index.js，比如把`res.end('Hello World\n');`这句改成`res.end('Hello World. I Love Git!\n');`
然后提交，并推送：

```shell
git commit -a -m 'edit' && git push
```

你会在本地终端看到类似如下的信息：

```shell
Counting objects: 3, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 269 bytes | 0 bytes/s, done.
Total 3 (delta 1), reused 0 (delta 0)
remote: Pulling code:
remote: From <和谐内容>/git_projects/githook-test
remote:  * branch            master     -> FETCH_HEAD
remote:    cd2b660..50118a9  master     -> origin/master
remote: Updating cd2b660..50118a9
remote: Fast-forward
remote:  index.js | 2 +-
remote:  1 file changed, 1 insertion(+), 1 deletion(-)
remote: Restarting server:
remote: [PM2] Applying action restartProcessId on app [githook-test](ids: 1)
remote: [PM2] [githook-test](1) ✓
remote: ┌──────────────┬────┬──────┬───────┬────────┬─────────┬────────┬─────────────┬──────────┐
remote: │ App name     │ id │ mode │ pid   │ status │ restart │ uptime │ memory      │ watching │
remote: ├──────────────┼────┼──────┼───────┼────────┼─────────┼────────┼─────────────┼──────────┤
remote: │ githook-test │ 1  │ fork │ 17297 │ online │ 4       │ 0s     │ 11.789 MB   │ disabled │
remote: └──────────────┴────┴──────┴───────┴────────┴─────────┴────────┴─────────────┴──────────┘
remote:  Use `pm2 show <id|name>` to get more details about an app
remote: Deployment Done
To <和谐内容>@<和谐内容>:~/git_projects/githook-test.git
   cd2b660..50118a9  master -> master
```

然后，再次打开浏览器访问服务器上的服务，如果能看到`Hello World. I Love Git!` 说明自动部署已经成功了！

### 非裸仓库方式

这种方式，不需要创建裸仓库，只需创建非裸仓库，感觉会节省点磁盘空间~
具体操与裸仓库方式大体一致：
裸仓库方式中的 `2.创建远程裸仓库`步骤，更改为:

```shell
cd ~/www && mkdir githook-test && cd &_ && git init
```

由于默认情况下，git不允许push代码到非裸仓库，我们需要对这个非裸仓库进行一些设置：

```shell
git config receive.denyCurrentBranch ignore
```

`3.推送代码到服务器`步骤中，由于不存在裸仓库了，我们直接push代码到非裸仓库，我们本地仓库的远程仓库地址也要修改一下：

```shell
git remote add origin git@vps.xx.com:~/www/githook-test
```

如果远程仓库不执行`git config receive.denyCurrentBranch ignore`的话，直接推送会看到如下错误信息：

```shell
remote: error: refusing to update checked out branch: refs/heads/master
remote: error: By default, updating the current branch in a non-bare repository
remote: error: is denied, because it will make the index and work tree inconsistent
remote: error: with what you pushed, and will require 'git reset --hard' to match
remote: error: the work tree to HEAD.
remote: error:
remote: error: You can set 'receive.denyCurrentBranch' configuration variable to
remote: error: 'ignore' or 'warn' in the remote repository to allow pushing into
remote: error: its current branch; however, this is not recommended unless you
remote: error: arranged to update its work tree to match what you pushed in some
remote: error: other way.
remote: error:
remote: error: To squelch this message and still keep the default behaviour, set
remote: error: 'receive.denyCurrentBranch' configuration variable to 'refuse'.
```

跳过`4.创建远程非裸仓库` 步骤.
即使可以向远程非裸仓库push，远程仓库的工作目录也不会随之改变，需要通过钩子，将工作目录更新，所以`6.配置git钩子`
步骤中，钩子的脚本也要改一下：

```shell
#!/bin/sh
unset GIT_DIR
cd ~/www/githook-test
echo 'Checkout working copy:'
git checkout -f
echo 'Restarting server:'
pm2 restart githook-test
echo 'Deployment Done'
exit 0
```

再次测试，发现也可以自动部署啦！
从git的默认行为来看，非裸仓库的方式，并不推荐，尤其是生产环境，一定要慎用。
在一些不重要的小应用里使用还是不错的~

### WebHook

1、仓库地址使用 ssh 例如：git@gitee.com:xxxxx/project.git

2、在 gitee 或者 github 上的对应的项目添加你的部署公钥

3、在 gitee 或者 github 上的对应的项目添加 WebHook 对应的 url 和访问秘钥

```php
<?php

//访问秘钥
$keySecret = '^!@KVzPZ^DaRNA7R53353%7aEAfsfdOptH7b%0i5';

//需要更新的项目目录
$wwwRoot = [
    '/www/wwwroot/test1',
    '/www/wwwroot/test2',
];

//保存运行脚本的日志
$logFile = 'log/layuimini-githook.log';

//git执行命令
$gitCommand = 'git pull';

// 判断是否开启秘钥认证(已实现gitee和github)
if (isset($keySecret) && !empty($keySecret)) {
    list($headers, $gitType) = [[], null];
    foreach ($_SERVER as $key => $value) {
        'HTTP_' == substr($key, 0, 5) && $headers[str_replace('_', '-', substr($key, 5))] = $value;
        if (empty($gitType) && strpos($key, 'GITEE') !== false) {
            $gitType = 'GITEE';
        }
        if (empty($gitType) && strpos($key, 'GITHUB') !== false) {
            $gitType = 'GITHUB';
        }
    }
    if ($gitType == 'GITEE') {
        if (!isset($headers['X-GITEE-TOKEN']) || $headers['X-GITEE-TOKEN'] != $keySecret) {
            die('GITEE - 请求失败，秘钥有误');
        }
    } elseif ($gitType == 'GITHUB') {
        $json_content = file_get_contents('php://input');
        $signature = "sha1=" . hash_hmac('sha1', $json_content, $keySecret);
        if ($signature != $headers['X-HUB-SIGNATURE']) {
            die('GITHUB - 请求失败，秘钥有误');
        }
    } else {
        die('请求错误，未知git类型');
    }
}

!is_array($wwwRoot) && $wwwRoot = [$wwwRoot];
foreach ($wwwRoot as $vo) {
    $shell = sprintf("cd %s && git pull 2>&1", $vo);
    $output = shell_exec($shell);
    $log = sprintf("[%s] %s \n", date('Y-m-d H:i:s', time()) . ' - ' . $vo, $output);
    echo $log;
    file_put_contents($logFile, $log, FILE_APPEND);
}
```

