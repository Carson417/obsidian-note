
- Git是一个开源的**分布式版本控制系统**

# git安装

```
git --version
```

- config设置

```
$ git config --global user.name "name"
$ git config --global user.email "email"

// 查询全部
$ git config --list
```


# git与github建立连接
## 本地Git仓库

- `git init`   --初始化成一个git可管理的仓库
	- `.git` 文件夹是git用来跟踪和管理版本库的
- `git status`
	- 红色表示未add到git仓库上的文件
	- 绿色表示已add到git仓库上的文件
- `git add` 
	- 把代码添加到**本地仓库暂存区**
	- `git add .`  把该项目下所有的文件添加到仓库
- `git commit` 
	- 把项目提交到仓库
	- `git commit -m "注释"`

## 通过ssh-key远程连接

- 本地Git仓库和GitHub仓库之间的传输是**通过SSH加密传输的**，所以需要配置ssh key

> [!SSH key]
    > 用户主目录下，查询是否有.ssh文件 
    > 有的话，看文件下有没有id_rsa和id_rsa.pub这两个文件
    > 有就不用创建了

- 创建ssh key：`ssh-keygen -t rsa -C "注册的邮箱"`
	- `-t rsa` type，指定生成密钥算法为rsa
	- `-C` comment
- **d_rsa是私钥，不能泄露；id_rsa.pub是公钥，可以公开**

- 验证连接 `$ ssh -T git@github.com`

- 新建一个远程仓库， `git remote add origin https://github.com/ybli/Elegent.git`
	- 将位于 GitHub 上 ybli 用户的 Elegent 仓库添加为当前本地 Git 仓库的远程仓库，并命名为 "origin"
		- `git remote add`- Git 命令，用于添加一个新的远程仓库引用
		- `origin`- 这是为远程仓库指定的默认名称（可自定义，但 origin 是约定俗成的名称）
		- `https://github.com/ybli/Elegent.git`- 远程仓库的 URL 地址

- 推送：`git push (-u) origin master`，把本地仓库的代码推送到远程仓库Github上

> [!新建远程仓库的时候勾选Initialize this repository with a README，推送时可能会报failed to push some refs to https://github.com/ybli/Elegent.git的错]
    > 这是由于你新创建的那个仓库里面的README文件不在本地仓库目录中，这时可以同步内容
    > `$ git pull --rebase origin master`
    > 之后再进行`git push origin master`就能成功了

    

