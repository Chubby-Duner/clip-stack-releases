# 安全策略

本文件说明如何确认你拿到的是官方产物、如何报告安全问题，以及本项目**已知的安全边界**。使用前请同时阅读[免责声明](DISCLAIMER.md)。

## 一、支持的版本

只对**最新正式版本**提供安全修复。旧版本请先升级到最新版再报告问题；已停止维护的版本不再回溯修复。

## 二、报告安全问题

- 反馈入口：[GitHub Issues](https://github.com/Chubby-Duner/clip-stack-releases/issues) · [Gitee Issues](https://gitee.com/chubby-duner/clip-stack-releases/issues)
- **若问题可被直接利用**（例如能导致数据泄露、命令执行、越权），请**不要**在 Issue 中贴出 PoC、完整复现步骤或受影响用户的数据，先用一句话说明「有安全问题，需要私下沟通」，等待作者回复后再提供细节；
- 建议附上：版本号、复现步骤、影响范围、以及你已排除的环境因素。

这是个人维护的项目，响应时间不做 SLA 承诺；确认后会尽快修复并在发行说明中体现。

> 说明：ClipStack 源码未公开，因此无法通过提交补丁的方式参与修复，请以报告为主。

## 三、手动下载后如何核验

### 3.1 官方渠道

只有以下两处发布安装包与更新清单，两者是**同一套产物**：

- <https://github.com/Chubby-Duner/clip-stack-releases/releases>
- <https://gitee.com/chubby-duner/clip-stack-releases/releases>

任何其他来源（第三方下载站、网盘、镜像、他人重新打包的版本）都**不可信**，见 [DISCLAIMER 第二节](DISCLAIMER.md#二被篡改的版本与作者无关)。

### 3.2 用应用内更新（推荐，零操作）

「设置 → 关于 → 检查更新」的整条链路自带验签：

1. 客户端从官方清单地址取 `updates/latest.json`；
2. 下载清单中 `url` 指向的安装包；
3. 用内置 minisign 公钥（密钥 ID `BF1A4D0034E078FC`）校验安装包签名，**签名不符或缺失即拒绝安装**（`requireSignedVersion: true`）。自 v1.25 起，清单签名里缺少 `version:` 标记、或标记的版本与清单声明不一致时同样会被拒绝，且这一步提前到**下载之前**——不会先下完 20~30 MB 才发现问题。

也就是说，即使有人劫持了更新端点、替换了清单里的下载直链，被换上的文件也会因验签失败而装不上。走这条路不需要你手工核对任何哈希。

### 3.3 手动核验安装包（进阶）

手动从 Release 页面下载安装包时，可以自行完成同样的验签。需要 [minisign](https://github.com/jedisct1/minisign/releases) 命令行工具。

**第一步：把公钥保存为 `clipstack.pub`**（与应用内更新校验所用公钥一致）

```
untrusted comment: minisign public key: BF1A4D0034E078FC
RWT8eOA0AE0av9UK5uSYkhU7dkOOagxab3x3IIySsqI9al6UUElPRK9w
```

**第二步：从官方清单取出签名并校验安装包**

```powershell
# 1) 取官方清单（Gitee / GitHub 任选，两处内容应完全一致）
$rel = Invoke-RestMethod "https://raw.githubusercontent.com/Chubby-Duner/clip-stack-releases/master/updates/latest.json"

# 2) signature 字段是 .sig 文件的 base64；已含换行时直接使用
$sig = $rel.platforms.'windows-x86_64'.signature
if ($sig -notmatch "`n") { $sig = [Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($sig)) }
[IO.File]::WriteAllText("$PWD\latest.json.sig", $sig)

# 3) 校验你下载的安装包（文件名换成你实际下载的那个）
minisign -Vm .\ClipStack_1.24.0_x64-setup.exe -p .\clipstack.pub -x .\latest.json.sig
```

输出 `Signature and comment signature verified` 且 trusted comment 中带有对应的 `file:` 与 `version:` 标记，才算通过。

> **注意这条链的边界**：`latest.json` 本身没有独立签名，签名针对的是**安装包**。因此清单的信任根仍是官方仓库的 HTTPS 地址——这正是必须只从官方渠道取清单的原因。签名能证明「这个安装包与作者发布的一致」，不能证明「这个软件本身没有缺陷」。

### 3.4 双端交叉核对

GitHub 与 Gitee 两处发布的同名安装包应当是**字节完全一致**的（发版流水线会回读校验）。若你发现两侧文件大小或哈希不同，说明其中一侧已被替换或上传出错——此时**两侧都不要用**，并通过第二节的方式反馈。

> **注意**：`ClipStack-v<版本>-portable-x64.zip`（免安装版）**不在**更新签名覆盖范围内——`latest.json` 的签名只针对 `-setup.exe`。免安装包目前只能靠上述双端哈希比对来交叉验证，安全性弱于安装包。如果你对来源有疑虑，请优先下载安装包。

## 四、已知的安全边界

以下不是缺陷，是设计上明确接受的行为或固有限制，请据此评估是否适合你的场景：

| 项 | 说明 |
| --- | --- |
| **闭源二进制** | 源码未公开，**无法自行审计**，也无法验证二进制与任何源码一致；本项目未做第三方安全审计，也不作此承诺 |
| 敏感识别是启发式 | 正则匹配，只作辅助防泄漏，**不保证覆盖**，也会误报；命中不阻断捕获（避免误报吞掉用户数据） |
| 数据明文存储 | 历史库为明文 SQLite，图片为原图文件，**未加密**；备份 ZIP 同样未加密 |
| 运行日志同样明文 | 前端错误与警告会落盘到数据目录下的 `clipstack.log`（单条上限 4000 字符），**不做脱敏**；异常信息里若带上了剪贴板正文，也会以明文留在该文件中。日志与历史库同等敏感，把它发给别人排查问题前请自行检查 |
| 发布流水线边界 | 产物由 CI 自动构建，**未**提供构建溯源（provenance）或可复现构建，构建工具链与依赖由第三方维护。签名能证明「这个安装包与作者发布的那一份一致」，不能证明「构建过程本身未被污染」 |
| 本机权限即数据权限 | 任何能以你的账户运行的进程都可读取 `%APPDATA%\com.clipstack\` |
| 网络行为 | 除检查更新外不联网，无遥测、无云端同步 |
| 管理员窗口限制 | 受 Windows UIPI 限制，管理员权限窗口的剪贴板内容捕获不到，向其注入按键也会被丢弃 |
| 自动粘贴 | 回贴时模拟按键，仅在用户显式触发时发生，可关闭 |
| 开发模式保留调试入口 | 发布版已关闭 WebView2 右键菜单与浏览器快捷键；开发模式保留以便调试 |
| 未做代码签名证书 | 安装包使用 minisign 签名保证**完整性**，但**未**购买商业代码签名证书（Authenticode）。因此 Windows SmartScreen 可能对首次运行给出提示，这属于预期现象，不代表文件被篡改 |

## 五、作者不会做的事

为避免冒充，明确声明：**作者不会**通过私信、群聊、网盘或任何非上述官方渠道向你发送安装包，也**不会**要求你关闭杀毒软件、添加信任目录后再运行。遇到此类情况，一律视为诈骗。
