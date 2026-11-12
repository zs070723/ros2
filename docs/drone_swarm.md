# 无人机集群编队控制 (Leader-Follower + Consensus)

> 本文档对应模块: `src/air/drone_swarm/`

## 场景介绍

多架无人机在室外/室内环境中协同完成**编队飞行任务**是集群协同的基础。
本模块演示 5 架无人机在 **Leader-Follower** 架构下，通过 **Consensus 一致性算法**
在保持期望相对偏移的同时，实现**全局协同与队形保持**。

与仓库中已有的单体无人机飞行、距离制动、幕墙扫描等模块不同，
本模块专注于**多机协同控制算法**，不依赖 AirSim/SITL，可直接运行。

## 运行

```powershell
# 依赖: Python 3.10+, NumPy
cd src/air/drone_swarm
python main.py
```

## 核心算法

### Leader-Follower 架构

```
        ╱╲                  ╱╲
       ╱  ╲                ╱  ╲
      ╱  F2 ╲   F0(Leader) ╱  F3 ╲
     ╱──────╲   ─────────  ╱──────╲
              ╲         ╱
        F4     ╲       ╱      F1
                 ╲    ╱
                  ╲  ╱
```

### Consensus + 队形反馈

每台 Follower 的控制量由两项组成：

```
u = K_shape * (Leader_pos + Desired_offset - my_pos)   # 队形误差
  + K_consensus * Σ (neighbor_pos - my_pos - relative_offset) / N   # 一致性项
```

| 项 | 作用 | 增益 |
|----|------|------|
| `K_shape` | 拉向期望位置 | 1.2 |
| `K_consensus` | 邻居间达成协议 | 0.5 |

### 支持的队形

| 队形 | 偏移生成方式 | 适用数量 |
|------|-------------|---------|
| `triangle` | 圆周均匀分布 | 3+ |
| `diamond` | 菱形四角 | 4 |
| `line` | 直线等距 | 任意 |

### 支持的通信拓扑

| 拓扑 | 每架邻居 | 延迟/鲁棒性 |
|------|---------|------------|
| `ring` | 前后各 1 | 低延迟，中鲁棒 |
| `fully_connected` | 所有其他 | 高冗余，高延迟 |

## 参数调优

编辑 `src/air/drone_swarm/scenario.json`：

| 参数 | 说明 | 默认 | 调大调小 |
|------|------|------|---------|
| `k_shape` | 队形反馈增益 | 1.2 | 大调得快但易振荡 |
| `k_consensus` | 一致性增益 | 0.5 | 大调得太硬，小调不协同 |
| `formation` | 队形类型 | triangle | diamond / line |
| `topology` | 通信拓扑 | ring | fully_connected |

## 输出示例

```
=======================================================
  无人机集群编队控制演示
  数量: 5  队形: triangle  拓扑: ring
=======================================================
  Step    0 | 队形误差: 3.2145 m | 领导者: [0. 0.]
  Step   50 | 队形误差: 1.4231 m | 领导者: [15. 0.]
  Step  100 | 队形误差: 0.5872 m | 领导者: [30. 0.]
  ...
✅ 演示结束
  初始队形误差: 3.2145 m
  最终队形误差: 0.0412 m
```

## 可扩展方向

- 障碍物避障（RRT* / A*）
- 动态队形切换（任务驱动）
- 丢包通信下的鲁棒 Consensus
- 与 PX4 SITL + DDS 桥接真实飞控

## 依赖

- Python 3.10+
- NumPy
