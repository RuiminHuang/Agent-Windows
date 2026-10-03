## 启动示例

### 在 Claude Code 对话里输入，由主会话派发子 Agent：

```shell
用 grsl-search 子 Agent 运行：year_start=2021, year_end=2025, collection="GRSL2021-2025"
用 grsl-search 子 Agent 运行：year_start=2026, year_end=2026, collection="GRSL2026"
用 j-stars-search 子 Agent 运行：year_start=2021, year_end=2025, collection="J-STARS2021-2025"
用 tgrs-search 子 Agent 运行：year_start=2021, year_end=2021, collection="TGRS2021"
用 tgrs-search 子 Agent 依次处理 2021–2025 年：每年运行一次，year_start=year_end=该年，collection="TGRS该年"，上一年跑完再跑下一年
用 ipt-search 子 Agent 运行：year_start=2002, year_end=2025, collection="IPT2002-2025"
用 ipt-search 子 Agent 运行：year_start=2026, year_end=2026, collection="IPT2026"，重新检索
用 other-elsevier-search 子 Agent 运行：year_start=2026, year_end=2026
用 other-elsevier-search 子 Agent 运行：year_start=2026, year_end=2026，只处理 PR、EAAI
用 other-ieee-search 子 Agent 运行：year_start=2021, year_end=2025
```


### 在 PowerShell 里用命令行无人值守运行，先 cd E:\ResearchProject：

```shell
claude -p 'year_start=2021, year_end=2025, collection=GRSL2021-2025' --agent grsl-search --max-turns 1000 --output-format text > grsl-2021-2025.txt
claude -p 'year_start=2002, year_end=2025, collection=IPT2002-2025' --agent ipt-search --max-turns 1000 --output-format text > ipt-2002-2025-run1.txt
claude -p 'year_start=2021, year_end=2025' --agent other-ieee-search --max-turns 1000 --output-format text > other-ieee-2021-2025.txt

# TGRS 按年依次运行
foreach ($y in 2021..2025) {
  claude -p "year_start=$y, year_end=$y, collection=TGRS$y" --agent tgrs-search --max-turns 1000 --output-format text > "tgrs-$y.txt"
}
```


## Journal Abbreviation Explanation

| # | No. | abbreviation | full name | publisher | IF (JCR 2025) |
|---|---|---|---|---|---|
| 1 | 1 | IF | Information Fusion | Elsevier | 17.4 |
| 2 | 2 | ISPRS | ISPRS Journal of Photogrammetry and Remote Sensing | Elsevier | 12.9 |
| 3 | 3 | AEI | Advanced Engineering Informatics | Elsevier | 11.5 |
| 4 | 4 | ESWA | Expert Systems with Applications | Elsevier | 9.4 |
| 5 | 5 | PR | Pattern Recognition | Elsevier | 9.1 |
| 6 | 6 | EAAI | Engineering Applications of Artificial Intelligence | Elsevier | 9.0 |
| 7 | 7 | JAG | International Journal of Applied Earth Observation and Geoinformation | Elsevier | 8.2 |
| 8 | 8 | KBS | Knowledge-Based Systems | Elsevier | 8.0 |
| 9 | 9 | ASOC | Applied Soft Computing | Elsevier | 7.8 |
| 10 | 10 | Neural Networks | Neural Networks | Elsevier | 7.2 |
| 11 | 11 | Neurocomputing | Neurocomputing | Elsevier | 6.5* |
| 12 | 12 | INS | Information Sciences | Elsevier | 6.0 |
| 13 | 13 | Defence Technology | Defence Technology | KeAi / Elsevier | 5.9* |
| 14 | 14 | Measurement | Measurement | Elsevier | 5.6* |
| 15 | 15 | OLT | Optics and Laser Technology | Elsevier | 4.6* |
| 16 | 16 | IPT | Infrared Physics and Technology | Elsevier | 3.8 |
| 17 | 17 | OLEN | Optics and Lasers in Engineering | Elsevier | 3.7* |
| 18 | 18 | Signal Processing | Signal Processing | Elsevier | 3.6* |
| 19 | 1 | TPAMI | IEEE Transactions on Pattern Analysis and Machine Intelligence | IEEE | 20.4 |
| 20 | 2 | TIP | IEEE Transactions on Image Processing | IEEE | 15.3 |
| 21 | 3 | GRSM | IEEE Geoscience and Remote Sensing Magazine | IEEE | 13.7 |
| 22 | 4 | TCSVT | IEEE Transactions on Circuits and Systems for Video Technology | IEEE | 10.8 |
| 23 | 5 | TMM | IEEE Transactions on Multimedia | IEEE | 9.9 |
| 24 | 6 | TGRS | IEEE Transactions on Geoscience and Remote Sensing | IEEE | 9.4 |
| 25 | 7 | TNNLS | IEEE Transactions on Neural Networks and Learning Systems | IEEE | 8.9* |
| 26 | 8 | IOTJ | IEEE Internet of Things Journal | IEEE | 8.9* |
| 27 | 9 | TITS | IEEE Transactions on Intelligent Transportation Systems | IEEE | 8.4* |
| 28 | 10 | TIM | IEEE Transactions on Instrumentation and Measurement | IEEE | 7.0 |
| 29 | 11 | J-STARS | IEEE Journal of Selected Topics in Applied Earth Observations and Remote Sensing | IEEE | 6.3 |
| 30 | 12 | TAES | IEEE Transactions on Aerospace and Electronic Systems | IEEE | 5.7* |
| 31 | 13 | GRSL | IEEE Geoscience and Remote Sensing Letters | IEEE | 4.8 |
| 32 | 14 | Sensors Journal | IEEE Sensors Journal | IEEE | 4.5* |
| 33 | 15 | SPL | IEEE Signal Processing Letters | IEEE | 3.9* |
| 34 | 1 | IJCV | International Journal of Computer Vision | Springer | 10.3 |
