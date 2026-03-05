# Quicknode Admin API Reference

REST API for programmatic management of Quicknode endpoints, usage monitoring, rate limits, security, billing, and teams.

## Quick Reference

| Resource | Methods | Endpoints |
|----------|---------|-----------|
| **Chains** | GET | `/v0/chains` |
| **Endpoints** | GET, POST, PATCH, DELETE | `/v0/endpoints`, `/v0/endpoints/{id}` |
| **Endpoint Status** | PATCH | `/v0/endpoints/{id}/status` |
| **Endpoint Tags** | GET, POST | `/v0/endpoints/{id}/tags` |
| **Metrics** | GET | `/v0/endpoints/{id}/metrics` |
| **Rate Limits** | GET, POST, PUT | `/v0/endpoints/{id}/method-rate-limits`, `/v0/endpoints/{id}/rate-limits` |
| **Security** | GET | `/v0/endpoints/{id}/security_options` |
| **Usage** | GET | `/v0/usage/rpc`, `/v0/usage/rpc/by-endpoint`, `/v0/usage/rpc/by-method`, `/v0/usage/rpc/by-chain` |
| **Billing** | GET | `/v0/billing/invoices` |
| **Teams** | GET | `/v0/teams` |
| **Logs** | GET | `/v0/endpoints/{id}/logs` (Enterprise) |
| **Prometheus** | GET | `/exporter/prometheus` (Enterprise) |

## Authentication

Base URL: `https://api.quicknode.com/v0`

All requests require the `x-api-key` header. Generate an API key from the Quicknode dashboard under **Settings > API Keys**.

```typescript
const QN_API_KEY = process.env.QUICKNODE_API_KEY!;
const BASE_URL = 'https://api.quicknode.com/v0';

async function adminApi<T>(
  path: string,
  options: RequestInit = {}
): Promise<T> {
  const res = await fetch(`${BASE_URL}${path}`, {
    ...options,
    headers: {
      'x-api-key': QN_API_KEY,
      'Content-Type': 'application/json',
      ...options.headers,
    },
  });
  if (!res.ok) {
    const body = await res.text();
    throw new Error(`Admin API ${res.status}: ${body}`);
  }
  return res.json();
}
```

All examples below reuse this `adminApi` helper.

## Chains

List all supported blockchain networks.

```typescript
const chains = await adminApi<{ data: Array<{
  id: string;
  name: string;
  network: string;
  chain: string;
}> }>('/chains');

chains.data.forEach(c => console.log(`${c.name} (${c.network})`));
```

## Endpoints

### List Endpoints

```typescript
const endpoints = await adminApi<{ data: Array<{
  id: string;
  name: string;
  chain: string;
  network: string;
  http_url: string;
  wss_url: string;
  status: string;
}> }>('/endpoints');

endpoints.data.forEach(ep =>
  console.log(`${ep.name}: ${ep.chain}/${ep.network} [${ep.status}]`)
);
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | number | No | Page number for pagination |
| `per_page` | number | No | Results per page (default: 25) |
| `tag` | string | No | Filter by tag name |

```typescript
// Paginate with tag filter
const filtered = await adminApi('/endpoints?tag=production&page=1&per_page=10');
```

### Create Endpoint

```typescript
const newEndpoint = await adminApi('/endpoints', {
  method: 'POST',
  body: JSON.stringify({
    name: 'my-eth-mainnet',
    chain: 'ethereum',
    network: 'mainnet',
  }),
});

