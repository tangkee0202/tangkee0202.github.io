---
title: "PostgreSQL 与 JSONB：兼具关系型数据库的能力与文档灵活性"
description: "PostgreSQL 不只是 MongoDB 的替代品。学习 JSONB、GIN 索引、提取函数和查询运算符，同时获得关系模型与文档模型的优势。"
pubDatetime: 2026-01-22T10:00:00Z
tags:
  - postgresql
  - databases
  - backend
  - sql
draft: false
---

当 PostgreSQL 对 JSON 文档提供成熟支持后，“SQL 还是 NoSQL？”这个问题已经不再那么重要。使用 `JSONB`，可以在同一个数据库中，让需要约束的数据保持严格结构，同时让可变数据拥有文档式的灵活性。

## Table of contents

## `JSON` 与 `JSONB`：通常优先使用 JSONB

```sql
-- JSON：按原样存储文本
-- JSONB：使用处理后的二进制格式存储

-- JSONB 的优势：
-- ✓ 支持 GIN 索引，可实现高速查询
-- ✓ 移除多余空格和重复键
-- ✓ 支持包含运算符：@>、<@
-- ✗ 写入时需要解析，速度稍慢
-- ✗ 不保留键顺序和原始空格

CREATE TABLE events (
  id         BIGSERIAL PRIMARY KEY,
  type       TEXT NOT NULL,
  timestamp  TIMESTAMPTZ DEFAULT NOW(),
  payload    JSONB NOT NULL,             -- [!code highlight]
  metadata   JSONB DEFAULT '{}'::JSONB
);
```

## 基础插入与查询

```sql
-- 插入带有灵活载荷的事件
INSERT INTO events (type, payload) VALUES
  ('user.register', '{"name": "Ana Garcia", "plan": "pro", "country": "MX"}'),
  ('payment.completed', '{"amount": 99.99, "currency": "USD", "method": "card"}'),
  ('error.api',        '{"code": 429, "endpoint": "/api/v2/items", "ip": "10.0.0.1"}');

-- 使用 ->> 运算符提取字段
SELECT payload->>'name' AS name
FROM events
WHERE type = 'user.register';

-- 提取嵌套字段
SELECT payload->'address'->>'city' AS city
FROM events
WHERE type = 'user.register';

-- 按 JSON 内部的值筛选
SELECT * FROM events
WHERE type = 'payment.completed'
  AND (payload->>'amount')::NUMERIC > 50;
```

## GIN 索引：用 SQL 的速度查询 JSON

```sql
-- 为整个 JSONB 列创建 GIN 索引
CREATE INDEX idx_events_payload ON events USING GIN (payload);  -- [!code highlight]

-- 为特定键建立索引，效率更高
CREATE INDEX idx_events_payment_type ON events
  USING GIN ((payload->'method'));

-- 以下查询现在可以使用索引
SELECT * FROM events
WHERE payload @> '{"plan": "pro"}';      -- 包含该对象

SELECT * FROM events
WHERE payload ? 'code';                  -- 包含该键
```

## 包含运算符

```sql
-- @> 表示“包含”
SELECT * FROM events
WHERE payload @> '{"currency": "USD", "method": "card"}';

-- <@ 表示“被包含于”
SELECT '{"a": 1}'::JSONB <@ '{"a": 1, "b": 2}'::JSONB;  -- true

-- ? 表示“拥有这个键”
SELECT * FROM events WHERE payload ? 'code';

-- ?| 表示“拥有其中任意一个键”
SELECT * FROM events WHERE payload ?| ARRAY['name', 'email'];

-- ?& 表示“拥有全部这些键”
SELECT * FROM events WHERE payload ?& ARRAY['amount', 'currency'];
```

## 使用 `jsonb_set` 进行局部更新

与纯文档存储相比，一个重要优势是：无需重写整个文档就能更新其中一个字段。

```sql
-- 更新 JSONB 中的字段
UPDATE events
SET payload = jsonb_set(payload, '{plan}', '"enterprise"')  -- [!code highlight]
WHERE type = 'user.register'
  AND payload->>'name' = 'Ana Garcia';

-- 删除一个键
UPDATE events
SET payload = payload - 'ip'
WHERE type = 'error.api';

-- 向 JSONB 内部数组追加一项
UPDATE events
SET payload = jsonb_insert(payload, '{tags, -1}', '"urgent"')
WHERE type = 'error.api';
```

## 聚合函数：`jsonb_agg` 与 `jsonb_object_agg`

```sql
-- 按货币类型分组，并把付款记录聚合为 JSON 数组
SELECT
  payload->>'currency' AS currency,
  COUNT(*)           AS total_payments,
  jsonb_agg(payload) AS detail          -- [!code highlight]
FROM events
WHERE type = 'payment.completed'
GROUP BY currency;

-- 根据多行数据构建对象
SELECT jsonb_object_agg(type, COUNT(*))  -- [!code highlight]
FROM events
GROUP BY 1;
```

## 混合模式：同时发挥两种模型的优势

```sql
CREATE TABLE products (
  id          BIGSERIAL PRIMARY KEY,
  sku         TEXT UNIQUE NOT NULL,
  name        TEXT NOT NULL,
  price       NUMERIC(10,2) NOT NULL,
  category    TEXT NOT NULL,
  -- 结构化字段适合 JOIN、B-tree 索引和约束
  attributes  JSONB DEFAULT '{}',
  -- 灵活属性可以根据产品类别变化
  CHECK (price > 0)
);

-- 电子产品：{ "voltage": 220, "warranty_months": 24 }
-- 服装：    { "sizes": ["S","M","L"], "material": "cotton" }
-- 图书：    { "isbn": "...", "pages": 320 }

-- 同时利用普通列和 JSONB 属性
SELECT name, attributes->>'warranty_months' AS warranty
FROM products
WHERE category = 'electronics'
  AND (attributes->>'warranty_months')::INT >= 12
  AND price < 500;
```

> JSONB 不应替代关键字段的类型化列。一个实用原则是：如果某个字段经常用于 `JOIN`、`WHERE` 或 `ORDER BY`，就把它设计成普通列；如果它属于可变元数据，或者很少参与查询，再放入 JSONB。
