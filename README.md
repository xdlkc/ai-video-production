# ai-video-production

> 用「Claude 当总导演 + Codex 当画师 + Seedance 当动画师」的多模型协作流水线，把一句想法做成一支完整的 AI 短片。

这是一个可被 Claude Code / Codex 等 agent 直接加载的 [`SKILL.md`](./SKILL.md) 技能包。核心目标只有一个：**用尽量少的钱、尽量低的抽卡率，做出一条自己满意的 AI 视频。**

## 为什么需要它

AI 视频最大的成本不是模型订阅，而是**反复重抽**：提示词写错一句话不要钱，重画一张图基本不要钱，但**重新生成视频按秒付费**。所以整条流水线的设计原则是：

```
文字里能发现的错  →  不留到图上
图上能发现的错    →  不留到视频上
视频里能修的错    →  不重拍（先剪掉 / 先写代码修）
必须重拍的镜头    →  先改提示词或首尾帧，再重拍，只拍这一镜
```

## 分工

| 角色 | 负责 |
| --- | --- |
| 用户 | 选方向、确认画风、批准花钱；看动态、听声音 |
| 导演（Claude Code 等主 agent） | 写故事和分镜、写所有提示词、调用其他模型、抽帧放大检查、用代码做后期 |
| 画师（Codex） | 画设定图、样帧、每个镜头的首帧和尾帧 |
| 动画师（Seedance） | 把首尾帧变成几秒钟的视频 |
| 工具（ffmpeg / Python） | 拼接、转场、调色、放大、卡点 |

用户全程只和「导演」一个角色对话。

## 八步流水线

1. 想法 → 定稿剧情
2. 剧情 → 分镜表
3. 定画风 + 色表（先把配色定下来，后面所有图共用）
4. 出首尾帧（设定图 → 样帧 → 每镜首帧/尾帧）
5. 写视频提示词 + **提交前另开 agent 审核**
6. 生成视频（先拍最难的镜头验路，再批量）
7. 后期（全部用代码：拼接、转场、调色、放大、卡点）
8. 全片只读复核 + 交付

## 仓库结构

```
.
├── SKILL.md                    # frontmatter + 八步流程 + 八条规矩 + 四线程模式
├── references/
│   ├── prompts.md              # 可直接复制的九段提示词模板
│   └── pitfalls.md             # 模型踩坑与有效写法对照表
├── README.md
└── LICENSE
```

`SKILL.md` 保持精炼，长提示词与对照表放 `references/`，只在需要时读取，避免一次性灌满上下文。

## 安装

推荐用 [`npx skills`](https://github.com/vercel-labs/skills) 安装，它会自动把 skill 软链到本机已装的各个 agent（Claude Code / Codex / Cursor / Gemini CLI 等）：

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

其他常用命令：

```bash
npx skills list          # 查看已安装的 skill
npx skills update -y     # 更新到最新版本
npx skills remove ai-video-production -y
```

也可以直接把本仓库 `SKILL.md` 的路径喂给 agent 读取，不安装也能用。

需要提前准备的环境：Codex（生图）、ffmpeg、Python、火山方舟的 `ARK_API_KEY`。

## 已知边界

- 导演 agent **看不到真正的动态，也听不到声音**，所以动态手感与音轨必须由用户亲自验收。
- 提示词里的时间是近似的，实际节奏靠后期剪辑与变速对拍。
- 模型有固定弱点（人物交叉、手拿小物件、画面文字、镜面反射、一镜内换光等），已在 `references/pitfalls.md` 汇总规避写法。

## License

MIT
