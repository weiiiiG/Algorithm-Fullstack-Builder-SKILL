# 算法分析参考

当算法主文件很大、结构混乱，或还没有被封装成函数时，使用本参考快速提取契约。

## 入口发现

优先搜索：

- `if __name__ == "__main__"`
- `main(`
- `train(`
- `predict(`
- `process(`
- `run_`
- `to_csv`、`save`、`dump`、`torch.save`、`plt.savefig`
- 硬编码输入路径
- 硬编码输出路径

## 类型判断

- 训练：包含 epoch、optimizer、fit、validation、loss、metric 或模型保存。
- 推理：加载模型，接受样本，输出标签、分数或预测值。
- 处理：转换文件或表格，并写出清洗、合并、筛选后的结果。
- 优化：搜索参数、目标函数、约束、最优值或收敛过程。
- 仿真：运行场景、步骤、回合或重复实验。
- 报表：聚合数据并生成图表、文档、表格或演示。

## 契约记录模板

```text
算法类型:
运行模式:
主入口:
输入:
参数:
必需文件:
输出:
产物:
外部依赖:
本地依赖:
失败模式:
```

## 重构规则

- 保留原算法数学逻辑和模型行为。
- 将硬编码路径改成参数。
- 将 `print` 输出转成结构化日志或摘要。
- 将生成文件统一写入 `artifacts_dir`。
- 只有前端需要静态图片时才保留绘图产物。
- 原算法依赖可复现性时，保留或显式设置随机种子。
- CPU/GPU 选择要在配置或结果摘要中可见。

## 结果映射示例

训练结果：

```python
{
    "summary": {"rows": 1000, "device": "cuda", "target": "close"},
    "metrics": {"mae": 0.1, "rmse": 0.2},
    "series": [
        {
            "name": "prediction",
            "points": [{"x": "2024-01-01", "actual": 1, "predicted": 1.1}],
        }
    ],
    "tables": {},
    "artifacts": [{"type": "model", "file_name": "model.pt"}],
    "logs": [{"level": "info", "message": "training completed"}],
}
```

处理结果：

```python
{
    "summary": {"input_rows": 1000, "output_rows": 950},
    "metrics": {"missing_rate": 0.03},
    "series": [],
    "tables": {"preview": [{"col": "value"}]},
    "artifacts": [{"type": "processed_csv", "file_name": "processed.csv"}],
    "logs": [],
}
```

推理结果：

```python
{
    "summary": {"model": "best.pt", "samples": 5},
    "metrics": {"avg_confidence": 0.93},
    "series": [],
    "tables": {"predictions": [{"label": "A", "confidence": 0.98}]},
    "artifacts": [],
    "logs": [],
}
```
