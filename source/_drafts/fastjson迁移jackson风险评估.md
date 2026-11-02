---
title: fastjson迁移jackson风险评估
date: 2026-10-30 10:31:16
tags: [AI生成, Java, fastjson, Jackson, 技术评审]
published: false
permalink: /articles/2026/10/30/1793605876384.html
---

> 📝 本文由 AI agent 起草,尚未经过人工审核。发布前请由 Kael 本人审阅修改。
> 撰写 agent:Hermes · 起草日期:2026-10-30

# fastjson 迁移 Jackson 风险评估

## 1. 背景与范围

本项目对一个大 JSON 文档做「反序列化 → 持久化到数据库 → 读取时再序列化」的处理。评估将 JSON 库从 fastjson(含 fastjson2)迁移到 Jackson 的风险。

**场景特点决定了本次迁移的核心矛盾:这不是普通的"运行时内存对象"场景,而是一条 JSON → 对象 → DB → 对象 → JSON 的持久化链路。** 风险不只在于"新库能不能正确解析",更在于:

1. 同一份存量数据,经 Jackson 反序列化再序列化后,能否与 fastjson 时代的产物**字节级/语义级等价**;
2. 反序列化行为差异会不会导致**写库字段错乱**(字段丢失、类型改变、null 语义漂移),一旦写入就是脏数据,回滚困难。

## 2. 风险清单(按严重程度排序)

### R1. 反序列化行为差异 → 脏数据入库【严重】

这是本场景最致命的一类风险,因为错误会随写库固化为存量数据。

| 差异点 | fastjson | Jackson | 对本项目的后果 |
|---|---|---|---|
| 未知字段 | 默认忽略 | 默认抛 `UnrecognizedPropertyException` | DB 里 JSON 结构若有历史演进(新增字段),迁移后直接解析失败 |
| 缺失字段 | 填默认值(0/false/null 依类型) | 引用类型填 null,基本类型抛异常(若用基本类型) | 实体类用 `int`/`boolean` 时,老数据缺字段会炸 |
| null 与缺省 | `toJSONString` 默认**不输出 null 字段** | 默认**输出 null** | 见 R2,序列化产物变化 |
| 泛型/嵌套类型推断 | 部分场景自行推断 | 严格按声明类型 | 嵌套泛型(`List<Map<String, Foo>>`)fastjson 宽松、Jackson 严格,复杂大 JSON 容易踩 |
| 数字类型 | int→long 溢出、`BigDecimal` 默认行为不同 | 严格按目标类型,溢出抛异常 | 金额/ID 字段需逐个核对目标类型 |
| 日期格式 | 默认 `yyyy-MM-dd HH:mm:ss`(fastjson1) | 默认 ISO-8601(`yyyy-MM-dd'T'HH:mm:ss`) | **时间字段入库格式漂移**,除非显式统一 |
| 枚举 | 按 name 或 ordinal(版本间有差异) | 默认按 name,`@JsonValue` 可定制 | 存量 ordinal 序列化的枚举会解析错位 |

**缓解:**
- Jackson 侧全局配置 `FAIL_ON_UNKNOWN_PROPERTIES=false` 对齐 fastjson 的宽容行为;
- 实体字段全部用包装类型(`Integer`/`Long`/`Boolean`),禁用基本类型;
- 日期统一显式 `@JsonFormat` 或全局 `ObjectMapper.setDateFormat`,迁移前扫描所有日期字段;
- 枚举序列化方式写单测锁定。

### R2. 序列化产物差异 → 下游兼容【高】

本项目「读取时序列化」,序列化结果会被下游(接口消费方、文件、缓存)看到:

| 差异 | fastjson | Jackson(默认) |
|---|---|---|
| null 字段 | 省略 | 输出 `"field":null` |
| 字段顺序 | 按字段名字母序(fastjson1 默认 `Map` 序) | 按类定义声明序 |
| 空集合/空串 | `SerializerFeature.WriteMapNullValue` 等开关控制 | `@JsonInclude(NON_NULL)` 等注解控制 |
| `transient` | 默认**会**序列化(fastjson1 历史行为) | 默认不序列化 | 
| 自转义/引号风格、unicode 转义 | 有差异 | 有差异 |

如果下游做**签名/摘要校验**或对字段集有硬编码假设,null 字段的出现/消失、字段顺序变化都会造成破坏。

