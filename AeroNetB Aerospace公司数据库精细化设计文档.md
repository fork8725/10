# 一、标题页

学生姓名：［请填写姓名］

学生ID:［请填写ID］

课程名称：［请填写课程名称］

提交日期：［请填写提交日期］

# 二、引言

## 2.1业务场景

AeroNetB Aerospace 在商用飞机关键部件流程中，当前数据库存在较多需要解决的部分。例如，部件生产过程中原材料批次与生产工序的绑定不够紧密，当某批次原材料出现质量问题时，需耗费大量时间追溯受影响的部件；此外，生产设备的维护数据与生产数据未联动，设备维护不及时导致的生产中断，无法提前预警；同时，客户订单的变更需求（如部件规格微调）缺乏高效的数据响应机制，变更后的数据更新易出现遗漏，影响订单交付质量。

## 2.2设计目标

本次设计紧扣“提升数据管理效率、保障数据安全、支撑业务决策”核心目标，细化落地路径：一是建立原材料批次与生产工序的精准绑定体系，实现原材料质量问题追溯时间缩短至1小时内；二是构建生产设备维护与生产数据的联动机制，提前7天预测设备维护需求，降低生产中断风险；三是设计订单变更数据响应流程，确保订单变更后2小时内完成全系统数据更新，避免数据遗漏。

## 2.3设计方法

采用“数据绑定分析-设备联动建模-变更响应设计-安全防护强化”的设计方法，提升数据精细化管理能力：

1. 数据绑定分析：梳理原材料批次、生产工序、部件成品间的关联关系，识别数据绑定关键节点；

2. 设备联动建模：在概念建模阶段新增设备维护、订单变更等实体，逻辑建模阶段设计设备数据与生产数据的联动规则；

3. 变更响应设计：建立订单变更数据的自动同步与校验机制，确保变更数据全链路覆盖；

4. 安全防护强化：针对原材料追溯、设备维护等核心数据，设计差异化安全策略，保障数据完整性与保密性。

# 三、概念设计

## 3.1新增核心实体及属性定义

1. 原材料批次记录（RawMaterialBatchRecord)：核心属性包括批次ID（唯一标识）、原材料ID（关联原材料）、供应商ID（关联供应商）、采购订单ID（关联采购订单）、批次数量、入库时间、质检结果（合格／不合格／待检测）、质检报告ID、存储位置、保质期截止时间、已使用数量、剩余数量、批次状态（在库／已用完／已冻结）；

2. 设备维护记录（EquipmentMaintenanceRecord)：核心属性包括维护ID（唯一标识）、设备ID（关联设备）、维护类型（日常维护／故障维修／定期保养）、维护开始时间、维护结束时间、维护人员ID列表、维护内容、更换零件清单、维护后检测结果、设备当前状态（正常／待维护／故障）、下次维护建议时间、维护成本、维护工单状态（待执行／执行中／已完成）；

3. 订单变更记录（OrderChangeRecord)：核心属性包括变更ID（唯一标识）、原订单ID（关联生产订单）、变更申请时间、变更申请人ID、变更类型（规格变更／数量变更／交付时间变更／其他）、变更内容描述、变更前数据、变更后数据、变更审批人ID、审批状态（待审批／已批准／已驳回）、审批时间、变更执行时间、变更影响范围（生产计划／库存／采购）、变更执行状态（未执行／执行中／已完成）。

## 3.2新增实体关系说明

1. 原材料批次记录与生产工序记录：一对多关系，一个原材料批次可用于多个生产工序，通过批次ID与工序ID关联，实现原材料批次追溯；

2. 设备与设备维护记录：一对多关系，一台设备可有多条维护记录，通过设备ID关联，形成设备全生命周期维护档案；

3. 生产订单与订单变更记录：一对多关系，一个生产订单可有多条变更记录，通过原订单ID关联，跟踪订单变更历史；

4. 订单变更记录与生产计划：多对一关系，多条订单变更记录可能影响同一生产计划，通过生产计划ID关联，确保生产计划及时调整。

