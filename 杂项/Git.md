# 基础操作

## 初始化
``` bash
git init                 # 将目录变成 git 可以管理的仓库
git add xxx.xxx          # 将文件 xxx.xxx 放在目录下后，执行告诉git将文件添加到仓库中
git rm xxx.xxx           # 删除文件在 git 里的索引  [--cached] 不删除本地文件 [-f] 强制删除已修改文件
git commit -m "message"  # 将目录提交到仓库

git config --global user.name "Your Name"   # --global 设置为全局属性
git config --global user.email "email@example.com"
`````
## 提交信息查看
```bash
git log                   # 查看提交历史记录  HEAD 是一个指针指向当前版本
git log --pretty=oneline  # 简化显示

git reflog                # 查看历史命令
```
## 撤回提交
git 有工作区和版本库 编辑文件在工作区 `add`后到暂存区 `push`后到版本库
```bash
git reset ID              # 回退提交版本 默认是 soft
git reset --hard ID       # 回退提交版本，工作区文件也回退
git push origin HEAD --force  # 强制远程仓库 commit 于本地一致
```

> 回退版本有四个模式 （常用 `Soft`、`Hard`）
> `soft` 回退到上个版本的未提交状态 `mixed` 回退到上个版本已添加但未提交的状态。
> `hard` 回退到上个版本的已提交状态  `keep` ~~未知~~

> `HEAD`表示当前版本，`HEAD^`表示上个版本
## 修改提交信息
```bash
git rebase -i ID  # 打开一个交互式的 commit log 操作台(vim)
```

> `reword | r` 修改提交信息
> `squash | s` 合并提交信息(不可以只合并一条)

## 分支管理
```bash
git branch                  # 查看当前分支
~  -d branch_name           # 删除分支
~  -m ord_name new_name     # 修改分支名字

git checkout branch_name    # 切换分支
git checkout -b branch_name # 创建一个新分支
```
## 分支合并
```bash
git merge branch_name   # 将当前分支与其他分支合并 默认:fast forward 
~  --no-ff branch_name  # 使用 no-ff 模式合并

git config merge.ff no  # 将合并模式修改为默认 no-ff
```

> `ff` 快进式合并，看不出分支结构
> `no-ff` 会在合并时建立合并节点

## 合并冲突
当分支合并有冲突时，`git`会直接修改本地文件将当前分支`HEAD`与选择合并分支的冲突显示出来，可以借助图形化软件修改或直接在文件上修改

## idea 操作
`idea`修改提交信息、合并提交...都是在未`push`前操作

`idea`将新增文件从暂存区删除， `reset --mixed` 到前一个`commit`

`idea`提交代码时`force push`不能选择是当前分支处于保护分支(`git` ->`Protected Branches`)上
![[idea的保护分支.png]]
# 添加远程仓库(github)

使用 `SSH KEY`
1. 生成`SSH Key` `ssh-keygen -t rsa -C "email"` 
2. 将生成的公钥`id_rsa.pub`添加到`github`中
3. 使用`ssh`与`github`链接 `ssh -T git@github.com` 或 `git clone` 

使用 `TOKEN`
1. 登录`github`选择权限生成`token`
2. 保存`token`在每次操作远程仓库时使用 用户名+Token 登陆

切换仓库的目标地址
```bash
从ssh切换至https 
git remote set-url origin(远程仓库名称) https://email/username/ProjectName.git 

从https切换至ssh 
git remote set-url origin git@email:username/ProjectName.git 
```

## git 乱码
文件路径不正常 `git config --global core.quotepath false`

提交信息中文乱码(windows)
1. 修改`git bash`配置；`options` -> `Text` -> `Local` -> `zh_CN`
2. 修改`windows`系统变量；控制面板 -> 区域 -> 管理 -> 更改系统区域设置 -> 使用 `UTF-8` 提供全球语言支持

