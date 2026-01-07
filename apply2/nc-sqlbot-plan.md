# NC SQLBot 方案（草案）

## 前提假设
- 目标是 NC 财务数据（NC/NCC/BIP），并具备可用的 DW/DIM 层。
- SQLBot 已部署可访问，可进行配置与二次开发。
- 准确度提升聚焦 SQL 生成与小白用户易用性。
- 已参考官方文档：https://dataease.cn/sqlbot/v1/

## 配置工作（按优先级）

### P0 配置（保证可用准确度的必须项）

CFG-P0-01：数据访问与安全基线
- Steps:
  1) 为 SQLBot 创建只读数据库账号。
  2) 配置网络白名单 / VPN / 防火墙放行。
  3) 确认数据归属与审计规则。
- Dependencies：无
- Materials：DB 主机/端口/库名、账号凭证、安全策略

CFG-P0-02：新增 NC 数据源并同步核心元数据
- Steps:
  1) 在 SQLBot 中创建数据源（Oracle/SQLServer/Postgres）。
  2) 填写 IP/端口/数据库/账号信息并测试连接。
  3) 仅选择 DW/DIM 表，排除 ODS 与系统表。
  4) 在表预览中只启用必要字段，禁用噪声字段。
  5) 为该数据源开启“智能问数”。
- Dependencies：CFG-P0-01
- Materials：核心表清单（gl_、ap_、ar_、fa_、bd_、org_）

CFG-P0-03：核心财务表与字段描述
- Steps:
  1) 为 20+ 核心表写业务描述（gl_voucher、gl_detail、gl_balance、bd_accsubj、org_orgs、ar_recbill、ap_paybill、fa_card 等）。
  2) 为关键字段补充说明（状态、金额、日期、组织、科目）。
  3) 写清枚举取值（vouchstatus、billstatus、dr）与单位。
- Dependencies：CFG-P0-02
- Materials：数据字典、NC 文档、业务规则

CFG-P0-04：配置表关系（Join）
- Steps:
  1) 定义主外键关联（voucher<->detail、detail<->subject、voucher<->org、ar/ap 头表<->体表）。
  2) 按业务逻辑选择 Join 类型（inner/left）。
  3) 在“表关系管理”里拖拽建关系并保存。
  4) 用 3-5 条样例 SQL 校验。
- Dependencies：CFG-P0-03
- Materials：ERD / 表关系清单

CFG-P0-05：建设核心 NC 术语库
- Steps:
  1) 定义 30-50 个核心术语（科目类别、凭证状态、期间口径、AR/AP、资产类术语）。
  2) 补充同义词与计算公式，并关联表/字段。
  3) 在 设置->术语配置 中创建并启用，限定生效数据源。
- Dependencies：CFG-P0-03
- Materials：KPI/指标口径、术语表、示例报表

CFG-P0-06：建设高频问题 SQL 示例库
- Steps:
  1) 收集 50+ 高频财务问题。
  2) 提供标准 SQL 并包含业务过滤（dr=0、vouchstatus>=3、billstatus>=2）。
  3) 在 设置->SQL 示例库 中创建（问题 + 示例 SQL + 数据源）。
- Dependencies：CFG-P0-03、CFG-P0-04
- Materials：现有报表 SQL、高频问题清单、BI 口径

CFG-P0-07：NC 财务自定义提示词规则
- Steps:
  1) 加入默认过滤规则（dr=0、vouchstatus>=3）与期间口径。
  2) 加入数据库方言规则（Oracle/SQLServer/Postgres）。
  3) 加入命名/别名规则提升可读性。
- Note：官方文档标注“自定义提示词”为 X-Pack 功能；若不可用，走 DEV-P0-04。
- Dependencies：CFG-P0-03
- Materials：DB 方言规则、财务业务规则

CFG-P0-08：配置 LLM 与 Embedding 模型参数
- Steps:
  1) 在 系统管理->AI 模型配置 添加模型（供应商、模型名、基础模型、API 地址、API Key）。
  2) 设置参数（temperature、max_tokens）以提升稳定性。
  3) 设置系统默认模型并校验可调用。
  4) 配置 embedding 模型与相似度阈值（按本地文档/代码配置）。
- Dependencies：无
- Materials：模型地址、API Key、算力预算

CFG-P0-09：数据权限基线（工作空间 + 行/列权限）
- Steps:
  1) 创建工作空间并定义角色（管理员/普通用户）。
  2) 配置权限规则组（行/列权限）并绑定用户。
  3) 按组织/公司配置行级规则，敏感字段做列级脱敏。
- Dependencies：CFG-P0-01
- Materials：组织架构、角色矩阵、数据分级清单

CFG-P0-10：用户与工作空间设置（小白可用）
- Steps:
  1) 在 系统管理->用户管理 创建用户并设置状态/密码。
  2) 在 系统管理->工作空间 分配成员与角色。
  3) 校验隔离性：同一用户可属于不同工作空间且角色不同。
- Dependencies：CFG-P0-09
- Materials：用户清单、空间映射

### P1 配置（提升覆盖与可用性）

CFG-P1-01：扩展元数据覆盖与描述
- Steps：新增 30-50 张扩展表（辅助核算、项目、库存、现金/银行），并补充描述。
- Dependencies：CFG-P0-03
- Materials：扩展表清单

