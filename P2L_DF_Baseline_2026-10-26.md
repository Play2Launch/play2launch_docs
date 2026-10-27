# P2L 整合实施前 Git 基线

日期：2026-10-26。标签：`baseline-before-df-integration-2026-10-26`。

下列 8 个仓库执行 `pull --ff-only` 成功；附注标签已创建并推送。打标签时服务端与后台在 release-prod，随后按用户要求切换至 main 并同步远程。切换时 main 与 release-prod 提交一致，标签无需变更。所有仓库后续统一在 main 工作，未提交已有本地修改。

| 仓库 | 打标签时的分支 | 标签对应提交 |
| --- | --- | --- |
| play2launch_game_sdk | main | f552f07ee1594ba34576a0014188c3ed820d8c72 |
| PlayToLaunch_admin | release-prod | e4414a223e954a75217a5ca61593f958637f92fc |
| PlayToLaunch_contracts | main | 4350af965d95c886c5406521269b8d5feb07a66c |
| PlayToLaunch_documents | main | 392b9b5d75d7a5eae4464a89060047fa1444c5ef |
| PlayToLaunch_marketcombo | main | 9529dd4cf55b3cc0d2dd21a9eecfac3b1ccc0d68 |
| PlayToLaunch_paperhood | main | 4a070969c4914cc40f9d0b26ce529e83f35c5e23 |
| PlayToLaunch_server | release-prod | b2f5f46ac4c88da7703e5718d1b30386b92b77e3 |
| PlayToLaunch_web | main | a4f5e76715b9c75b0e95ca313f59d2b9665e7d57 |

`sdk-publish-011` 是 SDK 相同远程的另一份 checkout，pull 成功，HEAD 同为 f552f07；未为同一远程重复创建标签。

`PlayToLaunch_paperhood_explore` 的 main HEAD 为 `1fe278dda90de0b00d847ab9d707e1969dec76d9`，通过 ls-remote 核实与远程 main 一致。该仓库 Git 目录权限拒绝创建 FETCH_HEAD 和对象，pull 及附注标签创建失败，未推送该仓库标签。未修改其 ACL 或所有权；此例外尚待处理。

## 本地修改与备份

标签只引用已提交代码，不包含未提交工作区。本次方案和本说明也在标签创建后新增，尚未提交。

原有变更：SDK 的 README、协议、package.json、类型、实现和测试；合约的部署记录、脚本及未跟踪部署文件；Web 的未跟踪 `public/images/p2l-icon.png`。均保留，没有自动提交、stash 或丢弃。

补丁和未跟踪文件备份保存在工作区 `baseline-backups/baseline-before-df-integration-2026-10-26/`；每仓库有 staged.patch、unstaged.patch，及适用的 untracked 目录，manifest.json 记录状态和提交。备份不在项目仓库内，不推送远程。忽略文件、运行时配置和数据库不属于本次 Git 备份。

恢复基线时先保护当前工作；标签只能恢复已提交代码，工作区快照还需结合对应补丁和未跟踪文件。不得直接使用破坏性 reset 覆盖后续修改。
