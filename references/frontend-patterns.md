# 前端模式参考

根据算法契约设计和实现前端时使用本参考。

## 页面模型

按工作流创建页面：

- 短算法：执行页。
- 长算法：任务提交页。
- 结构化输出：结果详情页。
- 持久化运行记录：历史页。
- 多次运行指标有价值：对比页或对比面板。

## 组件映射

| 契约输出 | 推荐组件 |
| --- | --- |
| `summary` | 紧凑描述列表或统计条 |
| `metrics` | 指标卡片 |
| `series` | ECharts 折线图、柱状图或散点图 |
| `tables` | 带分页和列显隐的数据表 |
| `artifacts` | 带文件类型图标的下载列表 |
| `logs` | 时间线、日志面板或折叠列表 |
| 图片产物 | 图片预览和下载 |
| 矩阵 | 热力图或密集表格 |

## 输入模式

控件匹配数据类型：

- CSV、Excel、图片、模型文件、压缩包：文件上传。
- 原始文本、prompt、JSON 片段：多行文本框。
- 模式枚举：选择器或分段控件。
- 阈值和批大小：滑块或数字输入。
- 布尔值：开关或复选框。
- 时间窗口：日期范围选择器。
- 多文件目录：目录上传或压缩包上传。

提交前校验：

- 必填文件存在。
- 配置 JSON 可解析。
- CSV 可预览时检查必需列。
- 数值范围与后端 schema 一致。

## 状态体验

异步任务需要展示：

- `pending`、`running`、`completed`、`failed`、`cancelled`
- 已运行时间
- 可取消时显示取消按钮
- 失败后支持重试，成功后支持复用配置重跑
- 后端错误码和简洁错误信息

默认每 1-3 秒轮询一次状态；已有 WebSocket 或 SSE 架构时沿用现有方案。

## 图表规则

- 真实值对预测值：两条线，共享 x 轴，tooltip 显示真实值、预测值、绝对误差、相对误差。
- 训练历史：展示 train/validation loss；有 reward、accuracy 时一起展示。
- 仿真轨迹：折线图加场景选择器。
- 优化任务：收敛曲线加最优参数表。
- 分类任务：类别数量柱状图加预测明细表。
- 数据处理：行数、缺失率等指标卡，加预览表。

大序列使用 `dataZoom`、后端分页或降采样。

## 视觉与布局

首屏直接呈现真实工作台，不做营销落地页。

- 布局紧凑但可读。
- 卡片只用于重复项、弹窗或明确的工具容器。
- 不嵌套卡片。
- 表格和图表尺寸稳定，避免加载或 hover 时跳动。
- 移动端按工作流顺序堆叠：输入、动作/状态、主要结果、详情。
- 控件内文字不能溢出。

## API Client

使用集中式 typed client，不要在组件里分散写 `fetch`。

推荐函数：

```ts
validateInput(payload): Promise<ValidationResult>
execute(payload): Promise<Result>
createJob(payload): Promise<{ job_id: string; status: string }>
getJob(jobId): Promise<Job>
getJobResult(jobId): Promise<Result>
listJobs(params): Promise<JobPage>
cancelJob(jobId): Promise<void>
deleteJob(jobId): Promise<void>
downloadArtifact(jobId, artifactType): string
```

前端类型要与后端响应字段保持一致。
