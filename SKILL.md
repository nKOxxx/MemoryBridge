---
name: memory-bridge
description: Long-term memory for AI agents. Store, query, and retrieve memories across sessions. Solves the "goldfish problem" where agents forget everything when sessions end. Use when the user wants to persist information between conversations, recall previous context, build user profiles, or maintain continuity across sessions. Works with both local SQLite (private, free) and Supabase (cloud, scalable) storage.
---

# Memory Bridge

**Long-term memory for AI agents.**

Solves the goldfish problem: agents that forget everything when sessions end or context windows fill up.

## Quick Start

```javascript
const MemoryBridge = require('memory-bridge');

// Initialize (SQLite = local & free)
const memory = new MemoryBridge({
  storage: 'sqlite',
  path: './memory.db'
});

// Store important insights
await memory.store("User prefers TypeScript over Python", {
  type: 'preference',
  importance: 9
});

// Retrieve later (even in new session)
const results = await memory.query("what user prefers");
// → [{content: "User prefers TypeScript over Python", ...}]
```

## Installation

```bash
npm install memory-bridge
```

## OpenClaw Integration

### Before Memory Bridge
```javascript
// Session 1
User: "I prefer dark mode"
Agent: "Noted!"  // But doesn't actually remember

// Session 2 (new session)
User: "Why is UI light?"
Agent: "What do you mean?"  // Completely forgot
```

### After Memory Bridge
```javascript
// Session 1 - Store the preference
await memory.store("User prefers dark mode", {
  type: 'preference',
  importance: 8
});

// Session 2 - New session, but memory persists
const prefs = await memory.query("user preferences");
// Agent: "I'll use dark mode like you prefer"
```

## API Reference

### Initialize

```javascript
const MemoryBridge = require('memory-bridge');

// Option 1: SQLite (default, local, private)
const memory = new MemoryBridge({
  storage: 'sqlite',
  path: './data/memory.db'
});

// Option 2: Supabase (cloud, shareable)
const memory = new MemoryBridge({
  storage: 'supabase',
  supabaseUrl: 'https://your-project.supabase.co',
  supabaseKey: 'your-service-key'
});
```

### Store Memory

```javascript
await memory.store(content, options)

// Simple
await memory.store("Important information");

// With metadata
await memory.store("User prefers TypeScript", {
  type: 'preference',      // 'insight', 'preference', 'error', 'goal', 'decision'
  importance: 9,           // 1-10 scale (auto-calculated if omitted)
  source: 'conversation',  // Where it came from
  agentId: 'my-agent'      // For multi-agent (default: 'default')
});
```

### Query Memories

```javascript
const results = await memory.query(queryString, options);

// Simple search
const results = await memory.query("TypeScript preferences");

// With filters
const results = await memory.query("project goals", {
  limit: 5,              // Max results (default: 5)
  days: 30,              // Search last N days (default: 30)
  minImportance: 7,      // Filter by importance (default: 0)
  agentId: 'my-agent'    // For multi-agent
});

// Results format
[
  {
    id: "123-abc",
    content: "User prefers TypeScript",
    content_type: "preference",
    importance: 9,
    relevance: 0.95,       // Match score
    created_at: "2026-02-19T10:00:00Z",
    metadata: { keywords: ['typescript', 'preference', 'user'] }
  }
]
```

### Timeline View

```javascript
// Get memories grouped by date
const timeline = await memory.timeline(days, options);

const week = await memory.timeline(7);
// {
//   "2026-02-19": [memory1, memory2],
//   "2026-02-18": [memory3]
// }
```

## CLI Usage

```bash
# Initialize in current directory
npx memory-bridge init

# Store memory
npx memory-bridge store "User prefers dark mode" --type=preference --importance=8

# Query
npx memory-bridge query "user preferences"

# Timeline
npx memory-bridge timeline 7
```

## Integration Patterns

### Pattern 1: Session Start (Context Loading)
```javascript
async function startSession() {
  // Load recent context
  const recent = await memory.query("recent work", { limit: 5, days: 7 });
  const goals = await memory.query("current goals", { minImportance: 8 });
  const prefs = await memory.query("user preferences");
  
  return buildContext(recent, goals, prefs);
}
```

### Pattern 2: During Conversation (Auto-Store)
```javascript
async function handleMessage(userMsg, response) {
  // Store important insights automatically
  if (isImportant(userMsg)) {
    await memory.store(userMsg, {
      type: 'insight',
      importance: calculateImportance(userMsg)
    });
  }
  
  // Enrich response with memory context
  const relevant = await memory.query(userMsg, { limit: 3 });
  return generateResponse(userMsg, response, relevant);
}
```

### Pattern 3: User Profile Building
```javascript
async function buildUserProfile() {
  const preferences = await memory.query("preference", { limit: 20 });
  const goals = await memory.query("goal", { limit: 10 });
  const decisions = await memory.query("decision", { limit: 10 });
  
  return {
    preferences: extractPatterns(preferences),
    goals: goals.map(g => g.content),
    decisionHistory: decisions
  };
}
```

## Storage Modes

| Feature | SQLite | Supabase |
|---------|--------|----------|
| Setup | Zero config | Requires URL/key |
| Privacy | 100% local | Your Supabase instance |
| Cost | Free | Supabase pricing |
| Sync | None | Built-in |
| Share | File copy | Multi-agent access |
| Best for | Personal agents | Teams, production |

## Smart Features

### Automatic Keyword Extraction
Content is automatically analyzed for keywords using NLP:
```javascript
await memory.store("Working on 2ndCTO security audit");
// Auto-extracts: ['2ndCTO', 'security', 'audit']
```

### Relevance Scoring
Query results ranked by keyword match + importance:
```javascript
// Query "2ndCTO" finds memories with:
// - Exact keyword matches
// - Related terms (security, audit)
// - Weighted by importance
```

### Importance Auto-Calculation
```javascript
// Default = 5
// +1 if content > 200 chars
// +2 if type = 'insight'
// +1 if type = 'error'
// +3 if type = 'security'
// +2 if type = 'goal'
```

## Security

### Data Protection
- **SQLite**: Data stays on your machine
- **Supabase**: Your own database (you control)
- **Encryption**: At-rest in both modes

### Best Practices
```javascript
// ✓ Good: Environment variables
const memory = new MemoryBridge({
  supabaseKey: process.env.SUPABASE_KEY
});

// ✗ Bad: Hardcoded keys
const memory = new MemoryBridge({
  supabaseKey: 'sb-abc-123'  // Never do this
});
```

## Error Handling

```javascript
try {
  await memory.store(content);
} catch (err) {
  if (err.message.includes('credentials')) {
    // Config error - check credentials
  } else if (err.message.includes('connection')) {
    // Network error - retry
  }
}
```

## GitHub

**Repository:** https://github.com/nKOxxx/MemoryBridge

**Issues/PRs welcome.**

---

**Built for the agent economy. Infrastructure that remembers.** 🧠
