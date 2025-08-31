# 对话流程DSL规范文档

## 概述

这是一个用于定义智能客服对话流程的领域特定语言(DSL)。该DSL采用JSON格式，通过节点(nodes)和边(edges)的方式描述对话流程的逻辑结构，支持多种类型的节点来处理不同的对话场景。

## 整体结构

```json
{
    "nodes": [
        // 节点数组，定义流程中的各个处理单元
    ],
    "edges": [
        // 边数组，定义节点之间的连接关系
    ]
}
```

## 节点(Nodes)规范

### 节点基础结构

每个节点都包含以下基础字段：

```json
{
    "node_id": "唯一标识符",
    "type": "节点类型",
    "name": "节点名称(可选)",
    "node_config": {
        // 节点特定配置
    }
}
```

### 节点类型详解

#### 1. START - 开始节点
流程的入口点。

```json
{
    "node_id": "uuid",
    "type": "START",
    "name": "开始",
    "node_config": {}
}
```

#### 2. END - 结束节点
流程的终点。

```json
{
    "node_id": "uuid",
    "type": "END",
    "name": "结束流程",
    "node_config": {}
}
```

#### 3. USER_INPUT - 用户输入节点
收集用户输入的消息。

```json
{
    "node_id": "uuid",
    "type": "USER_INPUT",
    "name": "消息采集",
    "node_config": {
        "output_variable": "变量名"
    }
}
```

**配置说明：**
- `output_variable`: 存储用户输入的变量名

#### 4. LLM_PROCESS - 大模型处理节点
使用大语言模型处理用户输入或执行特定任务。

```json
{
    "node_id": "uuid",
    "type": "LLM_PROCESS",
    "name": "大模型",
    "node_config": {
        "prompt": "提示词内容",
        "enable_memory": true/false,
        "output_variables": [
            "输出变量1",
            "输出变量2"
        ]
    }
}
```

**配置说明：**
- `prompt`: 发送给大模型的提示词
- `enable_memory`: 是否启用对话记忆
- `output_variables`: 输出变量列表

#### 5. CONDITION - 条件判断节点
根据条件进行流程分支。

```json
{
    "node_id": "uuid",
    "type": "CONDITION",
    "name": "条件",
    "node_config": {
        "condition_groups": [
            {
                "condition_group_id": "uuid",
                "logic": "and/or",
                "condition_group": [
                    {
                        "variable": "变量路径",
                        "operator": "操作符",
                        "value": "比较值"
                    }
                ]
            }
        ]
    }
}
```

**配置说明：**
- `condition_groups`: 条件组数组
- `logic`: 条件组内的逻辑关系(and/or)
- `variable`: 要判断的变量路径
- `operator`: 比较操作符(==, !=, in, not in, >=, <等)
- `value`: 比较的目标值

#### 6. INTENT - 意图识别节点
识别用户的意图。

```json
{
    "node_id": "uuid",
    "type": "INTENT",
    "name": "意图识别",
    "node_config": {
        "input_variable": "输入变量",
        "condition_groups": [
            // 条件组配置，同CONDITION节点
        ]
    }
}
```

**配置说明：**
- `input_variable`: 用于意图识别的输入变量
- `condition_groups`: 意图判断条件

#### 7. RAG_RESPOND - 知识库回复节点
基于检索增强生成的回复节点。

```json
{
    "node_id": "uuid",
    "type": "RAG_RESPOND",
    "name": "大模型",
    "node_config": {
        "prompt": "提示词",
        "enable_memory": true/false,
        "retrieval_query_variable": "检索查询变量",
        "rewrite_query": true/false,
        "is_global_knowledge_base": true/false
    }
}
```

**配置说明：**
- `retrieval_query_variable`: 用于检索的查询变量
- `rewrite_query`: 是否重写查询
- `is_global_knowledge_base`: 是否使用全局知识库

#### 8. CARD_REPLY - 卡片回复节点
发送卡片形式的回复。

```json
{
    "node_id": "uuid",
    "type": "CARD_REPLY",
    "node_config": {
        "card_content": "卡片内容",
        "card_biz_info": "业务信息",
        "option": [
            {
                "option_name": "选项标识",
                "option_id": "选项ID"
            }
        ]
    }
}
```

**配置说明：**
- `card_content`: 卡片显示内容
- `card_biz_info`: 卡片业务信息
- `option`: 卡片选项数组(可选)

#### 9. SELECTOR_REPLY - 选择器回复节点
提供选择器供用户选择。

```json
{
    "node_id": "uuid",
    "type": "SELECTOR_REPLY",
    "node_config": {
        "selector_name": "选择器名称",
        "option": [
            {
                "option_name": "选项标识",
                "option_id": "选项显示文本"
            }
        ]
    }
}
```

**配置说明：**
- `selector_name`: 选择器名称
- `option`: 选项数组

#### 10. API_INVOKE - API调用节点
调用外部API接口。

```json
{
    "node_id": "uuid",
    "type": "API_INVOKE",
    "node_config": {
        "api_type": "read/write",
        "api_output_variable": [
            "输出变量1",
            "输出变量2"
        ]
    }
}
```

**配置说明：**
- `api_type`: API类型(read/write)
- `api_output_variable`: API返回值存储的变量

#### 11. JUMP_SUB_FLOW - 子流程跳转节点
跳转到其他子流程。