console.log('Created:', newEndpoint.http_url);
```

**Body Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Display name for the endpoint |
| `chain` | string | Yes | Blockchain (e.g., `ethereum`, `solana`) |
| `network` | string | Yes | Network (e.g., `mainnet`, `sepolia`, `devnet`) |

### Get Endpoint

```typescript
const endpoint = await adminApi(`/endpoints/${endpointId}`);
console.log(endpoint.http_url, endpoint.status);
```

### Update Endpoint

```typescript
const updated = await adminApi(`/endpoints/${endpointId}`, {
  method: 'PATCH',
  body: JSON.stringify({ name: 'renamed-endpoint' }),
});
```

### Pause / Unpause Endpoint

```typescript
// Pause
await adminApi(`/endpoints/${endpointId}/status`, {
  method: 'PATCH',
  body: JSON.stringify({ status: 'paused' }),
});

// Unpause
await adminApi(`/endpoints/${endpointId}/status`, {
  method: 'PATCH',
  body: JSON.stringify({ status: 'active' }),
});
```

### Archive (Delete) Endpoint

```typescript
await adminApi(`/endpoints/${endpointId}`, { method: 'DELETE' });
```

### Tags

```typescript
// List tags on an endpoint
const tags = await adminApi(`/endpoints/${endpointId}/tags`);

// Add tags
await adminApi(`/endpoints/${endpointId}/tags`, {
  method: 'POST',
  body: JSON.stringify({ tags: ['production', 'critical'] }),
});
```

## Endpoint Metrics

Retrieve performance metrics for an endpoint.

```typescript
const metrics = await adminApi<{ data: {
  total_requests: number;
  success_rate: number;
  avg_latency_ms: number;
  p99_latency_ms: number;
} }>(`/endpoints/${endpointId}/metrics?period=24h&metric=requests`);

console.log('Requests (24h):', metrics.data.total_requests);
console.log('Success rate:', metrics.data.success_rate);
console.log('Avg latency:', metrics.data.avg_latency_ms, 'ms');
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `period` | string | No | Time period (`1h`, `24h`, `7d`, `30d`) |
| `metric` | string | No | Metric type (`requests`, `latency`, `errors`) |

## Rate Limits

### Get Method Rate Limits

```typescript
const methodLimits = await adminApi(
  `/endpoints/${endpointId}/method-rate-limits`
);
console.log(methodLimits);
```

### Set Method Rate Limits

Restrict specific RPC methods on an endpoint.

```typescript
await adminApi(`/endpoints/${endpointId}/method-rate-limits`, {
  method: 'POST',
  body: JSON.stringify({
    method_rate_limits: [
      { method: 'eth_getLogs', rate_limit: 50 },
      { method: 'debug_traceTransaction', rate_limit: 10 },
    ],
  }),
});
```

**Body Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `method_rate_limits` | array | Yes | Array of `{ method, rate_limit }` objects |
| `method_rate_limits[].method` | string | Yes | RPC method name |
| `method_rate_limits[].rate_limit` | number | Yes | Max requests per second for this method |

### Set Global Rate Limits

```typescript
await adminApi(`/endpoints/${endpointId}/rate-limits`, {
  method: 'PUT',
  body: JSON.stringify({
    rps: 100,   // requests per second
    rpm: 5000,  // requests per minute
    rpd: 500000 // requests per day
  }),
});
```

**Body Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `rps` | number | No | Requests per second |
| `rpm` | number | No | Requests per minute |
| `rpd` | number | No | Requests per day |

## Security Options

Retrieve security configuration for an endpoint.

```typescript
const security = await adminApi(
  `/endpoints/${endpointId}/security_options`
);
console.log(security);
```

**Security Option Types:**

| Option | Description |
|--------|-------------|
| `jwt_authentication` | JSON Web Token auth for endpoint access |
| `ip_allowlist` | Restrict access to specific IP addresses |
| `referrer_allowlist` | Restrict access to specific HTTP referrers |
| `token_authentication` | Additional token-based authentication |

## Usage

### Total RPC Usage

```typescript
const usage = await adminApi<{ data: {
  total_requests: number;
  total_credits: number;
} }>('/usage/rpc?start_time=2025-01-01T00:00:00Z&end_time=2025-01-31T23:59:59Z');

console.log('Total requests:', usage.data.total_requests);
console.log('Credits used:', usage.data.total_credits);
```

