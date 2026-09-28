# arr1chino

> 赛道群内微信昵称：-（就是一个短横线）

## 选择路线

路线二：新开源项目类 —— 训练营期间从 0 到 1 创建项目

## 项目简介

**选区生图**：一个 Photoshop UXP 面板插件。框一个选区 → 写提示词 → 调用生图接口
→ 结果自动缩放、对齐、贴回选区位置（新图层，可一次撤销）。

刻意做得很小：没有登录、没有账号、没有积分、没有云端依赖。
配置一次 API 就能一直用，主面板只留每次生成都要碰的东西。

功能：配置 API（地址 / Key / 协议 / 模型）、拉取模型列表、提示词批量（一行一张）、
按选区生成、结果自动贴回、网络并发、任务级与全局中断（提前截断）。

工程上：零第三方依赖，8 个源文件；纯逻辑部分有 56 项自动化断言；
不依赖 Creative Cloud，直接放进 Photoshop 的 `Plug-ins` 目录即可。

## 项目 / PR

- 仓库：https://github.com/arr1chino/ps-selection-gen
- PR：本 PR（提交文件 `submissions/arr1chino.md`）
- Demo：界面预览 <https://github.com/arr1chino/ps-selection-gen/tree/main/docs/screenshots>；
  真机演示录屏待补（见文末「还没做完的部分」）

## 训练营期间的主要增量

全部代码都是训练营期间写的（2026-09-27 开始），仓库从 `chore: 初始化工程骨架` 开始，
按模块分批提交，截至本文件最后一次更新共 27 次，完整列表见
<https://github.com/arr1chino/ps-selection-gen/commits/main>。

### 做了什么

| 步骤 | 内容 |
| --- | --- |
| 1. 定范围 | 把要做的事压到一条主链路：配置 API → 框选 → 生成 → 贴回。登录、账号、云存储、磁贴工作台这类全部不做 |
| 2. 搭骨架 | UXP 清单、图标、入口面板、目录结构、MIT 协议 |
| 3. 核心逻辑 | 零依赖工具（base64 / UTF-8 / 图片格式嗅探 / 尺寸换算）、配置持久化（Key 走系统加密存储，不落明文） |
| 4. 接口层 | 三种协议族（OpenAI 图片接口 / Gemini `:generateContent` / 对话式 `chat/completions`，覆盖纳米香蕉这类只在对话接口出图的模型）、模型列表拉取与排序、尺寸按 16 倍数换算 |
| 5. Photoshop 层 | 三级降级读选区边界；抓像素 → 发模型 → 结果写临时文件 → `app.open()` 解码 → `imaging.getPixels({targetSize})` 交给 PS 缩放 → `putPixels` 落到选区左上角，一次可撤销 |
| 6. 并发与中断 | 网络并发池与 Photoshop 串行锁分离；`AbortController` 按任务登记，支持「掐一个」和「掐全部」 |
| 7. 测试 | 不依赖 Photoshop 的纯逻辑测试，从 36 项加到 56 项断言 |
| 8. 文档与工程 | README、开发记录、更新日志、一键安装脚本、GitHub Actions（语法检查 + 测试矩阵） |
| 9. 按日志修真实问题 | 按 Photoshop 的 UXP 日志定位并修掉清单写法、图标路径两个真实报错；把 API 配置收进独立设置页，主界面只留生成要用的东西 |
| 10. 真机反馈返工界面 | 拿 PS 里的截图一条条查原因：宿主给原生控件的布局盒子偏大、原生下拉的弹出列表由系统绘制、图标条目重复会被宿主拒绝解析清单、原生控件的文字按宿主行高绘制（不写 `line-height` 就被裁）。改完顺手把"清单必须能解析"做成 14 项自动化测试 |

### 主要提交（节选，时间顺序）

