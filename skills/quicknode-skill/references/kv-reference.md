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

`qnLib` is available inside Streams filter functions without any import. All calls are synchronous within the filter context.

### List Operations

Manage ordered lists of string items (e.g., a watchlist of wallet addresses).

```javascript
// Create or overwrite a list
qnLib.qnUpsertList('my-watchlist', ['0xAddr1', '0xAddr2']);

// Add a single item
qnLib.qnAddListItem('my-watchlist', '0xAddr3');

// Remove a single item
qnLib.qnRemoveListItem('my-watchlist', '0xAddr1');

// Check membership (returns array of booleans, one per address)
const results = qnLib.qnContainsListItems('my-watchlist', ['0xAddr2', '0xAddr3']);
// results → [true, true]

// Delete the entire list
qnLib.qnDeleteList('my-watchlist');
```

### Set Operations

Manage key-value pairs (string keys, string values).

```javascript
// Create a key-value set entry
qnLib.qnAddSet('config', 'threshold', '500000');

// Read a value by key
const threshold = qnLib.qnGetSet('config', 'threshold');
// threshold → "500000"

// Bulk create or update multiple pairs
qnLib.qnBulkSets('config', [
  { key: 'threshold', value: '500000' },
  { key: 'alertEmail', value: 'ops@example.com' },
]);

// Delete a single set entry
qnLib.qnDeleteSet('config', 'threshold');
```

## REST API

All REST requests use `https://api.quicknode.com/kv/rest/v1/` and authenticate via the `x-api-key` header.

### Read a value

```typescript
const response = await fetch(
  'https://api.quicknode.com/kv/rest/v1/get?storeName=config&key=threshold',
  { headers: { 'x-api-key': process.env.QUICKNODE_API_KEY! } }
);
const { value } = await response.json();
```

### Write a value

```typescript
const response = await fetch('https://api.quicknode.com/kv/rest/v1/set', {
  method: 'POST',
  headers: {
    'x-api-key': process.env.QUICKNODE_API_KEY!,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    storeName: 'config',
    key: 'threshold',
    value: '750000',
  }),
});
```

### List all keys in a store

```typescript
const response = await fetch(
  'https://api.quicknode.com/kv/rest/v1/list?storeName=config',
  { headers: { 'x-api-key': process.env.QUICKNODE_API_KEY! } }
);
const { keys } = await response.json();
```

## CLI — `qn kv`

```bash
# Get a value
qn kv get --store config --key threshold

# Set a value
qn kv set --store config --key threshold --value 750000

# List all keys in a store
qn kv list --store config

# Delete a key
qn kv delete --store config --key threshold
```

## Limits

| Limit | Value |
|-------|-------|
| Max value size | 64 KB |
| Max list size | 10,000 items |
| Max key length | 256 bytes |
| Max store name length | 64 bytes |

Limits are subject to change — check https://www.quicknode.com/docs/key-value-store for current values.

## Documentation

- **Key-Value Store Docs**: https://www.quicknode.com/docs/key-value-store
- **Quicknode CLI**: https://www.quicknode.com/docs/quicknode-cli
- **Streams Integration**: https://www.quicknode.com/docs/streams
