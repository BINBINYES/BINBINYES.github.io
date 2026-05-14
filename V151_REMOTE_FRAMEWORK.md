# V151 Remote Framework

## 定位

远端页面是 V151 桌面模型的公网观察端，只展示模拟仓状态、模型动作、已卖出订单和远端可用状态。远端不负责计算、不负责下单、不暴露内部行情源细节。

## 模块分工

| 模块 | 责任 |
| --- | --- |
| `v151_ai_realtime_server.py` | 本地唯一计算与同步中枢 |
| `simulate_paper_trading` | 真实模拟仓自动买入、止损卖出、T+1 拦截 |
| `v151_sim_state.json` | 本地模拟仓状态、两天操作历史、已卖出订单 |
| `build_remote_status_payload` | 生成远端标准快照 |
| `remote_sync_github_status` | 将标准快照推送到 GitHub Pages `status.json` |
| GitHub Pages `index.html` | 只负责展示 `status.json` |

## 数据契约

所有远端快照必须带：

```json
{
  "schema_version": "v151.remote.v1",
  "generated_at": "YYYY-MM-DD HH:mm:ss",
  "positions": [],
  "operation_history_2d": [],
  "closed_orders": [],
  "remote_quote_available": true
}
```

## 展示规则

- 远端只显示一个行情可用绿灯，不显示具体行情源名称。
- 远端不显示 `auth_missing`、token、GitHub 错误、内部路径。
- 操作历史必须中文化，不允许裸露内部枚举，如 `MODEL_AUTO_SELL`。
- 已卖出订单必须进入 `closed_orders`，不能混在普通操作历史里冒充买卖历史。
- 风险提示只展示给用户有意义的中文原因。

## 模拟仓规则

- 模拟仓允许自动买入/卖出。
- 真实券商账户不连接、不下单。
- 默认止损线：`-8%`。
- 严格止损线：`-10%`。
- 当日买入默认 T+1 锁定，触发止损时记录 `T+1锁定，止损未成交`。
- 用户授权 `allow_t1_override` 后，才允许强制更改模拟仓。

## 同步规则

- 打开本地监控台后，本地模型默认尝试同步远端。
- 远端同步需要本地 GitHub 写入凭证。
- 凭证缺失只在本地设置/诊断中显示，不进入主界面和远端页面。
- 同一快照不重复推送；默认最小推送间隔 60 秒。

## 扩展规则

- 新增字段必须先加入 `v151_remote_status_schema.json`。
- 页面只能读取契约字段，不直接依赖本地内部字段。
- 所有内部英文枚举必须通过展示层映射成中文。
