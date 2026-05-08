# Git & GitHub 
## 常用建立仓库流程
```bash
#1. 在github上建立仓库，并复制SSH地址

#2. 在需上传的文件夹处打开终端

#3. 初始化
git init

#4. 与github仓库建立联系
git remote add origin +SSH地址

#5.新建本地main仓库,对应github上面的main分支
git branch main 

#6.切换到main分支（原来是master分支）
git switch main

#7.将文件加入缓冲区
git add .

#8.提交文件
git commit -m"...."

#9.上传值github（初次提交使用 
git push origin main 
```

## 一、常用 Git 命令
```bash
运行
# 查看当前文件修改状态（红=未暂存，绿=已暂存）
git status
bash
运行
# 添加所有修改到暂存区（最常用）
git add .

# 只添加某个文件到暂存区
git add 文件名
bash
运行
# 提交修改（必须写说明）
git commit -m "本次修改内容：例如新增文件、修复bug、整理代码"
bash
运行
# 第一次推送到 GitHub（必须加 -u origin 分支名）
git push -u origin main

# 以后推送（因为 -u 已经绑定过，直接简写）
git push
bash
运行
# 完整写法（拉取指定远程+指定分支）
git pull origin main

# 简写（已经绑定过，直接拉取当前分支的云端最新内容）
git pull  
```

## 二、分支
### 1.分支建立切换
```bash
#查看分支
git branch 

# 从当前分支新建 dev 分支
git branch dev  

# 切换到 main 主分支
git checkout main

# 切换到 dev 开发分支
git checkout dev
```
### 2.分支合并
#### 方式1：本地命令合并
```bash
# 1. 切回 main 主分支
git checkout main

# 2. 合并 dev 分支到 main
git merge dev

# 3. 推送到 GitHub
git push origin main
```

#### 方式2：GitHub 网页端合并
1. 进入仓库 → 点击 `Compare & pull request` 按钮
2. 设置合并方向：`base: main` ← `compare: dev`
3. 填写 PR 标题，点击 `Create pull request`
4. 确认可合并后，点击 `Merge pull request` → `Confirm merge`



### 3.分支删除
- 本地分支：必须先切换到其他分支，才能删除当前分支
```bash
# 先切到 main，再删除 dev
git checkout main
git branch -d dev
```
- GitHub 分支：合并 PR 后，可点击 `Delete branch` 清理

### 4. 版本回退
**核心原则：先备份，再回退**
```bash
# 1. 新建备份分支（避免数据丢失）
git branch backup-main

# 2. 查看提交历史，找到目标版本号
git log --oneline

# 3. 回退到指定版本
git checkout main
git reset --hard 目标版本号
git push -f origin main (强制推到云端)
```



## 四、项目git初始化
### 方式1：本地全新初始化
```bash
# 新建文件夹并进入后执行
git init
git remote add origin +地址
```

### 方式2：克隆远程仓库（自动初始化）
```bash
# 直接克隆，自动完成初始化 + 关联远程仓库
git clone 仓库地址
```

## 五、SSH 配置常用命令
```bash
# 1. 生成 SSH 密钥
ssh-keygen -t ed25519 -C "你的GitHub邮箱"

# 2. 查看公钥
复制返回的文件路径后，复制到github上，新增ssh密钥  

# 3. 测试 SSH 连接
ssh -T git@github.com
```
## 六、Git 初始配置）
```bash
# 设置用户名
git config --global user.name "你的名字"

# 设置邮箱
git config --global user.email "你的邮箱"
```

