# Key-Value Store Reference (Beta)

Quicknode KV Store is a serverless storage service for lists and key-value sets. It is primarily used from inside Streams filter functions via the `qnLib` helper library but is also accessible via REST API and the `qn kv` CLI commands.

**Docs:** https://www.quicknode.com/docs/key-value-store

## Access Methods

| Method | When to Use |
|--------|-------------|
| `qnLib` (inside Streams) | Read/write KV from a Streams filter function |
| REST API | Read/write KV from any external application |
| CLI (`qn kv`) | Inspect and manage KV from the terminal |

## qnLib — Stream Integration

`qnLib` is available inside Streams filter functions without any import. All calls are asynchronous and should be awaited from an `async function main(...)` filter.

### List Operations

Manage ordered lists of string items (e.g., a watchlist of wallet addresses).

```javascript
// Create or overwrite a list
await qnLib.qnUpsertList('my-watchlist', {
  add_items: ['0xAddr1', '0xAddr2'],
});

// Add a single item
await qnLib.qnAddListItem('my-watchlist', '0xAddr3');

// Remove a single item
await qnLib.qnRemoveListItem('my-watchlist', '0xAddr1');

// Check membership (returns array of booleans, one per address)
const results = await qnLib.qnContainsListItems('my-watchlist', ['0xAddr2', '0xAddr3']);
// results → [true, true]

// Delete the entire list
await qnLib.qnDeleteList('my-watchlist');
```

### Set Operations

Manage key-value pairs (string keys, string values).

```javascript
// Create a key-value set entry
await qnLib.qnAddSet('threshold', '500000');

// Read a value by key
const threshold = await qnLib.qnGetSet('threshold');
// threshold → "500000"

// Bulk create or update multiple pairs
await qnLib.qnBulkSets({
  add_sets: {
    threshold: '500000',
    alertEmail: 'ops@example.com',
  },
  delete_sets: [],
});

// List all set keys
const setKeys = await qnLib.qnListAllSets();

// Delete a single set entry
await qnLib.qnDeleteSet('threshold');
```

## REST API

All REST requests use `https://api.quicknode.com/kv/rest/v1/` and authenticate via the `x-api-key` header.

### Read a value

```typescript
const response = await fetch(
  'https://api.quicknode.com/kv/rest/v1/sets/threshold',
  { headers: { 'x-api-key': process.env.QUICKNODE_API_KEY! } }
);
const { value } = await response.json();
```

### Write a value

```typescript
const response = await fetch('https://api.quicknode.com/kv/rest/v1/sets', {
  method: 'POST',
  headers: {
    'x-api-key': process.env.QUICKNODE_API_KEY!,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    key: 'threshold',
    value: '750000',
  }),
});
```

### List all set keys

```typescript
const response = await fetch(
  'https://api.quicknode.com/kv/rest/v1/sets',
  { headers: { 'x-api-key': process.env.QUICKNODE_API_KEY! } }
);
const { keys } = await response.json();
```

### List operations

```typescript
// Create a list
await fetch('https://api.quicknode.com/kv/rest/v1/lists', {
  method: 'POST',
  headers: {
    'x-api-key': process.env.QUICKNODE_API_KEY!,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    key: 'allowlist',
    items: ['0xabc', '0xdef'],
  }),
});

// Check one item from outside a Stream filter
const response = await fetch(
  'https://api.quicknode.com/kv/rest/v1/lists/allowlist/contains/0xabc',
  { headers: { 'x-api-key': process.env.QUICKNODE_API_KEY! } }
);
const { contains } = await response.json();
```

## CLI — `qn kv`

```bash
# Get a value
qn kv set get threshold

# Set a value
qn kv set put threshold 750000

# List all set keys
qn kv set ls

# Delete a key
qn kv set delete threshold

# Manage lists
qn kv list create allowlist 0xabc 0xdef
qn kv list append allowlist 0x123
qn kv list contains allowlist 0xabc
qn kv list get allowlist
qn kv list delete allowlist
```

## Limits

| Limit | Value |
|-------|-------|
| Max value size | 64 KB |
| Max list size | 10,000 items |
| Max key length | 256 bytes |

Limits are subject to change — check https://www.quicknode.com/docs/key-value-store for current values.

## Documentation

- **Key-Value Store Docs**: https://www.quicknode.com/docs/key-value-store
- **Quicknode CLI**: https://www.quicknode.com/docs/cli
- **Streams Integration**: https://www.quicknode.com/docs/streams