## 3.3ER图

# 四、逻辑设计

## 4.1数据库选型补充说明

在原有多数据库架构基础上，引入Apache Flink 作为实时计算引擎，实现原材料批次数据与生产数据的实时关联分析，同时采用PostgreSQL数据库存储订单变更记录与审批流程数据，提升事务处理能力。各数据库分工调整：MS SQL Server 新增原材料批次记录表、设备维护记录表；PostgreSQL存储订单变更记录表、变更审批流程表；MongoDB新增设备维护附件文档集合、订单变更影响分析文档集合；InfluxDB新增设备运行状态与维护需求预测数据模型；Flink用于原材料批次与生产数据的实时关联计算。

## 4.2关系型数据库逻辑模型补充（MS SQL Server）

### 4.2.1原材料批次记录表（RawMaterialBatchRecord）

<table border="1" ><tr>
<td>字段名</td>
<td>数据类型</td>
<td>主键／外键</td>
<td>约束说明</td>
<td>备注</td>
</tr><tr>
<td>BatchID</td>
<td>VARCHAR( 50)</td>
<td>主键</td>
<td>NOT NULL,<br>UNIQUE</td>
<td>批次唯一标<br>识，格式如<br>"RB-<br>20250<br>01"</td>
</tr><tr>
<td>MateriallD</td>
<td>VARCHAR( 50)</td>
<td>外键（关联<br>RawMaterial.MateriallD<br>)</td>
<td>NOT NULL</td>
<td>关联原材料</td>
</tr><tr>
<td>SupplierID</td>
<td>VARCHAR( 50)</td>
<td>外键（关联<br>Supplier.SupplierID)</td>
<td>NOT NULL</td>
<td>关联供应商</td>
</tr><tr>
<td>PurchaseOrde rID</td>
<td>VARCHAR( 50)</td>
<td>外键（关联<br>PurchaseOrder.OrderID<br>)</td>
<td>NOT NULL</td>
<td>关联采购订单</td>
</tr><tr>
<td>BatchQuantity</td>
<td>DECIMAL(1 8,2)</td>
<td>-</td>
<td>NOT NULL,<br>CHECK(BatchQua $\text {ntity}>0$</td>
<td>批次数量</td>
</tr><tr>
<td>Storage Time</td>
<td>DATETIME</td>
<td>-</td>
<td>NOT NULL</td>
<td>入库时间</td>
</tr><tr>
<td>QualityCheckR esult</td>
<td>VARCHAR( 20)</td>
<td>-</td>
<td>CHECK<br>(QualityCheckResu ItIN(＇合格＇，不合<br>格＇,＇待检测＇)）</td>
<td>质检结果</td>
</tr><tr>
<td>InspectionRep ortID</td>
<td>VARCHAR( 50)</td>
<td>外键（关联<br>QualityInspection.Inspect ionID)</td>
<td>NULL</td>
<td>质检报告ID</td>
</tr><tr>
<td>StorageLocatio n</td>
<td>VARCHAR( 100)</td>
<td>-</td>
<td>NOT NULL</td>
<td>存储位置</td>
</tr><tr>
<td>ExpirationDate</td>
<td>DATE</td>
<td>-</td>
<td>NOT NULL</td>
<td>保质期截止时间</td>
</tr><tr>
<td>UsedQuantity</td>
<td>DECIMAL(1 8,2)</td>
<td>-</td>
<td>NOT NULL,<br>$CHECK(UsedQuan$ $\text {tity}>=0\text {)}$</td>
<td>已使用数量</td>
</tr><tr>
<td>RemainingQua ntity</td>
<td>DECIMAL(1 8,2)</td>
<td>-</td>
<td>GENERATED<br>ALWAYS AS<br>(BatchQuantity-<br>UsedQuantity)<br>STORED</td>
<td>剩余数量（自动计<br>算）</td>
</tr><tr>
<td>BatchStatus</td>
<td>VARCHAR( 20)</td>
<td>-</td>
<td>CHECK<br>(BatchStatus IN ('<br>在库＇,＇已用完＇,＇已冻结＇)）</td>
<td>批次状态</td>
</tr></table>

### 4.2.2新增关联表

1. 原材料批次与工序关联表（MaterialBatchProcessRelation)：关联原材料批次与生产工序，字段包括关联ID（主键）、批次ID（外键）、工序ID（外键）、使用数量、使用时间、操作人员ID、使用记录状态（正常／异常）；

2. 设备维护与生产影响关联表（EquipmentMaintenanceProductionRelation)：关联设备维护与生产计划，字段包括关联ID（主键）、维护ID（外键）、生产计划ID（外键）、影响生产时长（小时）、影响工序、应对措施、恢复生产时间。

## 4.3文档型数据库逻辑模型补充（MongoDB）

### 4.3.1设备维护附件文档集合

## (EquipmentMaintenanceAttachment)

**{**

＂id":"MA-2025001",/／附件ID

＂MaintenanceID":"EM-2025001",/／关联设备维护ID

＂AttachmentType":＂维护报告”，/／附件类型（维护报告／检测报告／零件清单）

＂AttachmentName":＂设备X-2025定期保养报告＂,/／附件名称

＂FileFormat":"PDF",/／文件格式

＂FileSize":2048,/／文件大小（KB）

＂UploadTime": "2025-11-26 16:00:00",/／上传时间

＂UploadPersonID": "EMP-2025020",/／上传人员ID

＂FileContent":"Base64编码的文件内容＂,/／文件内容（Base64编码）

＂RelatedEquipmentParts":[/／关联设备零件子文档

{

"PartID": "EP-2025003",

＂PartName":＂主轴轴承”，

＂ReplacementReason":＂磨损超出允许范围＂，

"ReplacementQuantity": 2

},

{し

"PartID": "EP-2025005",

＂PartName":＂密封件＂，

＂ReplacementReason":＂老化＂，

"ReplacementQuantity": 4

}

],

＂Version":"V1.0",/／文档版本

＂UpdateRecord":[/／更新记录子文档

{

"UpdateTime": "2025-11-26 16:30:00",

＂UpdateContent":＂补充维护后检测数据”，

"UpdatePersonID": "EMP-2025021"

}

]

}

