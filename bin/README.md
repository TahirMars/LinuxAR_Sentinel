# LinuxAR Sentinel 工具操作说明

## 执行命令
无参数执行（使用默认值）：

```bash
./rk-agent
```

默认等价于（参考 `-h`）：

```bash
./rk-agent --output-dir output --window-sec 90 --mode standard
```

查看完整参数说明：

```bash
./rk-agent -h
```

标准执行：

```bash
./rk-agent --output-dir output --window-sec 90
```

需要验证 eBPF 挂载时（推荐）：

```bash
sudo ./rk-agent --output-dir output --window-sec 90
```

静默模式（仅输出最终报告）：

```bash
./rk-agent --output-dir output --window-sec 90 --quiet
```

使用外部 BPF 对象覆盖内嵌对象：

```bash
sudo RK_EBPF_OBJ=/abs/path/rk_collector.bpf.o ./rk-agent --output-dir output --window-sec 90
```

## 输出说明
每次执行会生成目录：`output/case-<id>/`。

关键文件：
- `summary.json`：本次采集摘要（模式、窗口、采集计数、附着状态）。
- `findings.json`：异常列表（等级、检测器、异常点、证据）。
- `verdict.json`：最终风险等级与处置建议。
- `modules.json`：模块可见性与跨视图差异。
- `network.json`：网络连接可见性与关联结果。
- `bpf.json`：eBPF 资产与可疑规则命中情况。

终端重点字段：
- `Risk`：整体风险等级。
- `Findings`：异常数量统计（critical/high/medium/info）。
- `Collector`：eBPF 是否成功附着及事件计数。
