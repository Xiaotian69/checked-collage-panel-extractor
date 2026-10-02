# 参与贡献

感谢你愿意改进 PanelPick 与 Stoneframe。这个仓库目前规模不大，最有帮助的贡献通常是可复现的问题样例、针对边界情况的测试，以及准确、克制的文档修正。

## 开始之前

- Bug、行为变化和较大的新功能，请先创建 Issue 说明场景与预期；
- 小型拼写、链接和措辞修正可以直接提交 Pull Request；
- 不要上传含有隐私、未授权人物或无权公开传播的原始照片；
- 新功能必须在 README 中区分“已实现”和“规划中”，不要提前宣传尚未落地的能力。

## 本地开发

```sh
git clone https://github.com/Xiaotian69/checked-collage-panel-extractor.git
cd checked-collage-panel-extractor

python -m venv .venv
```

激活虚拟环境后安装依赖：

```sh
python -m pip install -r skills/checked-collage-panel-extractor/requirements.txt
```

运行全部测试：

```sh
python -m unittest discover -s skills/checked-collage-panel-extractor/tests -v
```

也可以运行仓库自带示例：

```sh
cd skills/checked-collage-panel-extractor
python extract_checked_panels.py --manifest examples/manifest.example.json --out demo-output --expected 3 --square-mode keep --zip
```

## 提交建议

### 修复检测或裁切问题

请尽量同时提供：

- 一个不包含隐私的最小复现样例；
- 实际运行命令；
- 预期选中行列与实际结果；
- 针对该问题新增或更新的自动化测试；
- 对默认行为、兼容性或输出格式的影响说明。

脚本以“不要悄悄导出错误结果”为重要原则。涉及数量校验、覆盖保护、干净版取像素或坐标边界的修改，需要保留相应安全检查。

### 修改文档或 Skill

- 所有描述必须以当前代码和实际工具能力为准；
- 保留“确定性裁切”和“生成式创作”的边界；
- 不要把作品图库描述为脚本实时生成结果；
- 调整 `SKILL.md` 时，请同步核对对应 README 与 `agents/openai.yaml`；
- 新增图片时，请记录来源、处理方式和可公开使用的授权情况。

## Pull Request 检查清单

- [ ] 改动范围单一，提交信息能够说明目的；
- [ ] 没有提交虚拟环境、缓存、个人输出或敏感素材；
- [ ] 现有测试通过，行为变化有相应测试；
- [ ] README、示例命令和实际实现一致；
- [ ] 新增功能已说明限制，不夸大当前能力；
- [ ] 如涉及素材，已说明来源和授权边界。

## 反馈与讨论

请使用 [GitHub Issues](https://github.com/Xiaotian69/checked-collage-panel-extractor/issues/new/choose) 报告 Bug 或提出功能建议。安全漏洞或可能暴露敏感数据的问题，请不要附带真实隐私素材公开提交；先用最小描述说明影响范围。