**缓解:**
- 对齐策略二选一:Jackson 加 `@JsonInclude(NON_NULL)` 模拟 fastjson 省略 null;或借迁移契机与下游确认 null 语义,显式化契约(推荐后者,但需要下游配合,属计划内变更);
- 若有依赖字段顺序的逻辑,迁移前必须改造为顺序无关。

### R3. 注解体系替换 → 改造面与遗漏【高】

fastjson 注解(`@JSONField`)与 Jackson 注解(`@JsonProperty` 等)不兼容,必须逐字段替换:

| fastjson | Jackson |
|---|---|
| `@JSONField(name="x")` | `@JsonProperty("x")` |
| `@JSONField(format=...)` | `@JsonFormat(pattern=...)` |
| `@JSONField(serialize/deserialize=false)` | `@JsonIgnore` / `@JsonProperty(access=...)` |
| `@JSONField(ordinal=...)` | `@JsonPropertyOrder` |
| `@JSONCreator` / `@JSONType` | `@JsonCreator` / `@JsonTypeInfo` |

大 JSON 对应的实体类通常字段多、层级深,替换是纯机械工作但**遗漏不易发现**——`@JSONField(name=...)` 漏改成 `@JsonProperty` 时,Jackson 默认用字段名,不报错,但 JSON key 直接变了,写库即错位。

**缓解:**
- 不靠肉眼:写脚本扫描源码中全部 `@JSONField` 出现点,生成替换清单,迁完后清单必须清零;
- 用「双跑比对」兜底(见 §4)。

### R4. API 迁移与工具类封装【中】

直接调用点(`JSON.parseObject` / `toJSONString` / `JSONArray` / `JSONObject`)需要替换为 `ObjectMapper`。要点:

- `ObjectMapper` 线程安全且构造昂贵,**必须全局单例**(或 Spring 注入),不能像 fastjson 静态方法那样随手调;
- `JSONObject`/`JSONArray` 这类弱类型容器在 fastjson 代码里经常被当作动态 Map 用,迁移时要么改 `ObjectNode`/`ArrayNode`,要么改 `Map`/`List`,涉及大量取值方式改写(`getString`/`getInteger` → 手动类型转换);
- fastjson 的 `Feature`/`SerializerFeature` 开关与 Jackson 的 `Feature` 不一一对应,每个开关都要单独决策对齐方式;
- TypeReference 用法不同:`new TypeReference<List<Foo>>(){}` 两边都有但 import 不同,IDE 批量替换容易漏。

**缓解:** 先收敛出项目内统一的 `JsonUtil` 门面(facade),业务代码只依赖门面;迁移即重写门面实现,业务侧零改动。这是降低改造面的关键一步,**建议先做,再换内核**。

### R5. 注入类漏洞面变化【中/低,取决于版本】

- 迁移动机通常包含 fastjson1 的反序列化 RCE 历史(autoType)。注意:**Jackson 的默认配置同样有Polymorphic Deserialization 风险**——开启 `DefaultTyping`(或 `@JsonTypeInfo(use=CLASS)`)且输入不可信时,等价于打开 autoType,历史上也出过 CVE(如 CVE-2017-7525、CVE-2019-12384 系列利用链);
- 本项目 JSON 来源若是外部输入(接口/文件),迁移后**禁止**开启全局 `enableDefaultTyping`;多态需求用 `@JsonTypeInfo(use=NAME)` + 白红 `@JsonSubTypes`;
- fastjson2 + safeMode 已大幅收敛 autoType 风险;若当前是 fastjson1 ≥1.2.83 或 fastjson2,纯安全动机的收益要重新评估,不应作为唯一理由。

### R6. 性能特征变化【中,需基准验证】

- fastjson 与 fastjson2 在部分反射/极端场景吞吐高;Jackson 在对象树不大、注解元数据充分预热后性能相当,大文档流式解析(`JsonParser`/`JsonGenerator`)反而更稳;
- 本项目是「大 JSON 单文档」,需关注:**一次反序列化的峰值内存**(两者中间表示不同)、深度嵌套的栈使用、以及 DB 读写侧的吞吐变化;
- 结论不能靠传闻:**用生产样本数据做 JMH/简单计时基准,对比迁移前后 p50/p95 与峰值堆内存**。

### R7. 存量数据的双格式共存【高,易被忽略】

