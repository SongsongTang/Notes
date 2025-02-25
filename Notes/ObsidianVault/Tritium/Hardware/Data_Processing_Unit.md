---
date: 2024-07-20
aliases:
  - 数据处理单元
  - DPU
tags:
  - 硬件设计
---
# DPU 硬件整体架构
[DPU_v0.drawio - draw.io (diagrams.net)](https://app.diagrams.net/#DDPU_v0.drawio#%7B%22pageId%22%3A%22ROOT%22%7D)
![[Pasted image 20240720173736.png]]

# DPU 电源方案
[DPU_v0.drawio - draw.io (diagrams.net)](https://app.diagrams.net/#DDPU_v0.drawio#%7B%22pageId%22%3A%22gWsKFA_pZpgCnYuY6tbw%22%7D)
![[Pasted image 20240720173904.png]]

# DPU 时钟方案
[DPU_v0.drawio - draw.io (diagrams.net)](https://app.diagrams.net/#DDPU_v0.drawio#%7B%22pageId%22%3A%22Clock%22%7D)
![[Pasted image 20240720173945.png]]
# DPU逻辑设计
[DPU_v0.drawio - draw.io (diagrams.net)](https://app.diagrams.net/#DDPU_v0.drawio#%7B%22pageId%22%3A%22Logic%22%7D)
![[Pasted image 20240720173150.png]]
逻辑实现的功能
![[Pasted image 20240720173227.png]]
数据和命令流向

# DPU 数据打包格式
[DPU_v0.drawio - draw.io (diagrams.net)](https://app.diagrams.net/#DDPU_v0.drawio#%7B%22pageId%22%3A%22SR0ij5yvw7FiCGDHh4l4%22%7D)
![[Pasted image 20240720174208.png]]
# 寄存器表
| 控制对象 | 地址   | 数据       | 描述                        |
| ---- | ---- | -------- | ------------------------- |
| 复位   | 0000 | 0001     | 系统复位                      |
|      | 0002 | 0001     | PLL复位                     |
|      | 0004 | 0001     | Eth复位                     |
|      | 0006 | 0001     | DDR3复位                    |
| 配置类型 | 1000 | X1XX1111 | bit6 广播配置，bit[3:0] 需配置的板号 |
|      | 1002 | XXX11111 | bit4                      |
