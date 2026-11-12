# Git 实操学习笔记：从 clone 到 push、pull、diff


整理日期：2026 年 11 月 12 日。根据本次对话整理，保留实际命令、操作结果、概念问答和排错过程。

示例远程仓库：[shuaiqideyange/hello-agents](https://github.com/shuaiqideyange/hello-agents)

本地仓库目录：`/Users/sqdyg/Documents/zhaixiaoyan/agent学习/gitagent`

练习文件：`git-learning-notes.md`

> **阅读说明**：重要命令和知识使用 HTML 红色标记。Markdown 本身没有统一的文字颜色语法，颜色是否显示取决于阅读器；GitHub 可能过滤这些颜色样式。配套的 HTML 阅读版可在浏览器中显示红色。命令框里的命令可以复制执行，但不要将整份笔记从头批量执行。

## 1. 本次学习走过的路线

```text
clone：把远程仓库及版本历史复制到本地
   ↓
修改文件 → status / diff：检查状态和修改
   ↓
add：选出下一次要提交的修改
   ↓
commit：保存一个本地版本
   ↓
push：把本地提交上传到 GitHub

GitHub 网页修改并提交
   ↓
pull：获取远程更新，并整合到当前本地分支
```

<span style="color:#c62828"><strong>修改文件、add、commit、push 是不同步骤。commit 不会自动上传，push 才会把本地提交上传到远程。</strong></span>

## 2. 本地仓库、工作区和远程仓库

### 2.1 gitagent 是不是本地仓库？

克隆成功后，`gitagent` 就是项目的本地 Git 仓库目录。里面通常有两部分：

```text
gitagent/
├── .git/                    ← 版本历史、分支、远程地址等管理信息
├── git-learning-notes.md     ← 可以直接编辑的项目文件
└── 其他项目文件
```

| 概念 | 含义 |
| --- | --- |
| 工作区 | 你能直接查看和编辑的项目文件 |
| 暂存区 | 下一次提交准备采用的内容快照 |
| 本地仓库 | 在电脑上保存版本历史；核心数据位于隐藏目录 `.git` 中 |
| 远程仓库 | GitHub 上保存和共享提交的仓库 |

<span style="color:#c62828"><strong>本地仓库不会自动与 GitHub 同步。没有网络时，也能修改文件、查看本地差异和执行 commit。</strong></span>

### 2.2 一台电脑可以有多个本地仓库吗？

可以。通常一个项目一个独立文件夹，分别有自己的 `.git`：

```text
agent学习/
├── gitagent/
│   └── .git/
├── another-project/
│   └── .git/
└── my-website/
    └── .git/
```

克隆另一个项目时，先进入父目录，再克隆。下面的“仓库地址”和“新文件夹名”需要替换：

```bash
cd "/Users/sqdyg/Documents/zhaixiaoyan/agent学习"
git clone 仓库地址 新文件夹名
```

不指定新文件夹名时，Git 默认使用仓库名称创建目录。

<span style="color:#c62828"><strong>通常你在哪个仓库目录及其子目录中执行 Git 命令，就是在操作哪个仓库。初学时建议将不同项目并列存放。</strong></span>

## 3. main、origin、origin/main 和 HEAD

| 名称 | 含义 |
| --- | --- |
| <span style="color:#c62828"><code>main</code></span> | 本例的本地分支名；名字本身没有特殊功能 |
| <span style="color:#c62828"><code>origin</code></span> | 克隆时为远程仓库起的默认名称，关联远程地址 |
| <span style="color:#c62828"><code>origin/main</code></span> | 本地保存的远程 main 状态记录，并不是实时查询 GitHub 得到的结果 |
| <span style="color:#c62828"><code>HEAD</code></span> | 表示当前所在位置；通常指向当前分支，再由分支指向当前提交 |

“默认分支是 main”表示打开 GitHub 仓库时通常先展示它；普通 clone 后通常会检出对应的本地分支。

<span style="color:#c62828"><strong>默认分支不等于唯一分支，也不保证比其他分支更新。工作区干净，也不等于已经与 GitHub 实时同步。</strong></span>

## 4. clone：把项目克隆到 gitagent

本次开始时，`gitagent` 已存在且为空，电脑也已经安装 Git。旁边原有的 `helloagent` 文件夹并不是本次的 Git 仓库。

进入目标目录：

```bash
cd "/Users/sqdyg/Documents/zhaixiaoyan/agent学习/gitagent"
```

<span style="color:#c62828"><strong>git clone：复制远程仓库及其版本历史，并建立远程关联。</strong></span>

普通网络环境下使用：

```bash
git clone https://github.com/shuaiqideyange/hello-agents.git .
```

<span style="color:#c62828"><strong>末尾的点号 <code>.</code> 表示直接克隆到当前目录。已有目录通常需要为空；不带点号，则默认再创建 hello-agents 子目录。</strong></span>

本次网络排查后实际采用了显式代理：

```bash
git -c http.proxy=http://127.0.0.1:12450 clone https://github.com/shuaiqideyange/hello-agents.git .
```

克隆成功后检查：

```bash
git status
```

本次成功输出：

```text
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

这里表示当前处于 main，与本地记录的 origin/main 一致，而且没有未提交的工作区修改。

## 5. 本次网络报错与代理排查

### 5.1 Could not resolve host: github.com

最初克隆出现：

```text
fatal: unable to access 'https://github.com/shuaiqideyange/hello-agents.git/':
Could not resolve host: github.com
```

<span style="color:#c62828"><strong>这个错误表示域名解析失败：还没有成功把 github.com 转换为可连接的网络地址，并不是已经验证了账号但没有写入权限。</strong></span>

检查顺序：

1. 浏览器尝试打开 GitHub 和百度。本次两者都能打开。
2. 在终端执行 DNS 查询：

   ```bash
   nslookup github.com
   ```

3. 本次查询输出：

   ```text
   ;; connection timed out; no servers could be reached
   ```

4. 查看 Sakuracat 设置，确认 Mixed 代理入口为 `127.0.0.1:12450`，支持 HTTP / SOCKS5。
5. 用显式代理测试远程访问：

   ```bash
   git -c http.proxy=http://127.0.0.1:12450 ls-remote https://github.com/shuaiqideyange/hello-agents.git HEAD
   ```

成功返回：

```text
36f583c233ea5cee4537ef93455702b55053af18    HEAD
```

这表示这条命令通过指定的代理成功读取了远程 HEAD。它不证明默认连接方式正常，也不证明具有 push 权限；本例是公开仓库，读取和写入的验证要求不同。

### 5.2 为什么开了 TUN，终端仍然报错？

TUN 正常工作且路由、DNS 配置匹配时，通常也能接管终端程序的流量，不依赖 Git 自己配置 HTTP 代理。

```text
DNS 查询 github.com 的地址
            ↓
连接该地址，传输仓库数据
```

本次报错发生在 DNS 阶段。浏览器可能使用缓存、自己的 DNS 机制或代理路径，因此浏览器可用与终端 DNS 查询失败可以同时发生。

<span style="color:#c62828"><strong>不能仅凭“Git 没设置代理”断定 TUN 没有接管 Git。本次没有查明 TUN 具体哪项配置导致失败，只确认了显式代理入口可用。</strong></span>

### 5.3 临时代理参数怎么理解？

| 参数部分 | 作用 |
| --- | --- |
| <span style="color:#c62828"><code>-c http.proxy=http://127.0.0.1:12450</code></span> | 只为这次 Git 命令指定代理，不写入永久配置 |
| `127.0.0.1` | 当前电脑的本机地址 |
| `12450` | 本次截图中的代理服务端口，换设备或设置后可能不同 |
| 代理地址中的 `http://` | 表示连接代理使用的协议，不要求与仓库的 `https://` 相同 |

## 6. status 和 add：检查修改、加入暂存区

在 `gitagent` 中新建并保存 `git-learning-notes.md`。最初建议写入：

```markdown
# Git 学习笔记

- clone：把远程仓库克隆到本地。
- status：查看当前分支和文件修改状态。
```

<span style="color:#c62828"><strong>git status：查看当前分支、暂存状态及工作区变化，不展示完整的具体差异。</strong></span>

```bash
git status
```

新文件尚未被跟踪时：

```text
Untracked files:
    git-learning-notes.md

nothing added to commit but untracked files present
```

文件在终端通常显示红色，属于状态提示，不是报错。不同编辑器可能有自己的配色。

<span style="color:#c62828"><strong>git add：将指定文件当前的内容加入暂存区，为下一次提交做准备。</strong></span>

```bash
git add git-learning-notes.md
git status
```

暂存成功后：

```text
Changes to be committed:
    new file:   git-learning-notes.md
```

<span style="color:#c62828"><strong>add 不创建提交，也不上传。它保存的是执行 add 时的内容；之后再改文件，需要再次 add，才能把新修改也纳入暂存区。</strong></span>

## 7. 设置提交署名：局部配置与 --global

Git 每次提交会记录作者姓名和邮箱。将下列占位文字替换为你自己的信息后执行：

```bash
git config user.name "你的署名"
git config user.email "你的提交邮箱"
```

邮箱建议使用 GitHub 已验证邮箱或其提供的隐私邮箱。署名会随提交记录共享。

| 写法 | 作用范围 |
| --- | --- |
| <span style="color:#c62828"><code>git config user.name "名字"</code></span> | 在当前仓库中设置，通常保存于 `.git/config` |
| <span style="color:#c62828"><code>git config --global user.name "名字"</code></span> | 为当前电脑用户的各个仓库提供默认署名 |

全局配置示例：

```bash
git config --global user.name "你的署名"
git config --global user.email "你的提交邮箱"
```

<span style="color:#c62828"><strong><code>--global</code> 中间没有空格。仓库单独设置的同名配置优先于全局默认值。修改署名影响之后的新提交，不会自动修改已有提交。</strong></span>

<span style="color:#c62828"><strong>user.name 和 user.email 是提交署名，不是 GitHub 登录凭据，不会授予仓库访问权限。</strong></span>

## 8. commit：保存一次本地提交

<span style="color:#c62828"><strong>git commit：将暂存区准备好的内容保存为本地的一个版本。</strong></span>

```bash
git commit -m "添加 Git 学习笔记"
```

`-m` 后面是本次修改的说明。提交完成后：

```bash
git status
```

本次得到：

```text
On branch main
Your branch is ahead of 'origin/main' by 1 commit.

nothing to commit, working tree clean
```

含义：本地比记录的远程分支多一次提交；修改已保存，本地工作区干净，但这次提交尚未上传。

<span style="color:#c62828"><strong>“本地仓库中的提交”不只包括自己创建的 commit，也包括 clone、fetch、pull 从远程获取并保存在本地的提交。</strong></span>

## 9. push：上传提交与 GitHub 身份验证

<span style="color:#c62828"><strong>git push origin main：把本地 main 推送到 origin 对应的远程 main。</strong></span>

本次通过代理推送：

```bash
git -c http.proxy=http://127.0.0.1:12450 push origin main
```

### 9.1 Username 和 Password 应该填什么？

```text
Username for 'https://github.com':
```

本例填 GitHub 登录用户名 `shuaiqideyange`，不是必须与 `git config user.name` 相同的署名。

```text
Password for 'https://shuaiqideyange@github.com':
```

<span style="color:#c62828"><strong>通过 HTTPS 用这种方式验证时，Password 处填个人访问令牌（Token），不能使用 GitHub 登录密码。输入或粘贴时终端通常不显示字符。</strong></span>

### 9.2 创建或修改细粒度 Token

入口：[细粒度 Token 列表](https://github.com/settings/personal-access-tokens)，或 [创建页面](https://github.com/settings/personal-access-tokens/new)。

手动导航：右上角头像 → Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token。

本次学习需要的设置：

| 设置 | 本例选择 |
| --- | --- |
| Token name | `git-learning` |
| Expiration | 较短有效期，例如 7 天 |
| Resource owner | `shuaiqideyange` |
| Repository access | `Only select repositories`，选 `hello-agents` |
| <span style="color:#c62828">Contents</span> | <span style="color:#c62828"><strong>Read and write</strong></span> |
| Metadata | 自动附带的必需权限，保持 Read-only |
| Account / User permissions | 本次无需额外添加 |

<span style="color:#c62828"><strong>Metadata 标注 Required 并自动出现是正常的。选择仓库和授予 Contents 写权限，是两个都需要确认的步骤。</strong></span>

### 9.3 本次 403 的原因与处理

本次出现：

```text
remote: Permission to shuaiqideyange/hello-agents.git denied to shuaiqideyange.
fatal: ... The requested URL returned error: 403
```

403 表示请求到达服务端，但操作未获授权。本次截图明确显示 Token 没有选中仓库，也没有仓库权限；修改为上表设置并点击 Update 后再试。

如果要排除凭据缓存的影响，本次重试使用：

```bash
git -c http.proxy=http://127.0.0.1:12450 -c credential.helper= push origin main
```

<span style="color:#c62828"><strong><code>-c credential.helper=</code> 临时跳过凭据助手，让此次命令重新询问凭据；它不会删除之前保存的凭据，也不会永久改变设置。</strong></span>

### 9.4 原来的 Token 找不到怎么办？

完整 Token 只在生成时显示一次，不能在详情页重新查看。如果没有自行保存，可在详情页点击 Regenerate token，按提示重新生成。

<span style="color:#c62828"><strong>重新生成后旧 Token 失效，应使用新 Token。Token 不要发到聊天、写进项目文件或提交到 Git 仓库；可保存在密码管理器中。</strong></span>

### 9.5 本次成功推送的证据

```text
To https://github.com/shuaiqideyange/hello-agents.git
   36f583c..901e614  main -> main
```

这表示远程 main 从原来的 `36f583c` 更新到了 `901e614`。刷新 GitHub 仓库页面，可以看到学习笔记。

## 10. pull：从 GitHub 获取并整合更新

为练习 pull，本次先在 GitHub 网页打开学习笔记，点击铅笔图标修改，新增：

```markdown
- pull：获取远程更新，并整合到当前分支。
```

然后点击 Commit changes，将修改直接提交到远程 main。网页提交不会立即更新电脑里的文件。

<span style="color:#c62828"><strong>git pull 会先获取远程提交，再按所选策略整合到当前本地分支；它不是简单地用远程文件覆盖本地文件。</strong></span>

```bash
git -c http.proxy=http://127.0.0.1:12450 pull --ff-only origin main
```

本次输出中的关键部分：

```text
901e614..55339b0  main -> origin/main
Updating 901e614..55339b0
Fast-forward
 git-learning-notes.md | 1 +
 1 file changed, 1 insertion(+)
```

表示已获取远程新提交，并让本地分支快进到 `55339b0`；一个文件新增一行。

### 10.1 --ff-only 和不加它有什么区别？

| 命令 | 本地历史是远程历史的前段时 | 本地与远程分叉时 |
| --- | --- | --- |
| <span style="color:#c62828"><code>git pull --ff-only origin main</code></span> | 可以直接快进跟上远程 | 停止整合，不自动处理分叉 |
| <span style="color:#c62828"><code>git pull origin main</code></span> | 通常同样快进，具体还受配置影响 | 根据配置合并、变基，或要求明确选择策略 |

<span style="color:#c62828"><strong>省略 --ff-only 不等于一定自动合并。显式使用 --ff-only 可以确保出现分叉时先停下来检查。</strong></span>

### 10.2 网页和本地各自 commit，会发生什么？

如果本地和远程从共同版本 A 出发，在网页提交 C 后，本地没有先拉取，又独立提交 B：

```text
A ── B  ← 本地 main
 \
  └─ C  ← GitHub 上的 main
```

这叫历史分叉。它不表示 Git 自动创建了一个新名字的分支。

执行 `pull --ff-only` 仍可能成功获取远程对象、更新远程跟踪记录，但会拒绝整合，本地 main 仍停在 B。

| 概念 | 判断条件 |
| --- | --- |
| <span style="color:#c62828"><strong>分叉</strong></span> | 两边各自有对方没有的新提交，即使改的是不同文件也可能分叉 |
| <span style="color:#c62828"><strong>冲突</strong></span> | 整合时 Git 无法自动决定如何组合内容，例如双方对同一行做了不兼容的修改 |

<span style="color:#c62828"><strong>分叉不等于冲突。不同文件的修改通常能自动整合；出现分叉后，需要明确采用合并（merge）或变基（rebase）等处理方式。</strong></span>

合并通常用一次新提交汇合两条历史；变基则将本地提交的修改重放到新的基础上，生成新的提交编号。这是后续进阶主题，本次没有实际执行分叉整合。

## 11. diff 实操：看清修改发生在哪里

### 11.1 三个对比位置

```text
工作区  ←── git diff ──→  暂存区  ←── git diff --staged ──→  HEAD
```

| 命令 | 比较对象 | 用途 |
| --- | --- | --- |
| <span style="color:#c62828"><code>git status</code></span> | 汇总当前状态 | 哪些文件新建、修改、暂存了 |
| <span style="color:#c62828"><code>git diff</code></span> | 工作区 vs 暂存区 | 查看已跟踪文件尚未暂存的修改 |
| <span style="color:#c62828"><code>git diff --staged</code></span> | 暂存区 vs HEAD | 检查准备纳入下次提交的修改 |
| <span style="color:#c62828"><code>git show</code></span> | 默认展示 HEAD 提交 | 查看最新提交的说明及补丁；合并提交等特殊情况展示形式会不同 |

<span style="color:#c62828"><strong>git diff 没有输出，不一定表示没有修改：内容可能已经暂存；未跟踪的新文件通常也不会被普通 git diff 展示。</strong></span>

### 11.2 本次操作顺序

先进入仓库并执行 `git status`。起始状态干净后，在已有的学习笔记末尾增加内容并保存，先不要 add。

对话中建议增加一条 diff 笔记；你实际增加的是 `哈哈哈哈哈`。执行：

```bash
git diff
```

你给出的差异为：

```diff
diff --git a/git-learning-notes.md b/git-learning-notes.md
index 5f17be5..06f9c0a 100644
--- a/git-learning-notes.md
+++ b/git-learning-notes.md
@@ -1 +1,2 @@
 - pull：获取远程更新，并整合到当前分支。
+哈哈哈哈哈
\ No newline at end of file
```

逐项解读：

| 输出 | 含义 |
| --- | --- |
| `a/…` 与 `b/…` | 对比两侧的标签；这里是同一个文件的修改前后，不是两个真实文件夹 |
| `index 5f17be5..06f9c0a` | 两侧文件内容的标识，不是两次提交的编号 |
| `100644` | 普通非可执行文件的文件模式 |
| `--- a/…` 与 `+++ b/…` | 旧侧和新侧的文件标题，不表示正文删除、新增了三个符号 |
| `@@ -1 +1,2 @@` | 差异片段在旧侧从第 1 行开始，占 1 行；新侧从第 1 行开始，占 2 行 |
| <span style="color:#c62828"><code>+哈哈哈哈哈</code></span> | <span style="color:#c62828">正文行首的加号表示新增一行</span> |
| 正文最左侧 `-` | 删除的行；若改写一行，通常展示为删除旧行、增加新行 |
| 正文最左侧空格 | 未改动的上下文行 |
| `\ No newline at end of file` | 文件末尾没有换行符，是提示，不是报错 |

原来的笔记内容以 `- pull` 开头。差异中的这一行实际最左边还有一个空格，所以是未修改的上下文；不能把笔记本身的短横线误读成删除标记。

如果 diff 进入翻页界面，按 `q` 退出。`diff` 本身只读，不会修改、暂存或上传文件。

接着执行：

```bash
git add git-learning-notes.md
git diff
git diff --staged
```

如果没有额外修改，预期是：普通 `git diff` 没有输出，而 `git diff --staged` 仍能显示 `+哈哈哈哈哈`。

### 11.3 提交后如何查看？

对话最后建议了下面的后续练习，但尚未收到你确认执行成功的结果：

```bash
git commit -m "练习使用 git diff 查看修改"
git diff
git diff --staged
git status
git show
```

如果把上述修改完整暂存并提交后没有再改文件，两条 diff 都应没有输出，工作区干净；`git show` 则可以查看刚创建的提交和新增内容。

<span style="color:#c62828"><strong>此处 HEAD 是当前所在的本地提交，不要求它已经 push。pull 获得的提交也可以成为 HEAD。</strong></span>

## 12. 日常命令速查与作用对比

下表中的简单命令不包含本次所需的代理参数。执行联网命令时，按后面的示例补上代理。

| 命令 | 作用 | 是否需要访问远程 |
| --- | --- | --- |
| <span style="color:#c62828"><code>git clone 仓库地址 目录</code></span> | 建立本地副本，获取历史并设置远程关联 | 本例需要 |
| <span style="color:#c62828"><code>git status</code></span> | 检查状态 | 否 |
| <span style="color:#c62828"><code>git diff</code></span> | 检查未暂存差异 | 否 |
| <span style="color:#c62828"><code>git add 文件名</code></span> | 把指定文件当前内容放入暂存区 | 否 |
| <span style="color:#c62828"><code>git diff --staged</code></span> | 检查暂存区相对 HEAD 的差异 | 否 |
| <span style="color:#c62828"><code>git commit -m "说明"</code></span> | 创建本地提交 | 否 |
| <span style="color:#c62828"><code>git show</code></span> | 查看当前提交 | 否 |
| <span style="color:#c62828"><code>git push origin main</code></span> | 上传本地 main 的提交 | 是 |
| <span style="color:#c62828"><code>git pull --ff-only origin main</code></span> | 获取远程更新，只允许快进整合 | 是 |
| <span style="color:#c62828"><code>git ls-remote 仓库地址 HEAD</code></span> | 查询远程 HEAD，不克隆工作区 | 是 |

一次常见的修改和上传流程：

```bash
cd "/Users/sqdyg/Documents/zhaixiaoyan/agent学习/gitagent"
git status
```

在编辑器修改并保存文件后：

```bash
git diff
git add git-learning-notes.md
git diff --staged
git commit -m "说明这次修改"
git -c http.proxy=http://127.0.0.1:12450 push origin main
```

需要获取远程更新时：

```bash
git -c http.proxy=http://127.0.0.1:12450 pull --ff-only origin main
```

只有需要重新输入凭据、排除缓存影响时，才使用此前的 `-c credential.helper=` 版本。它不是日常每次都必须添加的参数。

## 13. 本次练习已验证到哪里

| 项目 | 对话中的结果 |
| --- | --- |
| clone | 成功，项目位于 gitagent |
| add | 成功，学习笔记进入暂存区 |
| 首次 commit | 成功，状态提示领先远程 1 次提交 |
| push | 成功，远程从 36f583c 更新为 901e614 |
| 网页修改后 pull | 成功，本地快进到 55339b0，新增一行 |
| diff 查看新增内容 | 成功，看到新增的“哈哈哈哈哈” |
| 暂存后查看差异 | 已给出操作指导，并收到相同差异输出 |
| diff 练习后的再次 commit / push | 尚无成功输出确认，不假定已经完成 |

## 14. 官方资料

- [Git：git-config（代理、配置项）](https://git-scm.com/docs/git-config)
- [Git：gitcredentials（凭据助手）](https://git-scm.com/docs/gitcredentials)
- [GitHub：管理个人访问令牌](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)
- [Mihomo：TUN 配置说明](https://github.com/MetaCubeX/Meta-Docs/blob/main/docs/config/inbound/tun.en.md)（用于理解 TUN / DNS 机制，不代表已确认 Sakuracat 的具体故障原因）