CFG-P1-02：按场景扩展术语与 SQL 示例
- Steps：新增场景包（预算、成本、现金流、税务、合并报表），示例总量 100+。
- Dependencies：CFG-P0-05、CFG-P0-06
- Materials：场景目录、报表模板

CFG-P1-03：建设新手问题模板库
- Steps：整理 50-100 个问题模板，附建议字段与过滤条件。
- Dependencies：CFG-P0-06
- Materials：训练问题、业务方输入

CFG-P1-04：建立评测集与周度复盘机制
- Steps：定义 100 题金标集；每周评测准确率并更新术语/示例。
- Dependencies：CFG-P0-06
- Materials：标注的 Q->SQL、验收标准

### P2 配置（可选增强）

CFG-P2-01：多组织对比模板与 KPI
- Steps：增加合并与多组织对比的提示词/示例。
- Dependencies：CFG-P1-02
- Materials：合并口径规则

CFG-P2-02：知识库导出/导入打包
- Steps：将术语/示例/提示词按版本打包导出。
- Dependencies：CFG-P1-02
- Materials：版本管理流程

CFG-P2-03：嵌入小助手到内部系统（可选）
- Steps：配置 嵌入式管理，生成嵌入代码，设置 CORS 与数据源范围。
- Dependencies：CFG-P0-09
- Materials：宿主系统 URL、对接负责人

## 二次开发工作（按优先级）

### P0 开发（配置无法确保准确度/可用性）

DEV-P0-01：SQL 护栏与规则重写
- Why config is insufficient：仅靠提示词无法稳定强制 dr=0/vouchstatus/billstatus。
- Steps:
  1) 增加 SQL AST 重写管道（LLM 输出后注入强制过滤）。
  2) 增加按数据源的规则集（NC 财务规则）。
  3) 增加样例 SQL 单测。
- Dependencies：CFG-P0-03、CFG-P0-07
- Materials：规则清单、安全 SQL 模式

DEV-P0-02：主题域/表分组减少噪声
- Why config is insufficient：NC 表过多，检索容易命中无关表。
- Steps:
  1) 增加主题域元数据（总账、AR/AP、固定资产、现金）。
  2) 调整检索与提示词，优先选择对应主题域表。
  3) 增加 UI 选择或自动分类器。
- Dependencies：CFG-P0-02、CFG-P0-03
- Materials：主题分类、表-主题映射

DEV-P0-03：一键 NC Starter Pack 导入
- Why config is insufficient：初始配置过重，对小白不友好。
- Steps:
  1) 内置 NC 术语 + SQL 示例 + 提示词包。
  2) 增加导入 UI 与版本管理。
  3) 增加导入后校验报告。
- Dependencies：CFG-P0-05、CFG-P0-06、CFG-P0-07
- Materials：NC Starter Pack 内容

DEV-P0-04：自定义提示词能力缺口（X-Pack 不可用时）
- Why config is insufficient：官方文档标注“自定义提示词”为 X-Pack 功能，开源版可能不可用。
- Steps:
  1) 验证当前版本是否支持该功能。
  2) 若缺失，补充开源版提示词规则存储与注入链路。
  3) 提供从 Starter Pack 迁移规则的机制。
- Dependencies：CFG-P0-07
- Materials：提示词规则集

### P1 开发（准确度闭环与体验）

DEV-P1-01：反馈采集与主动学习
- Steps:
  1) 增加用户评分与纠错 UI。
  2) 将纠错转化为新示例/术语建议。
  3) 增加周度复盘看板。
- Dependencies：CFG-P1-04
- Materials：反馈分类、复盘流程

DEV-P1-02：面向小白的引导式问数
- Steps:
  1) 向导式入口：主题 -> 指标 -> 时间 -> 组织。
  2) 转为结构化 Prompt + SQL。
  3) 常用问题沉淀为预设。
- Dependencies：DEV-P0-02
- Materials：交互流程、指标清单

DEV-P1-03：性能与安全
- Steps:
  1) 增加查询超时/成本限制。
  2) 为高频问题增加缓存。
  3) 对大结果集进行提示与截断。
- Dependencies：无
- Materials：基础设施约束

### P2 开发（可选）

DEV-P2-01：自动化评测与趋势
- Steps：生成合成 Q->SQL，夜间跑分并展示趋势图。
- Dependencies：DEV-P1-01
- Materials：评分标准

DEV-P2-02：跨数据源查询支持
- Steps：增加联邦查询或物化视图管道。
- Dependencies：CFG-P1-01
- Materials：集成方案

## 依赖关系（汇总）
- 元数据同步 -> 表/字段描述 -> 表关系 -> SQL 示例
- 术语与提示词依赖数据字典与业务规则
- SQL 护栏依赖提示词规则与标准过滤
- 主题域依赖表目录与业务模块划分
- 工作空间/权限依赖用户清单与组织架构

## 时间计划（下周一 T0 -> 2025-02-10）
- Phase A（T0 到 T0+2 周）：完成 CFG-P0-01..10，启动 DEV-P0-01/02 方案设计，收集资料
- Phase B（T0+2 周到 2025-02-10）：完成 DEV-P0-01/02/03/04，完成 CFG-P1-01/02/04，发布 Starter Pack
- Phase C（2025-02-11 之后）：推进 CFG-P2-01/02/03 与 DEV-P1/DEV-P2

## 待确认事项
- 确认数据库类型与可用层级（DW/DIM vs ODS）。
- 确认用户角色与数据权限规则。
