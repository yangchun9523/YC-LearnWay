## Git学习
### 1.获取Git仓库

- 初始化git仓库: 

```shell
# 初始化git命令：
# 会在当前目录下生成.git子目录，存储git版本控制的所有数据。(与.svn相似)
git init
```

- 克隆已有git仓库: 

```shell
# SSH方式：
# 需要生成git公钥，并添加到github SSH key中
# 参考URL:https://git-scm.com/book/zh/v2/服务器上的-Git-生成-SSH-公钥
git clone git@github.com:yangchun9523/YC-LearnWay.git
# HTTPS方式：
git clone https://github.com/yangchun9523/YC-LearnWay.git
```
### 2.git忽略文件.gitignore配置

```text
# MacOS
.DS_Store

# Compiled class file
*.class

# Log file
*.log

# BlueJ files
*.ctxt

# Mobile Tools for Java (J2ME)
.mtj.tmp/

# Package Files #
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

# virtual machine crash logs, see http://www.java.com/en/download/help/error_hotspot.xml
hs_err_pid*
```

### 3.目前使用到的Git命令总结(持续补充)

```shell
# 0.将本地仓库关联到GitHub仓库上
git remote add origin git@github.com:yangchun9523/YC-LearnWay.git

# 1.创建文件
touch test.md

# 2.将文件添加到暂存区
git add test.md

# 3.数据添加到仓库
git commit -m'描述信息'

# 4.将GitHub仓库更新到本地
git pull origin master
	# 4.1 git fetch:相当于是从远程获取最新版本到本地，不会自动合并
		git fetch origin master:tmp
	  	git diff tmp //比较两次修改的差异
		git merge tmp
	# 4.2 git pull:相当于是从远程获取最新版本并merge到本地
		git pull origin master
		
# 5.将本地仓库代码提交到远程仓库
git push origin master

# 6.向文件中写内容
vim test.md

# 7.删除文件
rm -rf test.java

# 8.将文件从暂存区中删除
git rm test.java

# 9.将文件从仓库中删除
git commit -m'描述消息'

```

### 4. github/gitlab管理多个ssh key
以前只使用一个 ssh key 在github上提交代码，由于工作原因，需要再添加一个ssh key在公司的 gitlab上提交代码，下面记录下配置过程，防止遗忘。

#### 4.1 生成第一个SSH KEY
```shell
	# 直接回车 <默认生成 id_ras 文件>
	# 命令解析 -t 是类型, -C 是邮箱, -b 是字节
	ssh-keygen -t rsa -C "your.email@example.com" -b 4096
```
#### 4.2 生成第二个SSH KEY
```shell
	# 不能直接回车，需要给文件重新起一个名字
	ssh-keygen -t rsa -C "your.email@yourcompany.com" -b 4096
```
#### 4.3 添加私钥
```shell
$ cd ~/.ssh
$ ls -l
-rw-r--r--  1 yangchun  staff   268  5 17 15:18 config
-rw-------  1 yangchun  staff  1679  4 22 19:06 id_rsa
-rw-r--r--  1 yangchun  staff   406  4 22 19:06 id_rsa.pub
-rw-------  1 yangchun  staff  3243  5 17 15:07 iflytek_rsa
-rw-r--r--  1 yangchun  staff   747  5 17 15:07 iflytek_rsa.pub
# 查看私钥
$ ssh-add ~/.ssh/id_rsa
$ ssh-add ~/.ssh/iflytek_rsa
# 验证
$ ssh-add -l
#---- end ---- 
# 可以通过 ssh-add -D 来清空私钥列表
$ ssh-add -D
```
#### 4.4 修改配置文件
```shell
$ touch config

# 添加一下内容

# github
    Host github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa

# gitlab
    Host pl.git.iflytek.com
    HostName pl.git.iflytek.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/iflytek_rsa
```

#### 4.5 测试验证
```shell
$ ssh -T git@pl.git.iflytek.com
Welcome to GitLab, chunyang2!
$ ssh -T git@github.com
Hi yangchun9523! You've successfully authenticated, but GitHub does not provide shell access.
```


