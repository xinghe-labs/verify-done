# verify-done

通用 AI agent skill：**完成前硬门禁**——没有真实跑过的外部证据，不许宣称"完成"。

适用于任何实现 [Agent Skills 规范](https://agentskills.io) 的 AI 编程助手：**Pi、Claude Code、Codex、Cursor、Windsurf** 等。单文件、零依赖、零跨 skill 引用，装一个文件即获得完整能力闭环。

**解决的问题**：agent 把"写完了"当成"做完了"——没跑测试就说完成、验证失败就绕过去、顺手改了没让改的东西、用真实扣款来"验证"支付代码。

## 安装

```bash
# Pi
pi install github:xinghe-labs/verify-done
```

```bash
# Claude Code / 其他 harness
npx skills add xinghe-labs/verify-done
```

或直接复制 [SKILL.md](SKILL.md) 到你的 skills 目录。

## 核心机制

- **三个触发锚点**：标记 todo 完成前 / 最终汇报前 / 说出禁语（"应该可以了"）时
- **验证阶梯**（按改动规模伸缩）：静态检查 → 构建 → 针对性测试 → 行为复现 → 自读 diff，每级配有无基建时的降级手段
- **副作用红线**：验证永远不许真实发邮件/扣款/调付费 API——必须 dry-run / sandbox / mock
- **范围护栏（内置）**：编辑前声明改动范围；依赖、公共 API、schema、CI 属红档，必须先问
- **反逃逸**：报"unverified"必须附尝试记录 + 缺失说明 + 用户自行验证的最短路径
- **用户覆盖**：用户明确说"别验证"则照办，但汇报必须留痕"per your request, unverified"
- **范围精确汇报**：Verified 必须列出跑了什么命令、结果、没覆盖什么

## 设计原则

- **比例原则**：改一个 typo 只需静态检查 + 读 diff，不触发全量构建
- **超长构建例外**：单级几十分钟的检查可记录"末次全绿时间"代替阻塞式重跑，但必须具名
- **自洽单文件**：无任何跨 skill 依赖，不要求安装其他配套 skill
- **全环境覆盖**：有/无测试基建、有/无 git、dev server、CLI、notebook、视觉改动各有对应验证手段
- **迭代实证**：经三轮自我拷问 + 8 类用户画像（电商/运维/数据科学/独立开发/前端/移动端等）走查修出 9 个缺口

## 边界（刻意不做）

- 不管 diff 质量审查（代码写得好不好）——交给 code-review 类 skill
- 不管怎么修 bug——交给 TDD / 诊断类 skill
- 不管非代码产出（文档、调研）

## License

MIT
