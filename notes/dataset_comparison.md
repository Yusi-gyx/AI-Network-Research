# UNSW-NB15 与 CICIDS2017 数据集对比

根据已阅读的两篇论文及现有笔记整理。最后一行是基于数据生成方式和采集安排提出的潜在风险，尚需通过实验验证。

| 问题 | UNSW-NB15 | CICIDS2017 |
| --- | --- | --- |
| 正常流量如何产生 | 使用 **IXIA PerfectStorm** 在实验网络中生成正常行为流量，与攻击流量共同经过测试网络。 | 使用 **B-Profile Agent**，基于 25 名用户的行为画像生成正常活动，覆盖 HTTP、HTTPS、FTP、SSH 和邮件等协议。生成流量旨在模拟用户行为，其与生产流量的差异仍需验证。[官方说明](https://www.unb.ca/cic/datasets/ids-2017.html) |
| 攻击流量如何产生 | 使用 **IXIA PerfectStorm** 生成与 CVE 漏洞信息相关的攻击场景，包含 9 类攻击，如 DoS、Exploits、Reconnaissance 等。 | 在独立攻击网络中按日程执行攻击工具或脚本，如 Patator 暴力破解、Hulk/Slowloris 等 DoS、Metasploit 渗透、Ares Botnet、Nmap 扫描及 LOIC DDoS。不同攻击安排在不同日期和时段。[论文第 3–4 节](https://pdfs.semanticscholar.org/2342/9b5be933e16e6988da9a322ad95dfdc8c4b0.pdf) |
| 使用什么抓包工具 | 在 **Router 1 上使用 tcpdump** 捕获数据包，保存为 PCAP 文件。 | 使用受害网络主交换机的 **镜像端口** 捕获流量并保存为 PCAP；论文未明确指定具体抓包软件。**CICFlowMeter 用于流提取与分析**，不能据此把它写成实验中的抓包软件。[论文第 3.1 节](https://pdfs.semanticscholar.org/2342/9b5be933e16e6988da9a322ad95dfdc8c4b0.pdf) |
| 怎样聚合为网络流 | **Argus** 从 PCAP 提取流记录，**Bro-IDS** 提取连接和应用层信息；再依据源/目的 IP、源/目的端口、协议等流标识匹配两者的输出，并计算额外连接统计。 | **CICFlowMeter** 将数据包聚合为双向流，首包确定前向方向，反向数据包计入后向统计。连接结束和可配置的超时控制流的边界；具体复现实验需记录提取器版本与参数。[工具说明](https://www.unb.ca/cic/research/applications.html) |
| 使用哪些特征 | 原始版本有 **49 个属性：47 个描述字段与 2 个标签字段**。包括流标识、持续时间、包数/字节数、TTL、状态、服务、TCP/HTTP 信息、到达间隔和近期连接统计（如 `ct_*`）。 | 论文描述 **约 80 项流量特征**，包括持续时间、双向包数与字节数、包长统计、到达间隔（IAT）、TCP 标志、初始窗口及活跃时间等。特征选择使用 `RandomForestRegressor`；具体输入列数应以实际 CSV 为准。[论文第 5 节](https://pdfs.semanticscholar.org/2342/9b5be933e16e6988da9a322ad95dfdc8c4b0.pdf) |
| 如何生成标签 | 将处理后的流记录与 **IXIA 攻击报告生成的 ground-truth 表**匹配；`label` 表示正常/攻击，`attack_cat` 表示攻击类别或正常类别。 | 结合 **攻击执行日程、时间戳、IP、端口及协议**匹配流记录，生成 `Label`，区分 `BENIGN` 与各攻击类别。不能仅凭处于攻击时段就把同时发生的所有流标为攻击。[官方标注说明](https://www.unb.ca/cic/datasets/ids-2017.html) |
| 可能有什么分布偏移 | **时间与攻击强度变化：** 两次采集日期不同，攻击速率由每秒 1 次变为每秒 10 次，可能改变攻击比例与连接统计。**环境变化：** 从生成器和固定拓扑迁移到真实网络时，TTL、服务、流量模式等可能变化，导致模型依赖的规律失效。 | **日期与攻击类别变化：** 周一仅正常流量，其余日期的攻击类型不同；按天划分会同时改变时间、类别组成及攻击比例。**环境与用户行为变化：** 固定攻击主机、服务和工具，以及 B-Profile 生成的正常活动，可能与新网络存在差异，影响模型泛化。 |

特征数量不能直接作为模型输入维度：UNSW-NB15 原始文件与预定义训练/测试 CSV 的字段不同，CICIDS2017 的 CSV 版本也应单独核对。两者的标签都应排除出输入；尤其在预测 UNSW-NB15 的二分类 `label` 时，不能把 `attack_cat` 当作特征。

对分布漂移实验，建议先比较随机划分与按时间或采集场次划分的表现，并报告每个划分中的类别分布。CICIDS2017 的跨日实验还应明确测试集是否包含训练中未出现的攻击类别，以判断性能变化与哪些因素有关。

对应阅读笔记：[UNSW-NB15](papers/unsw_nb15_2015.md)、[CICIDS2017](papers/cic_ids_2017.md)。
