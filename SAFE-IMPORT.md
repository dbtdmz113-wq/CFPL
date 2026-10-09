# 安全入库（Mac / VS Code）

本仓库初始配置不运行、不重建现有程序，也不启用 GitHub Actions。
仅凭 .gitignore 或私有仓库不能证明源码没有凭证。
.env.example 只是占位模板；现有程序不会自动读取它。

## 1. 先设为私有
打开 GitHub 仓库 Settings → General → Danger Zone → Change visibility → Private，
核对仓库名后完成页面确认。确认仓库首页显示 Private 后才导入业务代码。
如果已安装 GitHub CLI，可在 VS Code 终端执行：
```sh
gh auth status
gh api user --jq .login
gh repo edit dbtdmz113-wq/CFPL --visibility private --accept-visibility-change-consequences
gh repo view dbtdmz113-wq/CFPL --json nameWithOwner,visibility
```
账号应为 dbtdmz113-wq，最后 visibility 必须为 PRIVATE。失败则停止入库。
不要把 GitHub Token 粘贴到命令、源码或聊天中；需要登录时使用 gh auth login 的浏览器流程。

## 2. 保留运行目录，另建代码副本
不要停止终端、移动运行脚本、修改现有配置或清除浏览器登录状态。
在 VS Code 终端进入一个新目录的父目录，然后：
```sh
git clone https://github.com/dbtdmz113-wq/CFPL.git CFPL-safe
cd CFPL-safe
git switch -c import-reviewed-code
```
若 CFPL-safe 已存在，请使用另一个新的目录名。
只复制需要管理的 .py/.js/.cjs/.mjs/.ts 源码和必要依赖清单。
不要复制整个运行目录、.git、.env、Cookie、浏览器用户目录、日志、HAR、截图、数据导出和备份。
保留 package-lock.json / pnpm-lock.yaml / yarn.lock 等实际使用的锁文件。

## 3. 只在副本审查
用 VS Code 全局搜索 Cookie、Token、password、Authorization、Webhook、
open.feishu.cn、hooks、storageState、userDataDir、connectOverCDP。
同时检查请求头、URL 查询参数、配置文件和依赖清单。
这是辅助搜索，不代表完整安全扫描；手动查看每一个拟提交文件。
若源码有硬编码凭证，在副本中移除真实值，替换为环境变量读取。
记录变量名到 .env.example，始终保留空值。不要用脱敏后的半截真实值。
原运行程序暂时不改。配置迁移和运行验证应另安排，不在本次入库中切换生产。
仅凭模板不能保证程序可运行；Node 的 process.env 和 Python 的 os.environ
均不自动读取 .env，加载方式需与现有程序匹配。
如果浏览器状态文件使用其他名称，把实际相对路径加到 .gitignore。

检查忽略规则：
```sh
git check-ignore .env logs/check.log browser-data/check cookies/check.json
git check-ignore .env.example
git status --short --ignored
```
第一条应输出四个路径；第二条正常情况无输出且返回码为 1。
不要使用 git add -f。ignore 对已经跟踪的文件不生效。
如果新副本意外暂存了敏感文件，先 git restore --staged -- 具体路径，
补充 ignore 后再检查；若已经提交，停止 push，审查整个本地提交历史。
不要认为删掉最新版本就从历史删除了凭证。

## 4. 白名单暂存、扫描、提交
先明确实际文件名，逐个暂存，例如：
```sh
git add -- .gitignore .env.example SAFE-IMPORT.md
git add -- 已审查脚本的实际相对路径
git diff --cached --name-only
git diff --cached
git diff --cached --check
```
第二条的中文是占位，必须换成真实路径；不要直接复制执行。
diff 只在本地审查，不要上传截图或贴到聊天。
可用本地 Gitleaks 加强扫描，先确认版本和可用命令：
```sh
gitleaks version
gitleaks git --help
gitleaks git --pre-commit --staged --redact .
```
需事先安装工具；扫描失败或有发现时停止提交，处理后重扫。
扫描不能代替人工检查，也不要盲目添加排除规则。
只在检查通过后：
```sh
git commit -m "Import reviewed automation source without credentials"
gh repo view dbtdmz113-wq/CFPL --json visibility --jq .visibility
```
再次确认输出 PRIVATE；然后：
```sh
git push -u origin import-reviewed-code
```
在 GitHub 比较分支，复查文件列表和 diff 后再合并 main。不使用强制推送。
此流程不会把 GitHub 代码自动部署到原 Mac，不会修复既有推送故障。

## 5. 运行配置和历史泄露
真实配置只留在 Mac 本地，不入 Git。私有仓库中的源码也必须去除凭证。
如以后使用 Actions，使用 GitHub Secrets，谨慎对待日志与构建产物。
若确认真实凭证曾推送，先撤销/轮换受影响凭证，再处理历史及副本；
私有化或删除文件不能撤回他人已取得的副本。
没有泄露证据时，不为本次入库强制轮换现有登录凭证，以免影响运行。
