# XuCPU

实现一个 32 位 LoongArch CPU，支持 LoongArch-C3 指令集的 25 条指令：`ADDI.W, ADD.W, SUB.W, LU12I.W, PCADDU12I, OR, ORI, ANDI, AND, XOR, SRLI.W, SLLI.W, JIRL, B, BEQ, BNE, BL, ST.W, LD.W, ST.B, LD.B, MUL.W, SLTI, SRL.W, BLTU`。

该 CPU 为经典五级流水线 CPU，包括取指、译码、执行、访存和写回五个阶段；单发射，无 Cache，无分支预测，无乱序执行。

碍于笔者暑期时间有限且事务繁忙，投入龙芯杯个人赛开发的时间较少，该 CPU 表现一般，架构上也存在诸多可以改进的地方。希望后来者能借此有所启发。

## 项目结构

```
.
├── .ci-scripts/                        # CI 脚本（由发布包提供）
├── asm/                                # 决赛使用的汇编代码
├── judge/                              # 比赛使用的评测代码
├── thinpad_top.srcs/                   # Vivado 项目文件
│   ├── constrs_1/new/thinpad_top.xdc   # 约束文件
│   ├── sim_1/                          # 仿真所需文件（由发布包提供）
│   └── sources_1/                      # 源码文件
│       ├── ip/                         # IP 核（由 Vivado 自动生成）
│       ├── new/                        # Verilog 源码（由发布包提供）
│       └── xucpu/                      # XuCPU 源码
├── .gitignore                          # Git 忽略文件（由发布包提供）
├── .gitlab-ci.yml                      # GitLab CI 配置文件（由发布包提供）
├── .gitmodules                         # Git 子模块配置文件（由发布包提供）
├── LICENSE                             # 开源许可证（GPLv3）
├── README.md                           # 本文件
└── thinpad_top.xpr                     # Vivado 项目文件（由发布包提供）
```

## Milestone

| Freq  | STREAM | MATRIX | CRYPTONIGHT | Final  |
| :---: | :----: | :----: | :---------: | :----: |
| 50MHz | 0.126s | 0.197s |   0.493s    |        |
| 55MHz | 0.114s | 0.179s |   0.448s    |        |
| 56MHz | 0.112s | 0.176s |   0.440s    |        |
| 57MHz | 0.110s | 0.173s |   0.432s    | 0.097s |

## LoongArch 指令集

### 寄存器

|   编号 | Normal Name | LP64 Name |
| -----: | ----------: | --------: |
|  `0x0` |       `$r0` |   `$zero` |
|  `0x1` |       `$r1` |     `$ra` |
|  `0x2` |       `$r2` |     `$tp` |
|  `0x3` |       `$r3` |     `$sp` |
|  `0x4` |       `$r4` |     `$a0` |
|  `0x5` |       `$r5` |     `$a1` |
|  `0x6` |       `$r6` |     `$a2` |
|  `0x7` |       `$r7` |     `$a3` |
|  `0x8` |       `$r8` |     `$a4` |
|  `0x9` |       `$r9` |     `$a5` |
|  `0xa` |      `$r10` |     `$a6` |
|  `0xb` |      `$r11` |     `$a7` |
|  `0xc` |      `$r12` |     `$t0` |
|  `0xd` |      `$r13` |     `$t1` |
|  `0xe` |      `$r14` |     `$t2` |
|  `0xf` |      `$r15` |     `$t3` |
| `0x10` |      `$r16` |     `$t4` |
| `0x11` |      `$r17` |     `$t5` |
| `0x12` |      `$r18` |     `$t6` |
| `0x13` |      `$r19` |     `$t7` |
| `0x14` |      `$r20` |     `$t8` |
| `0x15` |      `$r21` |      `$x` |
| `0x16` |      `$r22` |     `$fp` |
| `0x17` |      `$r23` |     `$s0` |
| `0x18` |      `$r24` |     `$s1` |
| `0x19` |      `$r25` |     `$s2` |
| `0x1a` |      `$r26` |     `$s3` |
| `0x1b` |      `$r27` |     `$s4` |
| `0x1c` |      `$r28` |     `$s5` |
| `0x1d` |      `$r29` |     `$s6` |
| `0x1e` |      `$r30` |     `$s7` |
| `0x1f` |      `$r31` |     `$s8` |

### 指令码

