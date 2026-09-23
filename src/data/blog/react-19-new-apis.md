---
title: "React 19：useActionState、useOptimistic 与手动加载状态的终结"
description: "React 19 重新设计了表单、数据变更和过渡状态的处理方式。本文通过实例介绍这些新 API。"
pubDatetime: 2026-01-28T10:00:00Z
tags:
  - react
  - javascript
  - frontend
  - ux
draft: false
---

React 19 是自 Hooks 引入以来最重要的一次更新。它没有带来激进的新概念，而是为一个我们曾用无数种方式重复解决的问题提供了正式答案：**处理表单和数据变更**。

## Table of contents

## React 19 解决的问题

在 React 19 之前，一个同时包含加载反馈、错误处理和乐观更新的表单，通常需要写成这样：

```tsx file=before.tsx
// 过去：一个“基础”需求需要 35 行以上代码
function ProfileForm() {
  const [isPending, setIsPending] = useState(false); // [!code --]
  const [error, setError] = useState<string | null>(null); // [!code --]
  const [success, setSuccess] = useState(false); // [!code --]

  async function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    // [!code --]
    e.preventDefault(); // [!code --]
    setIsPending(true); // [!code --]
    setError(null); // [!code --]
    try {
      // [!code --]
      const data = new FormData(e.currentTarget); // [!code --]
      await updateProfile(data); // [!code --]
      setSuccess(true); // [!code --]
    } catch (err) {
      // [!code --]
      setError("保存失败"); // [!code --]
    } finally {
      // [!code --]
      setIsPending(false); // [!code --]
    } // [!code --]
  }
  // ...
}
```

## `useActionState`：不再手动维护表单状态

```tsx file=profile-form.tsx
import { useActionState } from "react"; // [!code ++]

async function updateProfileAction(prevState: State, formData: FormData) {
  try {
    await updateProfile({
      name: formData.get("name") as string,
      bio: formData.get("bio") as string,
    });
    return { success: true, error: null };
  } catch {
    return { success: false, error: "保存个人资料失败" };
  }
}

function ProfileForm() {
  const [state, action, isPending] = useActionState(
    // [!code highlight]
    updateProfileAction,
    { success: false, error: null }
  );

  return (
    <form action={action}>
      <input name="name" placeholder="姓名" />
      <textarea name="bio" placeholder="个人简介" />

      {state.error && <p className="error">{state.error}</p>}
      {state.success && <p className="success">保存成功！</p>}

      <button type="submit" disabled={isPending}>
        {isPending ? "正在保存……" : "保存"}
      </button>
    </form>
  );
}
```

## `useOptimistic`：即时界面与自动回滚

乐观更新是指在服务器确认之前先更新界面。过去，实现这一模式往往很繁琐，现在可以直接使用 `useOptimistic`：

```tsx file=todo-list.tsx
import { useOptimistic, useActionState } from "react";

function TodoList({ initialTodos }: { initialTodos: Todo[] }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    // [!code highlight]
    initialTodos,
    (state, newTodo: Todo) => [...state, newTodo]
  );

  async function addTodoAction(_: State, formData: FormData) {
    const title = formData.get("title") as string;

    // 立即更新界面
    addOptimisticTodo({ id: crypto.randomUUID(), title, done: false }); // [!code highlight]

    // 执行真实变更，失败时 Hook 会恢复状态
    await createTodo(title);
    return { error: null };
  }

  const [state, action, isPending] = useActionState(addTodoAction, {
    error: null,
  });

  return (
    <>
      <ul>
        {optimisticTodos.map(todo => (
          <li
            key={todo.id}
            style={{ opacity: todo.id.startsWith("temp") ? 0.5 : 1 }}
          >
            {todo.title}
          </li>
        ))}
      </ul>
      <form action={action}>
        <input name="title" required />
        <button disabled={isPending}>添加</button>
      </form>
    </>
  );
}
```

## `use()`：按条件读取 Promise 和 Context

```tsx file=user-profile.tsx
import { use, Suspense } from "react";

async function fetchUser(id: string): Promise<User> {
  const res = await fetch(`/api/users/${id}`);
  return res.json();
}

function UserProfile({ userPromise }: { userPromise: Promise<User> }) {
  const user = use(userPromise); // [!code highlight] 可以在条件分支中使用

  return <h1>{user.name}</h1>;
}

// Suspense 边界负责等待并解析 Promise
function App() {
  const userPromise = fetchUser("123"); // 在组件外创建

  return (
    <Suspense fallback={<p>正在加载用户……</p>}>
      <UserProfile userPromise={userPromise} />
    </Suspense>
  );
}
```

## Server Actions 实战

React 19 正式确立了 **Server Actions**：使用 `"use server"` 标记、并在服务器上执行的函数。

```tsx file=actions.ts
"use server";

import { revalidatePath } from "next/cache";
import { db } from "@/lib/db";

export async function deletePost(id: string) {
  await db.post.delete({ where: { id } });
  revalidatePath("/posts"); // 使服务器缓存失效 // [!code highlight]
}
```

```tsx file=post-card.tsx
import { deletePost } from "./actions";

export function PostCard({ post }: { post: Post }) {
  return (
    <article>
      <h2>{post.title}</h2>
      <form action={deletePost.bind(null, post.id)}>
        <button type="submit">删除</button>
      </form>
    </article>
  );
}
```

## 新 API 总结

| API                | 替代的旧方式                           | 适用场景                   |
| ------------------ | -------------------------------------- | -------------------------- |
| `useActionState`   | 表单中的 `useState` + `useReducer`     | 需要界面反馈的数据变更     |
| `useOptimistic`    | 手动编写回滚逻辑                       | 改善感知速度的即时更新     |
| `use(promise)`     | 使用 `useEffect` + `useState` 获取数据 | 组件在渲染时读取 Promise   |
| `use(context)`     | `useContext`                           | 需要按条件读取 Context     |
| 将 `ref` 作为 prop | `forwardRef`                           | 直接传递 ref，减少多余包装 |
