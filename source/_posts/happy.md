---
title: 揭秘 Happy：如何实现 AI 编程助手输出的实时同步
enlink: unveil-happy-real-time-ai-assistant-sync
date: 2026-03-04 09:58:06
categories:
- AI
tags:
- vibecoding
- claude
- happy
- codex
---

> 深入分析跨平台 AI 会话同步的架构设计与实现细节

## 引言

在 AI 编程助手（如 Claude Code、Codex）日益普及的今天，一个常见需求是：**如何让移动端实时查看和控制桌面端的 AI 会话？** 这就是 [Happy](https://github.com/slopus/happy) 解决的问题。

Happy 是一个三部分组成的系统，能够精确捕获 Claude Code、Codex 等工具的输出，并通过服务端实时同步到移动端 App。本文将深入剖析其技术实现。

---

## 系统架构概览

Happy 采用经典的三层架构：

```
┌─────────────────────────────────────────────────────────────┐
│                        happy-app                            │
│                  (React Native 移动端)                       │
└──────────────────────────┬──────────────────────────────────┘
                           │ WebSocket
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                      happy-server                           │
│              (Fastify + Socket.io + PostgreSQL)             │
└──────────────────────────┬──────────────────────────────────┘
                           │ WebSocket
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                        happy-cli                            │
│              (Claude Code / Codex 包装器)                    │
└─────────────────────────────────────────────────────────────┘
```

核心挑战：**如何让 CLI 端精确捕获不同 AI 工具的输出，并实时推送到移动端？**

---

## 一、捕获 Claude Code 输出：文件监听机制

Claude Code 会将会话历史写入本地 JSONL 文件：

```
~/.claude/projects/{projectPath}/{sessionId}.jsonl
```

Happy 的核心洞察是：**与其尝试拦截进程输出，不如监听 Claude 自己维护的会话文件。**

### 1.1 Session Scanner 实现

```typescript
// packages/happy-cli/src/claude/utils/sessionScanner.ts

export async function createSessionScanner(opts: {
    sessionId: string | null,
    workingDirectory: string
    onMessage: (message: RawJSONLines) => void
}) {
    const projectDir = getProjectPath(opts.workingDirectory);
    const processedMessageKeys = new Set<string>();

    // 使用 InvalidateSync 实现高效同步
    const sync = new InvalidateSync(async () => {
        const sessions = collectActiveSessions();

        for (let session of sessions) {
            const messages = await readSessionLog(projectDir, session);

            for (let message of messages) {
                const key = messageKey(message);
                if (processedMessageKeys.has(key)) continue;

                processedMessageKeys.add(key);
                opts.onMessage(message);  // 回调发送新消息
            }
        }
    });

    // 文件变更监听 + 定期同步
    const watcher = startFileWatcher(
        join(projectDir, `${sessionId}.jsonl`),
        () => sync.invalidate()  // 文件变化时立即触发
    );

    setInterval(() => sync.invalidate(), 3000);  // 兜底同步
}
```

### 1.2 去重机制

由于文件监听可能触发多次，Happy 使用多重去重策略：

```typescript
function messageKey(message: RawJSONLines): string {
    if (message.type === 'user') return message.uuid;
    if (message.type === 'assistant') return message.uuid;
    if (message.type === 'summary') return `summary:${message.leafUuid}`;
    return message.uuid;
}
```

**关键设计**：每条消息都有唯一标识（UUID），确保同一条消息不会被重复处理。

---

## 二、捕获思考状态：自定义文件描述符

Claude Code 的"思考中"状态（显示用户正在等待响应）如何捕获？Happy 使用了一个巧妙的技巧：**通过自定义文件描述符与启动器脚本通信**。

### 2.1 启动器脚本拦截

```typescript
// packages/happy-cli/src/claude/claudeLocal.ts

const child = spawn(
    'node',
    [claudeCliPath, ...args],
    {
        // fd 0: stdin, fd 1: stdout, fd 2: stderr, fd 3: 自定义通信
        stdio: ['inherit', 'inherit', 'inherit', 'pipe'],
    }
);

// 监听 fd 3 获取思考状态
if (child.stdio[3]) {
    const rl = createInterface({
        input: child.stdio[3],
        crlfDelay: Infinity
    });

    const activeFetches = new Map();

    rl.on('line', (line) => {
        const message = JSON.parse(line);

        switch (message.type) {
            case 'fetch-start':
                activeFetches.set(message.id, {
                    hostname: message.hostname,
                    path: message.path
                });
                updateThinking(true);  // 开始思考
                break;

            case 'fetch-end':
                activeFetches.delete(message.id);
                if (activeFetches.size === 0) {
                    updateThinking(false);  // 结束思考
                }
                break;
        }
    });
}
```

**原理**：启动器脚本拦截 Claude 的 HTTP 请求，通过 fd 3 发送 fetch-start/fetch-end 事件，Happy 据此判断思考状态。

---

## 三、统一多代理协议：ACP (Agent Communication Protocol)

Claude Code 使用文件记录，但 Codex、Gemini 等其他代理呢？Happy 实现了 **ACP 后端**，通过官方 SDK 标准化通信。

### 3.1 ACP 连接建立

```typescript
// packages/happy-cli/src/agent/acp/AcpBackend.ts

import {
    ClientSideConnection,
    ndJsonStream,
} from '@agentclientprotocol/sdk';

this.process = spawn(this.options.command, args, {
    cwd: this.options.cwd,
    stdio: ['pipe', 'pipe', 'pipe'],  // stdin, stdout, stderr
});

// Node Stream → Web Stream
const streams = nodeToWebStreams(
    this.process.stdin,
    this.process.stdout
);
const stream = ndJsonStream(streams.writable, streams.readable);

// 创建 JSON-RPC 连接
this.connection = new ClientSideConnection(
    (agent: Agent) => client,
    stream
);
```

### 3.2 统一消息格式

不同代理的消息被转换为统一的 ACP 格式：

```typescript
export type ACPMessageData =
    | { type: 'message'; message: string }
    | { type: 'reasoning'; message: string }
    | { type: 'thinking'; text: string }
    | { type: 'tool-call'; callId: string; name: string; input: unknown }
    | { type: 'tool-result'; callId: string; output: unknown }
    | { type: 'file-edit'; filePath: string; diff?: string }
    | { type: 'permission-request'; permissionId: string; toolName: string };
```

---

## 四、会话协议映射：从原始消息到结构化信封

捕获原始消息后，Happy 使用 **Session Protocol Mapper** 将其转换为结构化的 Session Envelope。

### 4.1 协议映射核心逻辑

```typescript
// packages/happy-cli/src/claude/utils/sessionProtocolMapper.ts

export function mapClaudeLogMessageToSessionEnvelopes(
    message: RawJSONLines,
    state: ClaudeSessionProtocolState
): { envelopes: SessionEnvelope[]; currentTurnId: string | null } {

    const envelopes: SessionEnvelope[] = [];

    if (message.type === 'assistant') {
        const turnId = ensureTurn(state, envelopes);
        const blocks = message.message.content || [];

        for (const block of blocks) {
            // 文本内容
            if (block.type === 'text') {
                envelopes.push(createEnvelope('agent', {
                    t: 'text',
                    text: block.text
                }, { turn: turnId }));
            }

            // 思考过程
            if (block.type === 'thinking') {
                envelopes.push(createEnvelope('agent', {
                    t: 'text',
                    text: block.thinking,
                    thinking: true
                }, { turn: turnId }));
            }

            // 工具调用开始
            if (block.type === 'tool_use') {
                envelopes.push(createEnvelope('agent', {
                    t: 'tool-call-start',
                    call: block.id,
                    name: block.name,
                    title: toolTitle(block.name, block.input),
                    args: block.input
                }, { turn: turnId }));
            }
        }
    }

    if (message.type === 'user') {
        // 工具调用结束（通过 tool_result）
        for (const block of message.message.content) {
            if (block.type === 'tool_result') {
                envelopes.push(createEnvelope('agent', {
                    t: 'tool-call-end',
                    call: block.tool_use_id
                }, { turn: turnId }));
            }
        }

        // 用户消息
        if (typeof message.message.content === 'string') {
            envelopes.push(createEnvelope('user', {
                t: 'text',
                text: message.message.content
            }));
        }
    }

    return { envelopes, currentTurnId: state.currentTurnId };
}
```

### 4.2 子代理（Subagent）追踪

Claude 的 Task 工具会创建子对话，Happy 需要追踪这些"sidechain"：

```typescript
// 当检测到 Task 工具调用时
if (block.name === 'Task') {
    const prompt = pickTaskPrompt(block.input);
    const subagentId = ensureSessionSubagentId(state, block.id);

    // 缓冲后续消息，直到子代理启动
    queueTaskPromptSubagent(state, prompt, block.id);

    // 消费缓冲的消息
    const buffered = consumeBufferedSubagentMessages(state, block.id);
    for (const msg of buffered) {
        const replay = mapClaudeLogMessageToSessionEnvelopes(msg, state);
        envelopes.push(...replay.envelopes);
    }
}
```

---

## 五、服务端消息路由与广播

Happy 服务端使用 **Event Router** 模式管理消息分发。

### 5.1 三种连接类型

```typescript
// packages/happy-server/sources/app/api/socket.ts

type ClientConnection =
    // CLI 代理连接（按会话隔离）
    | { connectionType: 'session-scoped'; sessionId: string; ... }
    // 移动应用连接（接收所有会话更新）
    | { connectionType: 'user-scoped'; ... }
    // 守护进程连接（报告机器状态）
    | { connectionType: 'machine-scoped'; machineId: string; ... }
```

### 5.2 消息处理流程

```typescript
// packages/happy-server/sources/app/api/socket/sessionUpdateHandler.ts

socket.on('message', async (data) => {
    const { sid, message, localId } = data;

    // 1. 验证并存储到数据库
    const msg = await db.sessionMessage.create({
        data: {
            sessionId: sid,
            seq: await allocateSessionSeq(sid),
            content: { t: 'encrypted', c: message },
            localId  // 用于去重
        }
    });

    // 2. 广播给所有订阅者
    eventRouter.emitUpdate({
        userId,
        payload: buildNewMessageUpdate(msg, sid, updSeq, key),
        recipientFilter: {
            type: 'all-interested-in-session',
            sessionId: sid
        },
        skipSenderConnection: connection  // 不回发给发送者
    });
});
```

### 5.3 持久化 vs 临时消息

```typescript
class EventRouter {
    // 持久化事件（存储到 DB，可回放）
    emitUpdate({ userId, payload, recipientFilter }): void;

    // 临时事件（仅实时推送，如思考状态）
    emitEphemeral({ userId, payload, recipientFilter }): void;
}
```

---

## 六、精确传递等待操作：消息队列与权限处理

AI 工具常需要用户确认（如"是否允许编辑文件？"），Happy 需要**延迟发送工具调用消息，直到权限响应到达**。

### 6.1 出站消息队列

```typescript
// packages/happy-cli/src/claude/utils/OutgoingMessageQueue.ts

export class OutgoingMessageQueue {
    private queue: Array<{
        message: RawJSONLines;
        delay?: number;
        toolCallIds?: string[];  // 关联的工具调用
    }> = [];

    // 普通消息立即发送
    enqueue(message: RawJSONLines) {
        this.sender(message);
    }

    // 工具调用消息延迟发送
    enqueueWithDelay(
        message: RawJSONLines,
        delayMs: number,
        toolCallIds: string[]
    ) {
        this.queue.push({ message, delay, toolCallIds });
    }

    // 权限响应到达后释放
    releaseToolCall(toolCallId: string) {
        for (let i = 0; i < this.queue.length; i++) {
            const item = this.queue[i];
            if (item.toolCallIds?.includes(toolCallId)) {
                this.sender(item.message);
                this.queue.splice(i, 1);
            }
        }
    }
}
```

### 6.2 权限处理集成

```typescript
// packages/happy-cli/src/claude/claudeRemoteLauncher.ts

const messageQueue = new OutgoingMessageQueue(
    (msg) => session.client.sendClaudeSessionMessage(msg)
);

// 工具调用时延迟发送
if (message.type === 'assistant') {
    const toolCallIds = extractToolCallIds(message);
    if (toolCallIds.length > 0 && !isSidechain) {
        messageQueue.enqueue(logMessage, {
            delay: 250,
            toolCallIds
        });
    }
}

// 工具结果到达时释放
if (message.type === 'user') {
    for (const block of message.message.content) {
        if (block.type === 'tool_result') {
            messageQueue.releaseToolCall(block.tool_use_id);
        }
    }
}
```

---

## 七、端到端加密

所有消息都经过 **TweetNaCl/libsodium** 端到端加密。

### 7.1 CLI 端加密发送

```typescript
// packages/happy-cli/src/api/apiSession.ts

private enqueueMessage(content: unknown) {
    const encrypted = encodeBase64(
        encrypt(this.encryptionKey, this.encryptionVariant, content)
    );

    this.pendingOutbox.push({
        content: encrypted,
        localId: randomUUID()  // 用于去重和确认
    });

    this.sendSync.invalidate();  // 触发发送
}
```

### 7.2 App 端解密接收

```typescript
// packages/happy-app/sources/sync/sync.ts

const body = decrypt(
    sessionKey,
    encryptionVariant,
    decodeBase64(message.content.c)
);

this.routeIncomingMessage(body);
```

**安全特性**：
- 服务端只存储加密数据，无法读取内容
- 每个会话使用独立的加密密钥
- 支持密钥轮换（legacy → dataKey）

---

## 八、App 端状态重建

移动端使用 **Reducer** 模式从消息流重建会话状态。

### 8.1 Reducer 核心逻辑

```typescript
// packages/happy-app/sources/sync/reducer/reducer.ts

export function messageReducer(
    state: ReducerState,
    messages: Message[]
): ReducerState {

    // Phase 0: 处理 AgentState 中的权限请求
    for (const permission of agentState.permissions) {
        state = processPermissionRequest(state, permission);
    }

    // Phase 1: 创建或更新消息
    for (const message of messages) {
        switch (message.role) {
            case 'user':
                state = addUserMessage(state, message);
                break;
            case 'agent':
                state = addAgentMessage(state, message);
                break;
            case 'session':
                state = addSessionEvent(state, message);
                break;
        }
    }

    // Phase 2: 清理过期权限占位符
    state = cleanupStalePermissions(state);

    return state;
}
```

### 8.2 消息去重策略

```typescript
// 多层级去重
if (message.localId && state.seenLocalIds.has(message.localId)) {
    continue;  // 跳过重复的用户消息
}

if (message.id && state.seenMessageIds.has(message.id)) {
    continue;  // 跳过已处理的消息
}

if (permissionId && state.seenPermissionIds.has(permissionId)) {
    continue;  // 跳过重复的权限请求
}
```

---

## 九、关键技术洞察

### 9.1 为什么选择文件监听而非进程拦截？

| 方案 | 优点 | 缺点 |
|------|------|------|
| **进程拦截** (PTY) | 实时性好 | 复杂度高，易受终端控制序列干扰 |
| **文件监听** | 简单可靠，天然持久化 | 有毫秒级延迟 |

Happy 选择文件监听，因为 Claude Code 本身就需要持久化会话历史，监听文件既可靠又避免了 PTY 的复杂性。

### 9.2 如何处理会话恢复？

Claude Code 支持 `--resume` 和 `--continue` 标志恢复会话，此时会创建**新的会话文件**。Happy 的 Session Scanner 会：

1. 检测到新会话 ID
2. 启动对新文件的监听
3. **保留旧文件的监听**（因为某些更新可能仍写入原文件）
4. 使用 `processedMessageKeys` 确保跨文件去重

### 9.3 InvalidateSync：高性能同步原语

Happy 大量使用 `InvalidateSync` 类实现高效的批量同步：

```typescript
class InvalidateSync {
    private invalid = false;
    private running = false;

    invalidate() {
        this.invalid = true;
        if (!this.running) this.run();
    }

    private async run() {
        this.running = true;
        while (this.invalid) {
            this.invalid = false;
            await this.syncFunction();  // 执行同步
        }
        this.running = false;
    }
}
```

**优势**：快速连续调用 `invalidate()` 只会触发一次同步，避免资源浪费。

---

## 十、总结

Happy 的架构设计体现了以下核心原则：

1. **适配而非改造**：利用 Claude Code 已有的会话文件机制，而非尝试拦截进程 I/O
2. **协议抽象**：通过 ACP 和 Session Envelope 统一不同 AI 工具的输出格式
3. **延迟一致性**：使用消息队列和权限等待机制，确保工具调用的顺序正确
4. **端到端安全**：所有数据在客户端加密，服务端仅作为中继
5. **最终一致性**：移动端通过 Reducer 从消息流重建状态，支持离线后同步

这种架构使 Happy 能够可靠地同步复杂的 AI 会话状态，包括思考过程、工具调用、权限确认等细节，为用户提供无缝的跨端体验。

---

## 参考链接

- [Happy GitHub 仓库](https://github.com/slopus/happy)
- [Claude Code 官方文档](https://docs.anthropic.com/en/docs/claude-code)
- [Agent Communication Protocol](https://github.com/AgentClientProtocol/acp)
- [TweetNaCl.js 加密库](https://tweetnacl.js.org/)

---

*本文基于 Happy 开源项目代码分析撰写，代码版本截至 2025 年 3 月。*