- DB 里已存的是 fastjson 序列化产物。若其中包含 fastjson 特有产物(如 `$ref` 引用、`@type` 类型标记、特定日期格式),Jackson 读取时要么解析失败要么语义错乱;
- `@type` 若存在,Jackson 默认直接当普通字段处理或报未知字段——**必须先抽样扫描存量数据,确认是否存在 fastjson 专属结构**;
- 迁移窗口期内「旧数据由 Jackson 写、新数据由 Jackson 读」与「旧存量由 fastjson 写」并存,读取路径要对两种格式都兼容。

**缓解:** 迁移前对 DB 存量 JSON 抽样(按时间分桶):统计 `$ref`/`@type`/日期格式/字段集,产出《存量数据特征报告》,这是 go/no-go 的前置输入。

### R8. 其他工程性风险【低】

- 依赖树:Jackson 版本需与 Spring Boot `jackson-bom` 对齐,避免多版本共存(`NoSuchMethodError` 类问题排查成本高);
- 事务与超时:解析大 JSON 的耗时变化会影响 DB 连接占用时间与上游超时配置;
- 日志/监控:若现有代码 catch `JSONException`,需同步改为 `JsonProcessingException`,否则异常逃逸改变错误处理路径。

## 3. 风险矩阵

| 编号 | 风险 | 可能性 | 影响 | 等级 | 缓解手段 |
|---|---|---|---|---|---|
| R1 | 反序列化差异→脏数据入库 | 高 | 高 | **P0** | 配置对齐+包装类型+日期统一+单测锁定 |
| R2 | 序列化产物差异→下游破坏 | 高 | 中高 | **P0** | NON_NULL 对齐或下游契约确认 |
| R3 | 注解遗漏→字段错位 | 高 | 中 | **P1** | 脚本扫描清单清零+双跑比对 |
| R7 | 存量数据含 fastjson 专属结构 | 中 | 高 | **P0** | 迁移前抽样扫描存量数据 |
| R4 | API 改造面大 | 高 | 中 | **P1** | JsonUtil 门面先行 |
| R5 | 反序列化漏洞面变化 | 低 | 高 | **P2** | 禁 DefaultTyping+白名单 |
| R6 | 性能回归 | 中 | 中 | **P1** | 生产样本基准 |
| R8 | 依赖冲突/异常类型变更 | 中 | 低 | **P2** | 对齐 jackson-bom+全局搜索异常类型 |

## 4. 推荐迁移方案(降低风险的路径)

1. **第一步不是换库,是收口。** 建立统一 `JsonUtil` 门面,把散落的 `JSON.parseObject`/`toJSONString` 全部收敛进来。此步不改行为,可独立上线验证。
2. **存量数据画像。** 按 §2-R7 抽样扫描 DB 存量 JSON,确认无 fastjson 专属结构、字段集与日期格式分布。发现 `$ref`/`@type` 即需数据清洗预案。
3. **双跑比对(核心保障)。** 门面内部同时用 fastjson 和 Jackson 各执行一遍:
   - 读路径:同一份存量 JSON,两个库各反序列化一次,对比对象全字段(可用递归 equals 或再序列化后规范化对比);
   - 写路径:同一对象两个库各序列化,解析回 Map 后做**语义等价对比**(忽略字段顺序,显式对比 null 语义);
   - 灰度期双跑,Jackson 结果仅记录差异不上线,差异率收敛到 0 后切换。
4. **注解替换 + 清单清零。** 脚本扫描 `@JSONField` 生成替换清单,逐项替换,CI 里加检查确保清单归零。
5. **基准测试。** 用生产尺寸的样本 JSON 做迁移前后基准(p50/p95、峰值堆内存、GC),确认 DB 写入吞吐无显著回归。
6. **灰度与回滚。** 门面层加开关(配置中心)支持运行时切回 fastjson;回滚成本 = 改配置,前提是第 3 步期间没有用 Jackson 写出与旧格式不兼容的新数据——因此写路径的灰度要晚于读路径。

## 5. 结论

- 迁移**可行但不是"换依赖"级别的工作量**,核心成本在持久化链路的等价性验证,而非 API 替换本身;
- 三件事决定成败:**读路径对存量数据的兼容验证(R1/R7)、写路径对下游的产物等价(R2)、注解替换零遗漏(R3)**;
- 建议按 §4 的顺序推进,双跑比对是唯一能提供足够置信度的验证手段;
- 若当前 fastjson 已是 fastjson2 且无安全合规硬约束,迁移收益(生态、Spring 默认集成、可维护性)需与上述成本权衡,可排期但不必紧急。
