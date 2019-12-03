## 1，初始化仓库并提交

下面是初始化并提交到master

```shell
git init
git add README.md
git commit -m "first commit"
git remote add origin git@github.com:b1285920626/pangPratice.git
git push -u origin master
```

## 2，SSH连接

上一步需要SSH连接的话按照如下初始化，生成`id_rsa`和`id_rsa.pub`。

```shell
$ ssh-keygen -t rsa -C "youremail@example.com"
```

当在远程库上设置了SSH 之后还是**报错连接超时**，问题如下

```
$ git push origin master
ssh: connect to host github.com port 22: Connection timed out
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```



这个时候需要检查一下SSH是否能够连接成功，输入以下命令

```shell
ssh -T git@github.com
```



如果不通过，在存放公钥私钥(id_rsa和id_rsa.pub)的文件里，新建config文本，内容如下：

```
Host github.com
User YourEmail@163.com
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
Port 443
```

其中User为登录Github的账号名称。

然后 再次执行：

```shell
ssh -T git@github.com 
```



