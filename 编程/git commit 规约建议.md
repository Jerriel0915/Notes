## 核心格式

每次提交的 Commit 信息都应包含以下结构：
```text
<type>(<scope>): <subject>
// 空一行
<body>
// 空一行
<footer>
```

- **Header (必填):** 包含 `type`（类型）、`scope`（作用域）和 `subject`（简述）。
- **Body (选填):** 详细说明，解释 **“为什么做”** 和 **“主要变动是什么”**。
- **Footer (选填):** 关联 Issue 编号或备注不兼容的破坏性变更（Breaking Change）。

---

## 一、 Type (提交类型)

企业开发中最常用的标准类型列表：

- ** `feat` **: 新增功能 (Feature)
    
- ** `fix` **: 修复 Bug
    
- ** `docs` **: 仅修改文档 (Documentation)
    
- ** `style` **: 代码格式调整（不影响逻辑，如空格、缩进等）
    
- ** `refactor` **: 代码重构（既不是新增功能，也不是修复 Bug）
    
- ** `perf` **: 性能优化 (Performance)
    
- ** `test` **: 增加或修改单元测试/集成测试
    
- ** `chore` **: 构建过程或辅助工具的变动（如更新依赖库）
    
- ** `revert` **: 代码回滚
    

## 二、 Scope (作用域 - 选填但推荐)

标明本次提交影响的范围。通常是**模块名**、**包名**或**架构分层**。

- _例：`db` , `cli` , `parser` , `auth` , `router` _
    

## 三、 Subject (简述 - 必填)

对变更的简短描述，遵循以下黄金法则：

1. **动词开头**：使用祈使句（如 `add` , `fix` , `update`，避免使用 `added` , `fixes`）。
    
2. **全小写**：首字母不强制大写，保持统一。
    
3. **言简意赅**：控制在 50 个字符以内。
    
4. **结尾无句号**。
    

---

## 四、 实战案例 (结合后端与底层开发场景)

#### 1. 新增功能 (Feature)
```text
feat(cli): implement Trie tree for tab completion
```

#### 2. 修复 Bug (Fix)
```text
fix(db): resolve MVCC phantom read in transaction block
```

#### 3. 性能优化 (Performance)
```text
perf(gc): optimize JVM memory layout to reduce full GC frequency
```

#### 4. 补充测试 (Test)
```text
test(parser): add unit tests for prefix matching logic
```

#### 5. 完整的企业级 Commit 示例 (包含 Body 和 Footer)
``` text
feat(coupon): add high-concurrency user claiming logic

- Implement Redis Lua scripting for atomic stock deduction.
- Add message queue publisher for asynchronous order generation.
- Ensure database transaction isolation level is correctly handled.

Closes #ISSUE-4092
```

---

## 五、 避坑指南

- **❌ 拒绝无意义提交：** `update` , `fix bug` , `test` , `aaa`。
- **❌ 拒绝巨型提交：** 不要把 5 个无关的 Bug 修复放在一个 `fix` 提交里。遵循 **“原子化提交”**（一个 Commit 只做一件事）。
- **⚠️ 破坏性变更：** 如果改动会导致现有 API 或功能不兼容，务必在 Footer 处以 `BREAKING CHANGE:` 开头并详细说明。