## 4.4时序数据库逻辑模型补充（InfluxDB）

**4.4.1设备维护需求预测数据表**

**(EquipmentMaintenancePrediction)**

<table border="1" ><tr>
<td>组成部分</td>
<td>字段名称</td>
<td>数据类型</td>
<td>说明</td>
</tr><tr>
<td>Measurement</td>
<td>EquipmentMaintenancePrediction</td>
<td>-</td>
<td>测量值名称，代表设备维护需求预测数据集合</td>
</tr><tr>
<td>Tag （索引字段）</td>
<td>EquipmentID</td>
<td>字符串</td>
<td>设备唯一标识，<br>用于按设备过滤查询</td>
</tr><tr>
<td>Tag （索引字段）</td>
<td>EquipmentType</td>
<td>字符串</td>
<td>设备类型（数控机床／组装机器人等）</td>
</tr><tr>
<td>Tag （索引字段）</td>
<td>WorkshopID</td>
<td>字符串</td>
<td>所属车间ID ，<br>用于按车间聚合查询</td>
</tr><tr>
<td>Field（值字段）</td>
<td>CurrentHealthScore</td>
<td>浮点数</td>
<td>设备当前健康得分（0-100分）</td>
</tr><tr>
<td>Field（值字段）</td>
<td>PredictedMaintenance Time</td>
<td>字符串</td>
<td>预测下次维护时间（格式：<br>YYYY-MM-<br>DD)</td>
</tr><tr>
<td>Field（值字段）</td>
<td>MaintenanceUrgency</td>
<td>字符串</td>
<td>维护紧急程度<br>（低／中／高）</td>
</tr><tr>
<td>Field（值字段）</td>
<td>PredictedFaultProbability</td>
<td>浮点数</td>
<td>预测故障概率<br>(0-100%)</td>
</tr><tr>
<td>Field（值字段）</td>
<td>SuggestedMaintenanceContent</td>
<td>字符串</td>
<td>建议维护内容</td>
</tr><tr>
<td>Timestamp</td>
<td>Time</td>
<td>时间戳</td>
<td>预测数据生成时间，精确到小时</td>
</tr></table>

