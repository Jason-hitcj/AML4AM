资管反洗钱项目


4.1 数据整合
- 关键表关联：
  - 核心表：客户表（DC_CUST）、交易确认表（DC_TRADE_CONF）、反洗钱个人信息表（DC_AML_X1INFO）、反洗钱机构信息表（DC_AML_X3INFO）、账户表（DC_ACCO）。
  - 辅助表：受益人信息表（CUSTANDBNFINFO）、经办人信息表（DC_LINKMAN_INFO）、控股股东信息表（DC_SHAREORCTRL_INFO）。
  - 关联逻辑：通过客户编号（CUST_ID）、基金账号（ACCO_ID）、交易账号（TRADE_ID）等字段构建统一视图，补充客户画像、交易行为、关联网络信息。
- 数据预处理：
  - 缺失值处理：对关键字段（如交易金额、证件有效期）进行填充（均值/众数）或标记为异常。
  - 去重与一致性检查：处理重复交易记录，统一时间格式（如YYYYMMDD），验证主键唯一性。
4.2 特征工程优化（暂定）
1. 交易行为特征：
  - 高频交易标志：单日交易次数 > 阈值（如10次）。
  - 金额异常度：交易金额与客户历史均值的偏离度（Z-Score）。
  - 时间敏感特征：非工作时间（如凌晨0-5点）交易占比、节假日交易频率。
2. 客户画像特征：
  - 收入匹配度：年收入（INCOME_LEV）与累计交易金额的比值，异常高交易占比客户标记。
  - 账户活跃度：账户存续时长、历史交易频率衰减率。
3. 时序特征：
  - 滑动窗口统计：近7天/30天交易次数、金额总和、最大单笔金额。
  - 周期性异常：检测固定时间（如每周末）的资金转入转出模式。
4. 网络关系特征：
  - 资金网络深度：通过交易对手（TAR_DIS_CODE、TAR_TRADE_ID）构建资金流转图，计算节点的入度/出度。
  - 关联账户聚类：基于交易对手、受益人、经办人构建子图，识别密集社区（可能为团伙洗钱）。
5. 新增特征：
  - 账户变更频率：账户信息（如地址、联系人）修改次数。
  - 证件有效期异常：临近过期或已过期的证件占比。
  - 多设备/IP关联：同一客户使用多个设备或IP地址交易的次数。
4.3 模型策略（暂定）
1. 监督学习模型（基线分类器）
基线模型主要用于快速评估特征有效性，同时提供GNN的辅助特征输入。
  - 模型选择
    - LightGBM：适用于高维数据，能够有效处理类别特征，如账户类型、交易方式等。
    - CatBoost：自动处理缺失值，适合类别变量较多的情况，减少数据预处理工作量。
  - 训练策略
    - 交叉验证：使用K折交叉验证（K=5或10）。
    - 类别不平衡处理： 
      - 过采样（如SMOTE）或欠采样（如随机采样）。
      - Focal Loss 处理类别不均衡问题。

2. 图神经网络（GNN）
图神经网络（GNN）能够利用交易数据中的账户间关系，从网络结构和资金流向的角度检测洗钱模式。相比于基线模型，GNN能够捕捉更复杂的多跳交易路径、资金回流模式和社交网络结构，尤其适用于非法资金流动分析。

在GNN建模前，需要将交易数据转换为图结构。
设定：
  - 节点（Nodes）：账户、公司、交易对手等实体。
  - 边（Edges）：交易关系，包括金额、时间、交易类型等信息。

可能使用的GNN模型
暂时无法在飞书文档外展示此内容
可能使用的计算优化方法
  - Cluster-GCN（子图采样）：
    - 只在局部子图上进行训练，降低计算复杂度。
    - 适用于大规模交易网络，能够保证实时检测能力。
  - 边采样（Edge Sampling）：
    - 仅采样部分交易记录，减少计算量，提高模型训练速度。


