<table>
  <tr><th>指令码</th><th>31</th><th>30</th><th>29</th><th>28</th><th>27</th><th>26</th><th>25</th><th>24</th><th>23</th><th>22</th><th>21</th><th>20</th><th>19</th><th>18</th><th>17</th><th>16</th><th>15</th><th>14</th><th>13</th><th>12</th><th>11</th><th>10</th><th>09</th><th>08</th><th>07</th><th>06</th><th>05</th><th>04</th><th>03</th><th>02</th><th>01</th><th>00</th></tr>
  <tr><td>ADDI.W</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>1</td><td>0</td><td colspan="12">si12</td><td colspan="5">rj</td><td colspan="5">rd</td></tr>
  <tr><td>ADD.W</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td colspan="5">rk</td><td colspan="5">rj</td><td colspan="5">rd</td></tr>
  <tr><td>SUB.W</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td colspan="5">rk</td><td colspan="5">rj</td><td colspan="5">rd</td></tr>
  <tr><td>LU12I.W</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>1</td><td>0</td><td colspan="20">si20</td><td colspan="5">rd</td></tr>
  <tr><td>PCADDU12I</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>1</td><td>0</td><td colspan="20">si20</td><td colspan="5">rd</td></tr>
  <tr><td>OR</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>0</td><td colspan="5">rk</td><td colspan="5">rj</td><td colspan="5">rd</td></tr>
  <tr><td>ORI</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>1</td><td>0</td><td colspan="12">si12</td><td colspan="5">rj</td><td colspan="5">rd</td></tr>
  <tr><td>ANDI</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td><td>1</td><td colspan="12">si12</td><td colspan="5">rj</td><td colspan="5">rd</td></tr>
  <tr><td>AND</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>1</td><td>0</td><td>0</td><td>1</td><td colspan="5">rk</td><td colspan="5">rj</td><td colspan="5">rd</td></tr>
  <tr><td>XOR</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td colspan="5">rk</td><td colspan="5">rj</td><td colspan="5">rd</td></tr>
  <tr><td>SRLI.W</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>1</td><td colspan="5">ui5</td><td colspan="5">rj</td><td colspan="5">rd</td></tr>
  <tr><td>SLLI.W</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td colspan="5">ui5</td><td colspan="5">rj</td><td colspan="5">rd</td></tr>
  <tr><td>JIRL</td><td>0</td><td>1</td><td>0</td><td>0</td><td>1</td><td>1</td><td colspan="16">offs[15:0]</td><td colspan="5">rj</td><td colspan="5">rd</td></tr>
  <tr><td>B</td><td>0</td><td>1</td><td>0</td><td>1</td><td>0</td><td>0</td><td colspan="16">offs[15:0]</td><td colspan="10">offs[25:16]</td></tr>
  <tr><td>BEQ</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>0</td><td colspan="16">offs[15:0]</td><td colspan="5">rj</td><td colspan="5">rd</td></tr>
  <tr><td>BNE</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td colspan="16">offs[15:0]</td><td colspan="5">rj</td><td colspan="5">rd</td></tr>
  <tr><td>BL</td><td>0</td><td>1</td><td>1</td><td>0</td><td>0</td><td>0</td><td colspan="16">offs[15:0]</td><td colspan="10">offs[25:16]</td></tr>
  <tr><td>ST.W</td><td>0</td><td>0</td><td>1</td><td>0</td><td>1</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td><td colspan="12">si12</td><td colspan="5">rj</td><td colspan="5">rd</td></tr>
  <tr><td>LD.W</td><td>0</td><td>0</td><td>1</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td colspan="12">si12</td><td colspan="5">rj</td><td colspan="5">rd</td></tr>
  <tr><td>ST.B</td><td>0</td><td>0</td><td>1</td><td>0</td><td>1</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td colspan="12">si12</td><td colspan="5">rj</td><td colspan="5">rd</td></tr>
  <tr><td>LD.B</td><td>0</td><td>0</td><td>1</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td colspan="12">si12</td><td colspan="5">rj</td><td colspan="5">rd</td></tr>
  <tr><td>MUL.W</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td><td>1</td><td>0</td><td>0</td><td>0</td><td colspan="5">rk</td><td colspan="5">rj</td><td colspan="5">rd</td></tr>
  <tr><td>SLTI</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>0</td><td>0</td><td colspan="12">si12</td><td colspan="5">rj</td><td colspan="5">rd</td></tr>
  <tr><td>SRL.W</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td><td>1</td><td>1</td><td>1</td><td>1</td><td colspan="5">rk</td><td colspan="5">rj</td><td colspan="5">rd</td></tr>
  <tr><td>BLTU</td><td>0</td><td>1</td><td>1</td><td>0</td><td>1</td><td>0</td><td colspan="16">offs[15:0]</td><td colspan="5">rj</td><td colspan="5">rd</td></tr>
</table>