## 4.5关系型数据库逻辑模型补充（PostgreSQL）

**4.5.1订单变更记录表（OrderChangeRecord）**

<table border="1" ><tr>
<td>字段名</td>
<td>数据类型</td>
<td>主键／外键</td>
<td>约束说明</td>
<td>备注</td>
</tr><tr>
<td>ChangelD</td>
<td>VARCHAR(5 0)</td>
<td>主键</td>
<td>NOT NULL,<br>UNIQUE</td>
<td>变更唯<br>一标<br>识，格<br>式如<br>"OC-<br>202500<br>1"</td>
</tr><tr>
<td>OriginalOrderID</td>
<td>VARCHAR(5 0)</td>
<td>外键（关联<br>ProductionOrder.Orderl<br>D)</td>
<td>NOT NULL</td>
<td>关联原<br>生产订<br>单</td>
</tr><tr>
<td>ChangeApplyTim e</td>
<td>DATETIME</td>
<td>-</td>
<td>NOT NULL</td>
<td>变更申<br>请时间</td>
</tr><tr>
<td>ApplicantID</td>
<td>VARCHAR(5 0)</td>
<td>外键（关联<br>Employee.EmployeelD<br>)</td>
<td>NOT NULL</td>
<td>变更申<br>请人ID</td>
</tr><tr>
<td>ChangeType</td>
<td>VARCHAR(3 0)</td>
<td>-</td>
<td>CHECK<br>(ChangeType<br>IN (＇规格变更<br>＇，数量变更＇，<br>交付时间变更<br>，其他＇)）</td>
<td>变更类<br>型</td>
</tr><tr>
<td>ChangeDescripti on</td>
<td>TEXT</td>
<td>-</td>
<td>NOT NULL</td>
<td>变更内<br>容描述</td>
</tr><tr>
<td>DataBeforeChan ge</td>
<td>JSONB</td>
<td>-</td>
<td>NOT NULL</td>
<td>变更前<br>数据<br>(JSON 格式）</td>
</tr><tr>
<td>DataAfterChange</td>
<td>JSONB</td>
<td>-</td>
<td>NOT NULL</td>
<td>变更后<br>数据<br>(JSON 格式）</td>
</tr><tr>
<td>ApproverID</td>
<td>VARCHAR(5 0)</td>
<td>外键（关联<br>Employee.EmployeelD<br>)</td>
<td>NULL</td>
<td>变更审<br>批人ID</td>
</tr><tr>
<td>ApprovalStatus</td>
<td>VARCHAR(2 0)</td>
<td>-</td>
<td>CHECK<br>(ApprovalStatu<br>s IN(＇待审批＇,＇已批准＇,＇已驳<br>回＇)）</td>
<td>审批状<br>态</td>
</tr><tr>
<td>ApprovalTime</td>
<td>DATETIME</td>
<td>-</td>
<td>NULL</td>
<td>审批时<br>间</td>
</tr><tr>
<td>ExecutionTime</td>
<td>DATETIME</td>
<td>-</td>
<td>NULL</td>
<td>变更执<br>行时间</td>
</tr><tr>
<td>ImpactScope</td>
<td>VARCHAR(5 0)</td>
<td>-</td>
<td>CHECK<br>(ImpactScope<br>IN (＇生产计划<br>＇，库存＇，采购<br>，多范围＇)）</td>
<td>变更影<br>响范围</td>
</tr><tr>
<td>ExecutionStatus</td>
<td>VARCHAR(2 0)</td>
<td>-</td>
<td>CHECK<br>(ExecutionStat us IN(＇未执行，执行中＇，已<br>完成＇)）</td>
<td>变更执<br>行状态</td>
</tr></table>

# 五、安全设计与访问控制优化

## 5.1基于角色的访问控制（RBAC）优化

新增两大角色，完善原材料与设备管理权限体系：

**5.1.1新增角色及权限范围**

<table border="1" ><tr>
<td>角色ID</td>
<td>角色名称</td>
<td>对应岗位</td>
<td>数据访问权<br>限</td>
<td>操作执行权<br>限</td>
<td>审批权限</td>
</tr><tr>
<td>R-012</td>
<td>原材料管理员</td>
<td>供应链管理部门</td>
<td>原材料批次<br>数据、采购<br>订单数据、<br>库存数据</td>
<td>原材料入库<br>登记、批次<br>状态更新、<br>库存盘点</td>
<td>原材料批次<br>冻结审批、<br>不合格原材<br>料处理审批</td>
</tr><tr>
<td>R-013</td>
<td>设备维护主管</td>
<td>设备管理部门</td>
<td>设备维护数<br>据、设备运<br>行数据、生<br>产影响数据</td>
<td>维护计划制<br>定、维护工<br>单分配、维<br>护结果审核</td>
<td>设备故障维<br>修审批、重<br>大维护成本<br>审批</td>
</tr></table>

### 5.1.2权限实现机制优化

引入“数据操作日志分级”功能，原材料批次冻结、设备重大维护等关键操作，日志需包含操作前数据快照，便于后续追溯；针对订单变更数据，采用“多级审批”机制，涉及生产计划调整的变更需经生产主管与计划专员双重审批，确保变更合理性。

## 5.2数据安全保障措施强化

1. 原材料批次数据加密：对PostgreSQL中存储的原材料批次质检报告ID、存储位置等核心数据，采用Transparent Data Encryption(TDE）加密技术，防止数据文件被窃取；

2. 设备维护数据访问控制：设备维护记录中的维护成本、更换零件清单等敏感数据，仅设备维护主管可查看完整信息，维护人员仅能查看与自身负责维护相关的部分数据；

3. 订单变更数据校验：建立订单变更数据的完整性校验规则，变更后数据需与生产、库存系统数据匹配，校验不通过则触发预警，防止无效变更；

4. 数据备份与恢复：针对订单变更记录、设备维护记录等关键数据，采用“实时增量备份＋每日全量备份”策略，备份数据存储于异地灾备中心，RTO（恢复时间目标）≤2小时。

# 六、数据集成与实时处理优化

## 6.1多类型数据集成方案补充

1. 数据接入层新增：原材料批次数据通过采购系统API接入MS SQL Server；设备维护数据通过设备管理系统录入，经数据校验后同步至MS SQL Server 与MongoDB；订单变更数据通过订单管理系统提交，实时写入PostgreSQL；

2. 数据转换层优化：新增原材料批次与生产工序的关联校验规则，确保原材料使用数量不超过批次剩余数量；对设备维护数据中的维护时间、成本等字段进行格式标准化处理；

3. 数据服务层新增：针对原材料管理提供“原材料批次追溯API”“库存预警API”；针对设备管理提供“设备维护计划API”“设备健康状态查询API”；针对订单管理提供“订单变更查询API”“变更影响分析API”。

## 6.2物联网实时数据处理设计拓展

在InfluxDB与Flink协同架构中，新增“设备维护需求实时预测”功能：Flink实时读取InfluxDB中的设备运行数据（温度、振动、转速等），结合设备维护历史数据，通过LSTM 神经网络模型计算设备健康得分与故障概率；当设备健康得分低于60分或故障概率高于30％时，自动生成维护需求预警，推送至设备维护主管移动端，并同步更新InfluxDB中的设备维护需求预测数据，提前安排维护计划，减少生产中断。

