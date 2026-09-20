## 第一周：认识网络流量数据并搭好环境
### 本周目标（9.21~9.27）
1. 需要掌握的网络知识
   1. 数据包与网络流
   packet, flow, 五元组
   2. TCP与UDP的基本区别
   3. 常见的流量特征
2. 编程任务
   1. 建立Python环境
   学会启动Jupyter，运行pandas，读取CSV文件，会保存依赖到requirements.txt
   2. 下载并抽取小规模数据
   使用UNSW-NB15的训练集CSV，第一周只读取前1万～5万条数据，完成以下操作：
   ```
   df.head()
   df.shape
   df.columns
   df.info()
   df.describe()
   df["label"].value_counts()
   ```
   3. 完成探索性分析
   至少画出：
   * 正常流量和异常流量的数量；
   * 不同协议的数量；
   * 不同攻击类型的数量；
   * 流量持续时间的分布；
   * 发送字节数的分布