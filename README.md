# ai-test-orchestrator

> AI 测试流水线总控编排器 —— 一句话发起，自动调度多个专业测试 Skill，完成从测试设计到自愈修复的完整质量闭环。

## 这是什么

`ai-test-orchestrator` 是一个 Agent Skill（总控 MetaSkill）。它本身**不写测试、不修 Bug**，专职调度与决策：解析你的测试目标 → 按风险分级设计用例 → 人工门禁确认 → 调度 TDD / Playwright / Webapp Testing 执行 → 失败时调用 Systematic Debugging 定位根因 → 自动修复回归（最多 3 轮）→ 输出质量报告。

适合场景：

- 「帮我测一下这个网站/模块/功能」
- 「跑一轮自动化测试」
- 「这个失败的测试帮我修一下」

## 流水线 DAG

```
① 解析目标 → ② 上下文收集 → ③ 风险分析 → ④ 测试设计
→ ⑤【人工门禁】→ ⑥ TDD 测试实现 → ⑦ 测试执行
→ ⑧ 结果分析 ─┬─ 全部通过 → ⑫ 质量度量 → ⑬ 质量门禁 → 报告结束
              └─ 存在失败 → ⑨ 根因定位 → ⑩ 自愈判断
                            ├─ 置信度高 → ⑪ 自动修复 → 回到 ⑦ 回归
                            └─ 置信度低 → 转人工：输出缺陷单+证据，暂停等待
```

## 依赖的子 Skill

本 Skill 通过标准输入输出契约调度以下子 Skill，请先安装：

| 阶段 | Skill | 安装命令 |
|---|---|---|
| 测试实现（TDD 先行） | `obra/superpowers@test-driven-development` | `npx skills add obra/superpowers@test-driven-development -g -y` |
| 测试执行（浏览器） | `obra/superpowers@webapp-testing` | `npx skills add obra/superpowers@webapp-testing -g -y` |
| 测试执行（Playwright） | `alinaqi/maggy@playwright-testing` | `npx skills add alinaqi/maggy@playwright-testing -g -y` |
| 根因定位 | `obra/superpowers@systematic-debugging` | `npx skills add obra/superpowers@systematic-debugging -g -y` |

## 安装本 Skill

```bash
# 方式一：skills CLI（替换为你的 GitHub 仓库地址）
npx skills add <your-name>/<your-repo>@ai-test-orchestrator -g -y

# 方式二：手动安装（本仓库即技能目录）
git clone https://github.com/<your-name>/<your-repo>.git "$HOME/.agents/skills/ai-test-orchestrator"
```

安装后重启 Agent 会话即可生效。

## 使用方式

对 Agent 说一句话即可触发：

```
用 ai-test-orchestrator 对 https://example.com 跑一轮冒烟测试
用 ai-test-orchestrator 测试登录和订单模块
这个失败的测试帮我定位并修复
```

## 全局策略（不可违反）

1. **自愈上限 3 轮**：修复 → 回归最多循环 3 次，超限强制转人工，杜绝死循环
2. **人工门禁不可跳过**：测试用例清单未经确认，不得进入实现阶段
3. **证据先于结论**：任何通过/失败结论必须有测试输出或截图支撑
4. **转人工要给全**：缺陷单 = 复现步骤 + 失败证据 + 根因分析 + 修复建议
5. **数据安全**：写操作类用例走「创建 → 验证 → 清理」闭环，危险操作（删除/重置）只验证存在、不真实执行

## 阶段输入输出契约

| 阶段 | 输入 | 结构化输出 |
|---|---|---|
| 上下文收集 | 分支/模块/环境 | 变更影响面、高风险模块清单 |
| 风险分析 | 影响面 + 历史缺陷 | 模块 → 策略（full/regression/smoke） |
| 测试设计 | 策略 + 需求 | 用例清单：前置条件、步骤、预期、优先级 |
| 人工门禁 | 用例清单 | 通过 / 驳回 + 原因 |
| 测试实现 | 用例清单 | 失败测试代码 + 执行脚本 |
| 测试执行 | 脚本 + 环境 | 结果 JSON、截图、日志、Trace |
| 根因定位 | 失败结果 | 分类、置信度 0-1、修复建议 |
| 质量度量 | 全部结果 | 通过率、覆盖率、自愈成功率、耗时 |

## 实战验证

已对线上站点完成全功能回归：**40/40 用例通过**（认证、工作台、智能问答含 RAG 实问实答、统计报表、知识库管理、网页抓取闭环、系统日志、系统设置、用户管理增删闭环、个人信息），自愈 3 轮修复 6 个竞态用例后全绿。

## 仓库结构

```
ai-test-orchestrator/
├── SKILL.md      # 技能主文件（Agent 加载入口）
└── README.md     # 本说明文件
```

## License

MIT
