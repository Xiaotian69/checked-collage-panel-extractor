<div align="center">

# PanelPick · 勾选拆图

**从带标记的联系表中识别选中格子，再从无标记版本导出干净原图。**

[![Python 3.10+](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Tests](https://github.com/Xiaotian69/checked-collage-panel-extractor/actions/workflows/test.yml/badge.svg)](https://github.com/Xiaotian69/checked-collage-panel-extractor/actions/workflows/test.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

[快速开始](#快速开始) · [使用方法](#使用方法) · [作品示例](skills/checked-collage-panel-extractor/examples/GALLERY.md) · [问题反馈](https://github.com/Xiaotian69/checked-collage-panel-extractor/issues)

</div>

![11 张大同摄影拼贴作品](skills/checked-collage-panel-extractor/examples/gallery-contact.jpg)

一张联系表里有很多小图，但你只想保留其中几张。PanelPick 对照“带青色标记版”和“无标记版”，定位选中的规则网格，再从无标记版中裁出原始像素，输出独立 PNG、选择报告和可选 ZIP。

这个仓库源于一次摄影拼贴选片实践：直接从带勾图片裁切会把标记一起留下，重新生成又会改变已经满意的作品。项目因此把“选择位置”和“提供最终像素”分开，尽量让拆图过程可核对、可重复，也不依赖图像生成服务。

仓库还包含配套的 [Stoneframe · 石间拼贴](skills/heritage-photo-collage/) Skill，用于从石窟、古建和壁画摄影中整理选片、构图、配色与参考图编辑提示词。它与拆图脚本相互独立，可按需单独使用。

## 项目组成

| 组件 | 用途 | 运行方式 |
| --- | --- | --- |
| [PanelPick · 勾选拆图](skills/checked-collage-panel-extractor/) | 识别亮青色标记，从匹配的干净联系表中导出选中格子 | Python CLI 或 Agent Skill |
| [Stoneframe · 石间拼贴](skills/heritage-photo-collage/) | 为文化遗产摄影整理选片、视觉方案和图像编辑提示词 | Agent Skill；实际生成需要客户端提供图像工具 |
| [石间作品集](skills/checked-collage-panel-extractor/examples/GALLERY.md) | 展示 11 张已有 AI 拼贴与 7 张摄影参考 | GitHub 文档与图片 |

## 功能特性

- 对照带标记版与无标记版，检测新增的亮青色/蓝色笔迹；
- 只从无标记版裁切，避免把勾选标记写入最终图片；
- 支持规则网格、手动行列、多联系表清单和期望数量校验；
- 支持保持原比例、留白补方形或中心裁方形；
- 可按像素收缩外部分隔线，并可选进行有限白边裁剪；
- 支持 1–4 倍 Lanczos 放大、轻锐化、JSON 报告与 ZIP 打包；
- 导出前先校验整个批次，数量不符时不写入结果；
- 拒绝覆盖同名输出，减少误删已有结果的风险；
- 提供 9 项自动化测试和可直接运行的示例素材。

## 快速开始

### 1. 获取项目并创建环境

```sh
git clone https://github.com/Xiaotian69/checked-collage-panel-extractor.git
cd checked-collage-panel-extractor/skills/checked-collage-panel-extractor

python -m venv .venv
```

激活虚拟环境：

```sh
# Windows PowerShell
.venv\Scripts\Activate.ps1

# macOS / Linux
source .venv/bin/activate
```

安装依赖：

```sh
python -m pip install -r requirements.txt
```

需要 Python 3.10 或更高版本。运行时依赖只有 Pillow 和 NumPy。

### 2. 运行仓库示例

```sh
python extract_checked_panels.py \
  --manifest examples/manifest.example.json \
  --out demo-output \
  --expected 3 \
  --square-mode keep \
  --zip
```

成功后，`demo-output/` 中会出现 3 张 PNG、`selection_report.json` 和 `selected_panels.zip`。

## 使用方法

### 自动识别青色标记

准备两张内容、网格和位置一致的图片：

- `marked.png`：画有亮青色标记，例如 RGB `0, 200, 255`；
- `clean.png`：没有标记，用于提供最终导出像素。

下面的例子处理 2 行 × 3 列的联系表，并要求恰好识别 3 个格子：

```sh
python extract_checked_panels.py \
  --marked marked.png \
  --clean clean.png \
  --rows 2 \
  --cols 3 \
  --out selected \
  --expected 3 \
  --zip
```

### 手动指定格子

自动检测不稳定，或只有无标记版时，可以直接指定行列。行列编号从 1 开始：

```sh
python extract_checked_panels.py \
  --clean clean.png \
  --rows 2 \
  --cols 3 \
  --manual "1,1;1,3;2,2" \
  --out manual-output \
  --expected 3 \
  --square-mode keep
```

### 常用参数

| 需求 | 参数 |
| --- | --- |
| 保持每格原比例 | `--square-mode keep` |
| 用白色留白补成方形（默认） | `--square-mode pad` |
| 从中心裁成方形 | `--square-mode crop` |
| 本地放大 2 倍 | `--upscale 2` |
| 每格四边各收 4 像素 | `--gutter 4` |
| 启用有限的自动白边裁剪 | `--trim-white` |
| 手动指定选中行列 | `--manual "1,1;1,3;2,2"` |
| 批量处理多张联系表 | `--manifest batch.json` |
| 校验全批导出数量 | `--expected 3` |

`--trim-white` 默认关闭，因为白色边框可能属于作品本身。放大仅使用传统插值和轻锐化，不会恢复源图不存在的细节。

### 输出内容

```text
selected/
├── sheet01_01_r1_c1.png
├── sheet01_02_r1_c3.png
├── sheet01_03_r2_c2.png
├── selection_report.json
└── selected_panels.zip
```

报告会记录网格、检测分数、行列、裁切坐标和输出文件名。若 `--expected` 与实际数量不符，脚本返回错误且不创建输出目录。

## 在 Agent 中使用

将所需 Skill 文件夹复制到客户端的 skills 目录：

- Windows：`%USERPROFILE%\.codex\skills\`
- macOS / Linux：`~/.codex/skills/`

例如安装 PanelPick 后，可以这样描述任务：

> 使用 $checked-collage-panel-extractor，按照我的青色勾选标记，从无标记版拼图拆出 11 张。输出 1:1 PNG 和 ZIP，保留内部白框与插图。

调用标识是 `checked-collage-panel-extractor`。配套创作 Skill 的调用标识是 `heritage-photo-collage`。详细行为边界分别见各自的 `SKILL.md`。

## 项目结构

```text
.
├── .github/
│   ├── ISSUE_TEMPLATE/              # Bug 与功能建议模板
│   └── workflows/test.yml           # Python 自动测试
├── skills/
│   ├── README.md                    # 两份 Skill 与作品集总览
│   ├── checked-collage-panel-extractor/
│   │   ├── extract_checked_panels.py
│   │   ├── requirements.txt
│   │   ├── tests/
│   │   ├── examples/                # 演示图、作品集与来源记录
│   │   ├── agents/openai.yaml
│   │   ├── SKILL.md
│   │   └── README.md
│   └── heritage-photo-collage/
│       ├── references/design-guide.md
│       ├── agents/openai.yaml
│       ├── SKILL.md
│       └── README.md
├── CHANGELOG.md
├── CONTRIBUTING.md
├── LICENSE
└── README.md
```

## 技术栈

- Python 3.10+
- [Pillow](https://python-pillow.org/)：图像读取、EXIF 方向处理、裁切、缩放与保存
- [NumPy](https://numpy.org/)：像素差分、颜色掩膜与边缘统计
- `unittest`：无需额外测试框架的自动化测试
- GitHub Actions：在受支持的 Python 版本上持续运行测试

拆图流程为本地确定性处理，不需要 API Key，也不会自动调用图像生成模型。Stoneframe 只有在用户明确要求制作图片且客户端具备相应工具时才进行生成。

## 当前状态

项目处于可用的早期开源阶段。核心拆图命令、手动选择、批处理、数量校验、报告和 ZIP 已实现，并有自动测试覆盖主要安全边界。作品集与两份 Agent Skill 已整理完成。

目前没有图形界面、自动网格识别、透视校正或生成式超分辨率。仓库尚未发布 PyPI 包或独立安装程序。

## 已知限制

- 只支持等宽、等高的规则网格；不规则拼贴需先在其他工具中取得明确裁切区域；
- 标记检测识别的是新增亮青色/蓝色像素，不理解“对勾”语义；新增青色文字或跨格笔迹可能误选；
- 带标记版与无标记版必须位置一致。脚本可处理等比例缩放，但不能自动修正平移、旋转、裁切差异或透视变化；
- 默认阈值来自当前示例，压缩严重、标记过细或色彩不同的图片可能需要调整 `--diff-threshold`、`--min-ratio` 或 `--min-pixels`；
- `--square-mode crop` 会丢失边缘内容；`--trim-white` 也可能移除作品本身的白边；
- 输出放大不会增加真实细节，仅便于后续展示；
- Stoneframe 是创作方法与提示词 Skill，不包含固定图像模型或在线服务。

## FAQ

### 为什么需要两张联系表？

带标记版只负责说明“选哪些格子”，无标记版负责提供最终像素。这样导出的图片不会残留勾选笔迹。

### 原图里本来就有蓝色，会不会被选中？

脚本同时检查颜色和两张图片之间的像素差异。两张图中都存在的蓝色内容通常会被排除，但新增加的青色文字、线条或跨格标记仍可能触发检测。

### 为什么数量不对时什么也没有导出？

这是有意的批次保护。使用 `--expected` 后，脚本会先完成全批检测和数量核对，再写入文件，避免得到一组不完整但看似成功的结果。

### 能处理不规则拼贴或自动识别网格吗？

当前不能。命令行工具按 `--rows` 和 `--cols` 等分图片；不规则布局需要先手动测量并使用其他裁切工具。自动网格识别仍在 Roadmap 中。

### 可以直接从带勾图片裁切吗？

不建议。那会把标记一起保留。若没有无标记版，请先取得原始联系表；脚本不会用生成式修补冒充精确裁切。

## Roadmap

- [x] 青色标记差分检测与规则网格裁切
- [x] 手动行列、批处理清单、数量校验与覆盖保护
- [x] JSON 报告、ZIP 输出、示例素材与自动化测试
- [ ] 更清晰的命令行诊断与阈值调试输出
- [ ] 自动网格/分隔线检测
- [ ] 可选的预览图与选中格可视化
- [ ] 更丰富的批处理配置与跨平台安装方式
- [ ] 在真实设备与更多联系表样式上补充回归样例

Roadmap 表示方向，不代表已经承诺发布日期。欢迎先通过 Issue 讨论需求和样例。

## 贡献

欢迎修正文档、补充测试或改进检测稳定性。提交前请阅读 [CONTRIBUTING.md](CONTRIBUTING.md)，并运行：

```sh
python -m unittest discover -s skills/checked-collage-panel-extractor/tests -v
```

请勿在 Issue 或 Pull Request 中上传含有隐私、未授权人物或未获许可传播的原始照片。必要时请提供裁剪、打码或自行构造的最小复现样例。

## Issues / Bug 反馈

如果遇到误选、漏选、命令报错或文档问题，请通过 [GitHub Issues](https://github.com/Xiaotian69/checked-collage-panel-extractor/issues/new/choose) 提交，并尽量附上：

- 操作系统与 Python 版本；
- 完整运行命令和错误输出；
- 预期格子与实际格子；
- 已脱敏的最小示例，或 `selection_report.json`；
- 是否修改过检测阈值。

## License

代码与文档采用 [MIT License](LICENSE)。仓库中的摄影原片与 AI 作品用于展示工作流和视觉来源，其复用许可尚未单独声明；如需转载或用于其他项目，请先联系作者确认。素材对应关系与处理记录见 [gallery_manifest.json](skills/checked-collage-panel-extractor/examples/gallery_manifest.json)。

## English summary

PanelPick detects newly added bright cyan/blue annotations on a marked contact sheet, then exports the corresponding cells from a matching clean sheet. It supports manual cell coordinates, batch manifests, expected-count validation, square padding/cropping, JSON reports and ZIP output. The extraction pipeline runs locally and does not require an image-generation service or API key.