```json
{
    "node_id": "uuid",
    "type": "JUMP_SUB_FLOW",
    "name": "跳转流程",
    "node_config": {
        "sub_flow_group_id": "子流程ID"
    }
}
```

**配置说明：**
- `sub_flow_group_id`: 目标子流程的ID

#### 12. HUMAN - 转人工节点
将对话转接给人工客服。

```json
{
    "node_id": "uuid",
    "type": "HUMAN",
    "name": "转人工",
    "node_config": {}
}
```

## 边(Edges)规范

边定义了节点之间的连接关系和流转条件。

### 边的基础结构

```json
{
    "edge_id": "边的唯一标识",
    "source_node_id": "源节点ID",
    "target_node_id": "目标节点ID",
    "source_condition_group_id": "源条件组ID"
}
```

### 边的类型

#### 1. 普通连接边
直接连接两个节点。

```json
{
    "edge_id": "uuid",
    "source_node_id": "源节点ID",
    "target_node_id": "目标节点ID",
    "source_condition_group_id": "节点默认输出"
}
```

#### 2. 条件分支边
基于条件判断的分支连接。

```json
{
    "edge_id": "uuid",
    "source_node_id": "条件节点ID",
    "target_node_id": "目标节点ID",
    "source_condition_group_id": "条件组ID"
}
```

#### 3. 虚拟边
系统自动生成的连接边。

```json
{
    "edge_id": "virtual_源节点ID_目标节点ID",
    "source_node_id": "源节点ID",
    "target_node_id": "目标节点ID",
    "source_condition_group_id": ""
}
```

## 变量系统

### 变量命名规范

1. **系统变量**: `ola_flow_systemVariable_变量名`
2. **自定义变量**: `ola_flow_customVariable_变量名`
3. **数据变量**: `$.data.服务名.字段名`

### 常用变量类型

- `ola_flow_systemVariable_last_user_response`: 用户最后一次输入
- `ola_flow_systemVariable_history_rounds_count`: 对话轮次
- `ola_flow_systemVariable_eva_first_tag`: 一级标签
- `ola_flow_systemVariable_eva_second_tag`: 二级标签
- `ola_flow_systemVariable_eva_third_tag`: 三级标签

## 操作符规范

### 比较操作符
- `==`: 等于
- `!=`: 不等于
- `>`: 大于
- `>=`: 大于等于
- `<`: 小于
- `<=`: 小于等于
- `in`: 包含在数组中
- `not in`: 不包含在数组中

### 逻辑操作符
- `and`: 逻辑与
- `or`: 逻辑或
- `AND`: 条件组间的逻辑与

## 最佳实践

### 1. 节点设计原则
- 每个节点应该有明确的单一职责
- 使用有意义的节点名称
- 合理设置条件判断逻辑

### 2. 流程设计原则
- 确保有明确的开始和结束节点
- 避免创建无法到达的节点
- 合理处理异常情况和边界条件

### 3. 变量管理
- 使用规范的变量命名
- 及时清理不再使用的变量
- 注意变量作用域

### 4. 条件设计
- 条件判断应该覆盖所有可能的情况
- 使用合适的默认分支
- 避免过于复杂的嵌套条件

## 示例流程

以下是一个简单的客服对话流程示例：

```json
{
    "nodes": [
        {
            "node_id": "start-001",
            "type": "START",
            "name": "开始",
            "node_config": {}
        },
        {
            "node_id": "input-001",
            "type": "USER_INPUT",
            "name": "用户输入",
            "node_config": {
                "output_variable": "user_message"
            }
        },
        {
            "node_id": "intent-001",
            "type": "INTENT",
            "name": "意图识别",
            "node_config": {
                "input_variable": "user_message",
                "condition_groups": [
                    {
                        "condition_group_id": "intent-group-1",
                        "logic": "or",
                        "condition_group": [
                            {
                                "variable": "complaint_intent",
                                "operator": "==",
                                "value": "是"
                            }
                        ]
                    }
                ]
            }
        },
        {
            "node_id": "response-001",
            "type": "RAG_RESPOND",
            "name": "智能回复",
            "node_config": {
                "prompt": "请根据用户问题提供专业回复",
                "enable_memory": true,
                "retrieval_query_variable": "user_message"
            }
        }
    ],
    "edges": [
        {
            "edge_id": "edge-001",
            "source_node_id": "start-001",
            "target_node_id": "input-001",
            "source_condition_group_id": "trigger-source-handle-start-001"
        },
        {
            "edge_id": "edge-002",
            "source_node_id": "input-001",
            "target_node_id": "intent-001",
            "source_condition_group_id": "input-001_message_collection"
        },
        {
            "edge_id": "edge-003",
            "source_node_id": "intent-001",
            "target_node_id": "response-001",
            "source_condition_group_id": "intent-group-1"
        }
    ]
}
```

## 扩展性

该DSL设计具有良好的扩展性：

1. **新节点类型**: 可以通过添加新的`type`值来支持新的节点类型
2. **新操作符**: 可以在条件判断中添加新的操作符
3. **新变量类型**: 可以定义新的变量命名规范
4. **新配置项**: 每个节点的`node_config`可以根据需要添加新的配置项

## 总结

这个DSL提供了一个完整的对话流程定义框架，通过节点和边的组合可以构建复杂的智能客服对话逻辑。其JSON格式易于解析和处理，同时具有良好的可读性和扩展性。