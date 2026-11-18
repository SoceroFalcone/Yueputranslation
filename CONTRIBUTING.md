# 贡献指南

感谢参与粤普同传。请优先提交范围明确、可以复现和验证的改进。

## 仓库、分支和发布

本项目采用 GitHub flow。打开仓库查看最新源码，请选 `main`。Windows 和 Android 在同一条主线维护，各自的代码按目录区分。[GitHub flow 官方说明](https://docs.github.com/en/get-started/using-github/github-flow)

| 要找的内容 | 在哪里 |
| --- | --- |
| Windows 应用、测试及安装包构建脚本 | [桌面源码](src/yuepu_translate/)、[测试](tests/)、[安装包构建](installer/windows/) |
| Android 应用 | [Android 源码和构建说明](android/) |
| 最近改了什么、如何使用 | [更新日志](docs/CHANGELOG.md)、[Windows 使用说明](docs/windows.md) |
| 模型与第三方软件的许可 | [模型许可](licenses/models/)、[第三方清单](THIRD_PARTY.md) |
| 某个已发布版本的源码与安装附件 | 在发布该版本的仓库查看 Tags 和 Releases。标签固定源码提交，安装附件放入对应 Release。[GitHub Releases 说明](https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases) |

当前维护的仓库是 [SoceroFalcone/Yueputranslation](https://github.com/SoceroFalcone/Yueputranslation)。在维护者的本地副本中，`origin` 指向这个仓库，`upstream` 指向 [原始仓库 Richard-168/Yueputranslation](https://github.com/Richard-168/Yueputranslation)。`origin/main` 和 `upstream/main` 分别记录两边最近取回的主线位置。历史版本 `Release-v0.1.0-alpha.2` 可在上游按标签查找。[上游版本源码](https://github.com/Richard-168/Yueputranslation/tree/Release-v0.1.0-alpha.2)

### 一项改动的流程

1. 先更新本地 `main`，再从它创建一条短期分支，用任务命名，例如 `fix/capture-tail`。同一项改动放在同一条分支。
2. 将工作分支推到自己的仓库，发起合并请求（Pull Request，简称 PR）。在 GitHub 核对目标仓库和目标分支。维护这个派生仓库时，目标选 `SoceroFalcone/Yueputranslation` 的 `main`。
3. 检查改动和测试结果后合并。确认 PR 已合并，且该分支没有其他待合并请求或后续提交，再删除远端和本地工作分支。未使用 PR 的旧分支，先检查每个提交都已包含在 `main` 中。历史可通过已合并的 PR 或提交号查找。
4. 需要发布安装包时，在验证通过的提交上打版本标签，使用 Release 提供安装附件。日常源码更新记入更新日志。

### 空间维护

Git 存放源代码、测试、说明和许可。模型、运行库、编译缓存和本机实验输出由 `.gitignore` 排除。下载大模型或生成安装包时，分别通过模型准备脚本和 Releases 分发。[GitHub 大文件指南](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github)

检查空间时分开看 Git 历史、本机工作目录，以及 GitHub 上的发布附件和 Actions 缓存。清理已合并的分支主要方便浏览，公共历史仍由 `main` 保存。清理本机缓存前先确认对应工具已退出、目录内容可重建，并保留模型、输入样本及验证结果。

## 开始修改

1. 阅读 README 的开发环境和当前限制，使用 Python 3.12 虚拟环境。
2. 按上面的分支流程选择本次改动的目标仓库和工作分支。
3. 应用实现放在 `src/yuepu_translate/`；诊断脚本放在 `experiments/offline-poc/`。运行代码不得反向依赖诊断脚本。
4. 运行 README 的格式、静态、测试和构建检查。修改音频或模型调用时，补充相关真实链路验证，注明测试输入、设备和限制。

文档和纯外观修改不要求新增单元测试；逻辑修改优先补充能捕捉实际错误的回归检查。不得把合成或模拟输出描述成真人实测结果。

## 提交与 PR

提交信息使用 `类型: 简要说明`，中英文均可，例如：

```text
fix: 保留停止前已经完成的字幕
refactor: 独立模型加载模块
docs: 补充 Windows 安装步骤
```

常用类型：`feat`、`fix`、`refactor`、`docs`、`test`、`build`、`ci`、`chore`。这是一项项目约定，GitHub 本身不强制这些类型。

一个提交或 PR 解决一个相关问题。说明触发条件、修改后的行为、验证命令和未验证内容；界面变更可附不含私人内容的截图。避免把大规模格式化与行为修复混在一起。

## 提交前检查

- 查看 `git diff` 和 `git diff --cached`，确认只有预期源码及公共文档。
- 不提交 `.env`、账号凭据、个人录音、歌词转录、内部笔记或机器专属路径。
- 不提交 `models/`、`runtime/`、`downloads/`、`outputs/`、虚拟环境或依赖缓存。
- 不随 PR 附上大模型和第三方二进制；以固定来源、版本和校验值说明依赖。
- 新增依赖时说明用途、许可证和体积影响；不要悄悄更换默认模型。

提交贡献表示你有权提交这些内容，并同意原创贡献按本项目 MIT 许可提供；第三方内容须保留原许可和来源。
