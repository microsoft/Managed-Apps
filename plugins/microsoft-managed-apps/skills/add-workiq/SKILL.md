---
name: add-workiq
description: Adds Work IQ Copilot MCP by delegating to /add-connector with api-id=shared_a365copilotchatmcp and action mode. Use when users need Microsoft 365 knowledge-grounded Work IQ search/chat.
user-invocable: true
allowed-tools: Read, AskUserQuestion, Skill
model: sonnet
---

**📋 Shared Instructions: [shared-instructions.md](${CLAUDE_PLUGIN_ROOT}/shared/shared-instructions.md)** — Cross-cutting concerns.

# Add Work IQ Copilot MCP (Wrapper)

This skill is a thin wrapper. Use `/add-connector` as the single implementation path.

## Delegation contract

Invoke `/add-connector` with:

- `api-id`: `shared_a365copilotchatmcp`
- `mode`: `action`

---

# Work IQ Integration: MCP Session Pattern

After the connector is added, you'll have access to `WorkIQCopilotMCPService`. **Work IQ uses MCP (Model Context Protocol), a stateful protocol. Use the `McpSession` wrapper class to manage this properly.**

## Setup: Create McpSession Wrapper

Work IQ requires session initialization and protocol handling. Create `src/connectors/mcpClient.ts`:

```typescript
import type { IOperationResult } from '@microsoft/managed-apps/data'
import { WorkIQCopilotMCPService } from '../generated/services/WorkIQCopilotMCPService'

interface JsonRpcResponse {
  jsonrpc?: string
  result?: Record<string, unknown>
  error?: { code?: number; message?: string }
}

export class McpSession {
  private sessionId: string | undefined
  private initialized = false

  private extractSessionId(raw: IOperationResult<unknown>): string | undefined {
    const dataObj = raw.data && typeof raw.data === 'object' ? raw.data : undefined
    const candidates = [
      dataObj?.['Mcp-Session-Id'],
      dataObj?.sessionId,
      (dataObj?.result as Record<string, unknown>)?.['Mcp-Session-Id'],
    ]
    return candidates.find((v) => typeof v === 'string') as string | undefined
  }

  async initialize(): Promise<void> {
    const req = {
      jsonrpc: '2.0',
      id: '1',
      method: 'initialize',
      params: {
        protocolVersion: '2025-06-18',
        capabilities: {},
        clientInfo: { name: 'Your App', version: '1.0.0' },
      },
    }

    const raw = await WorkIQCopilotMCPService.mcp_m365copilot(undefined, req as any)

    if (!raw.success) throw new Error(raw.error?.message)

    this.sessionId = this.extractSessionId(raw)
    this.initialized = true
  }

  async callCopilotChat(message: string): Promise<{ text: string; conversationId?: string }> {
    if (!this.initialized) await this.initialize()

    const req = {
      jsonrpc: '2.0',
      id: '2',
      method: 'tools/call',
      params: {
        name: 'CopilotChat',
        arguments: { message },
      },
    }

    const raw = await WorkIQCopilotMCPService.mcp_m365copilot(this.sessionId, req as any)

    if (!raw.success) throw new Error(raw.error?.message)

    const rpc = raw.data as JsonRpcResponse
    const text = this.extractText(rpc)
    const conversationId = this.extractConversationId(rpc)

    return { text, conversationId }
  }

  private extractText(rpc: JsonRpcResponse): string {
    const content = rpc.result?.content as any
    if (Array.isArray(content)) {
      const textBlock = content.find((c) => c.type === 'text')
      if (textBlock?.text) return textBlock.text
    }
    return rpc.error?.message ?? '(no response)'
  }

  private extractConversationId(rpc: JsonRpcResponse): string | undefined {
    const content = rpc.result?.content?.[0]?.text
    if (typeof content === 'string') {
      try {
        const json = JSON.parse(content)
        return json.conversationId
      } catch {}
    }
    return undefined
  }
}
```

## Usage: Call via McpSession

Initialize once per app and call via the wrapper:

```typescript
import { McpSession } from './connectors/mcpClient'

// Initialize once per app
const workIqSession = new McpSession()

// Call with a well-structured prompt
const { text } = await workIqSession.callCopilotChat(`
You are a professional meeting summarizer.

Meeting: ${meeting.subject}
Organizer: ${meeting.organizer}
Time: ${meeting.start} to ${meeting.end}

Format your response with:
## Summary
- 4-6 bullets on key points

## Action Items
- Owner: task (Due: date)

Keep under 180 words.
`)
```

## Prompt Structure

Always provide explicit format guidance so responses are predictable:

```typescript
// Structured prompt = predictable response format
const prompt = `
You are preparing a meeting recap.

Context:
- Meeting: ${meeting.subject}
- Attendees: ${attendeeCount}
- Duration: ${duration} minutes

Task: Return markdown with:
1. ## Summary (4-6 bullets)
2. ## Action Items (- Owner: task (Due: date))

Limit 200 words.
`

const { text } = await workIqSession.callCopilotChat(prompt)

// Now parsing is reliable
const summary = text.match(/##\s*Summary\s*([\s\S]*?)(?=##|$)/)?.[1]?.trim()
const actionItems = text.match(/##\s*Action Items\s*([\s\S]*?)$/)
```

## Response Parsing

Work IQ returns markdown-formatted text. Parse the structured format:

```typescript
const { text } = await workIqSession.callCopilotChat(prompt)

// Extract sections using regex
const summaryMatch = text.match(/##\s*Summary\s*([\s\S]*?)(?=##|$)/)
const actionItemsMatch = text.match(/##\s*Action Items\s*([\s\S]*?)$/)

const summary = summaryMatch ? summaryMatch[1].trim() : text
const actionItems = actionItemsMatch
  ? actionItemsMatch[1]
      .split('\n')
      .filter(line => line.trim().startsWith('-'))
      .map(line => line.replace(/^-\s*/, ''))
  : []
```

## Error Handling

Surface errors to users:

```typescript
try {
  const { text } = await workIqSession.callCopilotChat(prompt)
  
  if (text.includes('Error:')) {
    throw new Error(text)
  }
  
  setSummary({ text, error: undefined })
} catch (error) {
  setError(error instanceof Error ? error.message : 'Failed to generate summary')
  setSummary({ text: '' })
}
```

## Key Requirements

- ✅ Use McpSession wrapper (handles initialization + protocol)
- ✅ Initialize once per app session
- ✅ Structure prompts with explicit format guidance (markdown sections)
- ✅ Call `workIqSession.initialize()` automatically on first `callCopilotChat()`
- ✅ Reuse session for multiple calls
- ✅ Surface errors by throwing
- ✅ Parse structured markdown responses

## Why McpSession is Required

Work IQ (MCP protocol) is **stateful**:
- Requires session initialization before first call (handshake)
- Session ID must be tracked and reused across calls
- Response format is nested JSON-RPC protocol (not direct REST data)

The McpSession wrapper class handles all this complexity, so you just call `callCopilotChat(message)` and get back plain text.
