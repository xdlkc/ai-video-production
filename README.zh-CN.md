# ai-video-production

[English](./README.md) | 简体中文

> 用「Claude 当总导演 + Codex 当画师 + Seedance 当动画师」的多模型协作流水线，把一句想法做成一支完整的 AI 短片。

这是一个可被 Claude Code、Codex、Cursor、Gemini CLI 等兼容 Agent Skills 的智能体直接加载的 [`SKILL.md`](./SKILL.md) 技能包。核心目标只有一个：**用尽量少的钱、尽量低的抽卡率，做出一条自己满意的 AI 视频。**

## 为什么需要它

AI 视频最大的成本不是模型订阅，而是**反复重抽**：提示词写错一句话不要钱，重画一张图基本不要钱，但**重新生成视频按秒付费**。因此整条流水线严格遵循成本阶梯：

```text
文字里能发现的错  →  不留到图上
图上能发现的错    →  不留到视频上
视频里能修的错    →  不重拍（先剪掉 / 先写代码修）
必须重拍的镜头    →  先改提示词或首尾帧，再重拍，只拍这一镜
```

## 角色分工

用户全程只和「导演」一个入口对话，其余模型与工具在后台各司其职：

| 角色 | 负责 |
| --- | --- |
| **用户** | 选方向、确认画风、批准花钱；亲自验收动态手感与声音 |
| **导演**（Claude Code 等主 agent） | 写故事和分镜、写全部提示词、调用 Codex 与 Seedance、抽帧放大检查、用代码做后期 |
| **画师**（Codex） | 通过 `codex exec --skip-git-repo-check -s read-only` 画设定图、样帧与每个镜头的首帧/尾帧 |
| **动画师**（Seedance） | 通过火山方舟接口（`doubao-seedance-2-5-260628` / `doubao-seedance-2-0-mini-260615`）把首尾帧生成视频片段 |
| **工具**（`ffmpeg` / Python） | 拼接、运动模糊转场、调色、放大与按拍点卡点 |

为进一步提高一次成片率，[`SKILL.md`](./SKILL.md) 还提供了可选的**四线程工作模式**：由主做协调**审核**、**监督**与**代理**三个只读子 agent，在既定预算与用户偏好范围内把关每一步交付。

## 八步流水线

1. **想法 → 定稿剧情**：逐转折追问现实逻辑、一句说清主题、开头抓人、旁人反应分先后、避开 AI 固定弱点。
2. **分镜与配色**：设计每镜景别/机位/运镜与画面运动源，按 38%/62% 构图生成 1920×1080 色块草图并转黑白检验明暗对比。
3. **定画风 + 画设定图**：先定画风再为角色与固定道具出正面/侧面/表情设定图，写死不对称位置细节。
4. **出首尾帧**：先写首尾帧文本方案并过审 → 先画内容更多的那一帧（常为尾帧）→ 局部修改派生另一帧锁定机位 → 裁成一致 16:9 尺寸 → 按 8 项清单逐张放大核查。
5. **七段式视频提示词 + 提交前审核**：按 `【画风】【镜头】【构图与配色】【人物】【逐秒】【声音】【约束】` 撰写，逐条对照 [`references/pitfalls.md`](./references/pitfalls.md) 规避踩坑，并在付费提交前由**新开只读审核 agent** 逐帧核对通过。
6. **生成视频**：720p 起拍、先用最难镜头验路、保留所有旧版本文件，并人工确认音轨可用性。
7. **代码后期（`ffmpeg` + Python）**：严格执行 BT.601 解码 → 无损中间文件 → BT.709 交付色彩管理，按段对齐亮度彩度、8–16 份运动模糊切分转场、lanczos 放大 1080p 及基于原曲测拍卡点。
8. **全片只读复核 + 交付**：新开盲审 agent 不带预设结论解码抽帧复查切点与硬伤，明确标出动态与声音等不可机检项后交付。

项目产物统一归档于新建项目目录下的 `01-story/`、`02-shotlist/`、`03-style/`、`04-frames/`、`05-prompts/`、`06-clips/`、`07-final/`。

## 安装

推荐使用 [`npx skills`](https://github.com/vercel-labs/skills) 安装，它会自动将技能软链到本机已装的各个 agent（Claude Code / Codex / Cursor / Gemini CLI 等）：

```bash
# 装到当前项目（默认 project 作用域）
npx skills add xdlkc/ai-video-production -y

# 装到全局，供所有项目使用
npx skills add xdlkc/ai-video-production -g -y

# 只装这一个 skill，并软链到全部已检测到的 agent
npx skills add xdlkc/ai-video-production --skill ai-video-production --all
```

不安装、只想临时用一次：

```bash
npx skills use xdlkc/ai-video-production@ai-video-production
```

常用管理命令：

```bash
npx skills list          # 查看已安装的 skill
npx skills update -y     # 更新到最新版本
npx skills remove ai-video-production -y
```

也可以直接把本仓库 [`SKILL.md`](./SKILL.md) 的路径提供给 agent 读取，无需安装即可使用。

### 前置环境

- **Codex CLI**（负责生图）
- **`ffmpeg`** 与 **Python**（抽帧检查、色块草图、后期合成）
- 火山方舟 **`ARK_API_KEY`** 环境变量（切勿打印或提交至仓库）

## 仓库与参考文档结构

```text
.
├── SKILL.md                    # 核心八步流水线、八条铁律、四线程模式与交付自查清单
├── references/
│   ├── prompts.md              # 9 套可直接复制的提示词模板（从总纲到四线程模式）
│   └── pitfalls.md             # 模型踩坑对照表、Seedance 版本差异与后期补救优先级
├── README.md
├── README.zh-CN.md
└── LICENSE                     # MIT 许可证
```

`SKILL.md` 保持精炼，长提示词与踩坑对照表拆分至 [`references/prompts.md`](./references/prompts.md) 和 [`references/pitfalls.md`](./references/pitfalls.md)，仅在对应步骤按需读取，避免灌满上下文。

## 已知边界

- 导演 agent **看不到真正的动态，也听不到声音**，动态手感与音轨必须由用户亲自验收。
- 生成提示词中的秒数是近似值，实际节奏依赖后期代码剪辑与变速卡点对齐。
- 模型固有弱点（人物交叉、手拿小物件、画面文字、镜面反射、一镜内换光、多人同时动作等）均已在 [`references/pitfalls.md`](./references/pitfalls.md) 汇总规避写法。

## 许可证

[MIT](./LICENSE)