**Query Parameters (shared across all usage endpoints):**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `start_time` | string | Yes | ISO 8601 start time |
| `end_time` | string | Yes | ISO 8601 end time |

### Usage by Endpoint

```typescript
const byEndpoint = await adminApi(
  '/usage/rpc/by-endpoint?start_time=2025-01-01T00:00:00Z&end_time=2025-01-31T23:59:59Z'
);
```

### Usage by Method

```typescript
const byMethod = await adminApi(
  '/usage/rpc/by-method?start_time=2025-01-01T00:00:00Z&end_time=2025-01-31T23:59:59Z'
);
```

### Usage by Chain

```typescript
const byChain = await adminApi(
  '/usage/rpc/by-chain?start_time=2025-01-01T00:00:00Z&end_time=2025-01-31T23:59:59Z'
);
```

## Billing

Retrieve invoice history.

```typescript
const invoices = await adminApi<{ data: Array<{
  id: string;
  amount: number;
  currency: string;
  status: string;
  period_start: string;
  period_end: string;
}> }>('/billing/invoices');

invoices.data.forEach(inv =>
  console.log(`${inv.period_start} - ${inv.period_end}: $${inv.amount / 100} [${inv.status}]`)
);
```

## Teams

List team members and roles.

```typescript
const teams = await adminApi<{ data: Array<{
  id: string;
  name: string;
  members: Array<{
    email: string;
    role: string;
    status: string;
  }>;
}> }>('/teams');

teams.data.forEach(team => {
  console.log(`Team: ${team.name}`);
  team.members.forEach(m => console.log(`  ${m.email} (${m.role})`));
});
```

## Logs (Enterprise)

Query request logs for an endpoint. Available on Enterprise plans.

```typescript
const logs = await adminApi<{ data: Array<{
  timestamp: string;
  method: string;
  status_code: number;
  latency_ms: number;
  ip_address: string;
}> }>(`/endpoints/${endpointId}/logs?from=2025-01-01T00:00:00Z&to=2025-01-02T00:00:00Z&limit=100`);

logs.data.forEach(log =>
  console.log(`${log.timestamp} ${log.method} ${log.status_code} ${log.latency_ms}ms`)
);
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `from` | string | Yes | ISO 8601 start time |
| `to` | string | Yes | ISO 8601 end time |
| `limit` | number | No | Max results (default: 100, max: 1000) |
| `next_at` | string | No | Cursor for pagination |

## Prometheus (Enterprise)

Export endpoint metrics in Prometheus format for integration with Grafana or other monitoring tools.

```
GET https://api.quicknode.com/exporter/prometheus
Header: x-api-key: YOUR_API_KEY
```

Returns metrics in Prometheus exposition format, suitable for scraping.

## Best Practices

1. **API key storage** — Store keys in environment variables or a secrets manager. Never hardcode in source.
2. **Pagination** — Use `page` and `per_page` on list endpoints to avoid large responses.
3. **Retry logic** — Implement exponential backoff for transient `5xx` errors.
4. **Caching** — Cache chain and endpoint list responses (they change infrequently).
5. **Tagging** — Tag endpoints by environment (`production`, `staging`, `dev`) for easy filtering.
6. **Usage monitoring** — Poll usage endpoints periodically and alert when approaching plan limits.
7. **Rate limit headroom** — Set method-level limits below your plan ceiling to prevent runaway consumers.

## Documentation

- **Admin API Docs**: https://www.quicknode.com/docs/admin-api
- **Admin API Docs (llms.txt)**: https://www.quicknode.com/docs/admin-api/llms.txt
- **Quicknode Dashboard**: https://dashboard.quicknode.com/
- **API Key Management**: https://dashboard.quicknode.com/settings/api-keys
- **Guides**: https://www.quicknode.com/guides/tags/admin-api