```text
09-27 14:34  chore: 初始化工程骨架（UXP 清单、图标、忽略规则、MIT 协议）
09-27 14:34  feat(ui): 面板界面与入口接线
09-27 14:34  feat(core): 零依赖工具与配置持久化
09-27 14:34  feat(api): 生图接口对接、模型列表拉取与尺寸换算
09-27 14:34  feat(ps): 选区三级降级读取与结果贴回图层
09-27 14:34  feat(queue): 网络并发池、PS 串行锁与分级中断
09-27 14:34  test: 不依赖 Photoshop 的纯逻辑测试（36 项断言）
09-27 14:34  docs: README、开发记录与更新日志
09-27 14:34  chore: 统一文本换行符，避免 Windows 检出时整文件翻红
09-27 15:44  feat(tools): 一键安装脚本，改为直接装进 PS 的 Plug-ins 目录，不再依赖 Creative Cloud
09-27 15:56  fix(manifest): host 改为对象写法，图标补齐 1x/2x 两种尺寸（PS 报 Failed to parse the manifest.json）
09-27 16:07  feat(ui): API 配置收进独立设置页，主界面只留提示词与任务
09-27 16:09  style(ui): 顶栏与设置页改用 margin 控制间距，不依赖 flex gap
09-27 16:22  feat(api): 补上 nano banana 适配——对话式协议、图片提取补全、字段降级重试
09-27 17:14  ci: 加入 Actions 语法检查与测试矩阵；README 补界面预览、接口支持说明与 56 项测试说明
09-27 17:15  fix(manifest): 图标路径改用基名，交给宿主拼 @1x/@2x（原先日志刷 Scaled Icon not found）
09-27 17:57  style(ui): 钉死参数行行高与控件高度，修 Photoshop 面板里三行被撑开
09-27 18:00  fix(ui): 设置视图改为互斥显示，修 Photoshop 里浮层叠字；预览截图按面板宽度重出
09-27 18:01  fix(ui): 原生下拉框换成自绘档位按钮与模型列表，修 PS 里字体与配色不受控
09-27 18:02  docs: 重出界面预览（档位按钮与模型列表），并压缩设置页提示文案
09-27 18:07  fix(manifest): 修图标条目重复导致清单解析失败、插件整个不显示；加清单自检测试
09-27 18:21  style(ui): 设置页放宽行距与控件高度，输入框加高到 30px；预览图改按 360px 宽度重出
09-27 18:26  fix(ui): 输入框显式 line-height，修 PS 里首行文字被裁；行高放宽到 34px
09-27 18:50  style(ui): 参数行再放宽一档，去掉会裁字的 overflow；多行框改像素行高并挪走长占位符
09-27 19:13  docs: 开发记录补上真机反馈驱动的界面返工（8 次提交的原因与取舍）
09-28 16:46  fix(ui): 设置页加即时状态行，修「点拉取模型列表没反应」；拉取加 15 秒超时与报错翻译
09-28 16:46  docs: 开发记录补上「点拉取没反应」的原因与反馈可见性原则
```

## 过程记录

- **commit**：<https://github.com/arr1chino/ps-selection-gen/commits/main> —— 按模块分批提交，每条说明改了什么，上面的时间线就是提交顺序
- **开发日志**：<https://github.com/arr1chino/ps-selection-gen/blob/main/DEVELOPMENT-LOG.md> —— 按「我提的问题 → AI 给了什么 → 我的判断 → 验证结果」记录关键决策与踩坑
- **更新日志**：<https://github.com/arr1chino/ps-selection-gen/blob/main/CHANGELOG.md> —— 按版本倒序，含「已知限制」与「待办」
- **测试**：<https://github.com/arr1chino/ps-selection-gen/tree/main/test> —— `test-core.js` 56 项断言、
  `test-manifest.js` 14 项清单自检，`node test/test-core.js` 与 `node test/test-manifest.js` 全部通过
- **CI**：<https://github.com/arr1chino/ps-selection-gen/actions> —— 每次提交自动跑语法检查与测试
- **截图**：<https://github.com/arr1chino/ps-selection-gen/tree/main/docs/screenshots> —— 面板主界面与设置页（按面板尺寸在浏览器里渲染的界面预览，会注明；不拿预览图充当实机截图）
- **版本**：标签 `v0.1.0` —— <https://github.com/arr1chino/ps-selection-gen/tags>

## 其他说明

### 关于参考资料

参考插件：sdppp、hhps。**代码是独立编写的，没有复制它们的源码。**

借鉴的是「别人踩过的坑」（PS 操作必须串行、网络与 PS 分两把锁、读选区要多方法降级、
16 位文档的值域坑、multipart 手工拼装等）；重新设计的是「我认为可以更简单的地方」
（不用 `<webview>` 套壳，省掉整层消息桥；用 `imaging.encodeImageData` 代替手写编解码；
定位改成一次到位的缩放贴回；只保留主链路，文件数从三位数降到 8 个）。

两边逐条对照写在仓库 README 的「来源说明」与「与参考工程的关系」两节里，可对着代码核对。

### 还没做完的部分（如实记录）

- **真机验证进行中**：插件已被本机 Photoshop 2026 识别并成功解析清单（按 UXP 日志确认），
  日志里的两处真实报错（清单 `host` 写法、图标路径）已修复；
  选区读取、像素方向、贴回对位、16 位文档、并发 3 任务这几项**尚未实测**，跑完会补录屏并更新本文件。
- 不规则选区目前按外接矩形贴回，计划改成把选区当图层蒙版裁掉溢出部分。
- 发送前预览（生成前先看将发给模型的那张裁切图）尚未实现。
