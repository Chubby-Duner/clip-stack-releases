# ClipStack 分发仓库

ClipStack 是一款 Windows 桌面剪贴板管理工具：文本 / 富文本 / 图片历史、拼音搜索、固定片段、快捷唤起、按格取词与表格重建，数据全部保存在本机。

**本仓库只托管分发产物**——安装包、免安装包与自动更新清单 `updates/latest.json`，**不含源码**（源码仓库未公开）。

## 官方下载

| 渠道 | 地址 |
| --- | --- |
| GitHub Releases（推荐） | <https://github.com/Chubby-Duner/clip-stack-releases/releases> |
| Gitee Releases（国内加速） | <https://gitee.com/chubby-duner/clip-stack-releases/releases> |

两处是**同一套产物**，互为备份。**除这两处之外的任何来源都不是官方发布**——第三方下载站、网盘分流、「绿色版 / 破解版 / 去广告版」合集、他人 fork 本仓库后替换的安装包，作者均无法保证其安全性，也不承担相关责任。

> 剪贴板工具天然能读取你复制的全部内容（包括密码、令牌、私钥）。来源不明的安装包请直接删除，**不要**抱着「先试试能不能用」的心态运行。

## 完整性校验

| 场景 | 方法 |
| --- | --- |
| 应用内更新（推荐） | 「设置 → 关于 → 检查更新」自动 minisign 验签，签名不符直接拒绝安装，无需手工核对 |
| 手动下载安装包 | 用清单中的签名 + 公开公钥自行核验，步骤见 [SECURITY.md](SECURITY.md#三手动下载后如何核验) |
| 免安装 zip | **不在签名覆盖范围内**，只能双端哈希交叉比对，安全性弱于安装包 |

## 版本与反馈

- 当前版本与发行说明见 [Releases](https://github.com/Chubby-Duner/clip-stack-releases/releases)
- 问题反馈：[GitHub Issues](https://github.com/Chubby-Duner/clip-stack-releases/issues) · [Gitee Issues](https://gitee.com/chubby-duner/clip-stack-releases/issues)
- 安全问题请先看 [SECURITY.md](SECURITY.md)，**不要**在 Issue 里直接贴出可被利用的细节

## 免责与许可

- [免责声明](DISCLAIMER.md)——官方渠道、无担保、剪贴板数据的固有风险、责任限制
- [安全策略](SECURITY.md)——漏洞报告、产物完整性校验、已知安全边界
- [使用许可](LICENSE) · [标识条款](NOTICE)——免费使用范围，以及名称 / 图标的保留条款

> **摘要**：本软件按「现状」提供，不附带任何担保。历史数据以**明文**存放在本机默认数据目录 `%APPDATA%\com.clipstack\`（可在设置里改到其他目录），运行日志同样不脱敏，备份 ZIP 同样不加密，卸载即丢失。使用风险与合规责任由使用者自行承担，完整条款以 [DISCLAIMER.md](DISCLAIMER.md) 为准。

## 关于本仓库的提交

`updates/latest.json` 由发版流水线自动生成并提交，因此提交历史中出现 `chore: update manifest to vX.Y.Z` 之类的机器提交属正常现象。本仓库不接受 Pull Request（会干扰自动发布的清单与标签），如需反馈请提 Issue。
