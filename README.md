# Tidelight React

Typed React hooks and accessible, themeable components for Tidelight market and research workflows. The package wraps the official SDK client interface; it does not duplicate API transport, authorization, billing, or research logic.

[Product](https://tidelight.app) · [Developer API guide](https://dev.tidelight.app/api) · [npm package](https://www.npmjs.com/package/@tidelight/react)

## Repository and source of truth

This repository is a public mirror of the React package in the [canonical Tidelight monorepo](https://github.com/TidelightOrg/tidelight/tree/main/packages/react). The monorepo is the sole source of truth for implementation code. Make code changes there; this mirror is refreshed from the monorepo's `main` branch each day and can also be synchronized manually from its Actions page. The mirror workflow preserves this README and its own workflow file.

## Install

```sh
npm install @tidelight/react@0.1.2 @tidelight/sdk@0.1.2 react react-dom
```

React 18 and 19 are supported. Import the optional default styles when desired:

```tsx
import { TidelightProvider, TokenizedEquityGrid, useTokenizedEquities } from "@tidelight/react";
import "@tidelight/react/styles.css";
```

## Example

```tsx
function MarketPanel() {
  const result = useTokenizedEquities({ underlyingTicker: "NVDA", limit: 20 });

  return (
    <TokenizedEquityGrid
      instruments={result.data?.data ?? []}
      isLoading={result.isLoading}
      error={result.error?.message}
      onRetry={result.refresh}
    />
  );
}

// This client calls your authenticated, same-origin application server route.
// Keep the Tidelight project key on that server, never in a browser bundle.
export function App({ applicationServerClient }) {
  return (
    <TidelightProvider client={applicationServerClient}>
      <MarketPanel />
    </TidelightProvider>
  );
}
```

The provider accepts a structurally compatible client. Put the official SDK and project credential behind an application server route or server action, then return only authorized, structured responses to the signed-in user.

## Included APIs

- `useTidelightClient()` returns the configured client and gives a clear error when used outside the provider.
- `useTokenizedEquities(query)` loads supported instruments, aborts stale requests, and exposes data, loading, error, and refresh state.
- `useResearchJob(jobId, { pollIntervalMs })` reads a research job. Polling is opt-in and stops when the job reaches a terminal state.
- `useStartCompanyResearch()` exposes `start(ticker, idempotencyKey?)` and `cancel()`, with request, error, and loading state.
- `TokenizedEquityGrid` renders loading, error/retry, empty, and result states. Its optional CSS uses themeable `--tidelight-*` custom properties.

## Security and data boundaries

Never bundle a Tidelight project key in browser code. The React package does not authenticate application users or store credentials. Apply your own user authentication, tenant isolation, CSRF protections, and request limits at the server boundary.

Instrument data reflects the service response and its availability. The UI preserves the distinction between tokenized-equity spot products and direct stock shares or perpetual contracts. Retry behavior belongs to the supplied server-side SDK client.

## Compatibility

| Surface | Support |
| --- | --- |
| React peer versions | React 18 and 19 |
| Runtime | Browser or React server/client application with an injected compatible transport |
| API operations | SDK instrument listing and company research jobs |
| Styling | Optional stylesheet and CSS custom properties |
| Peer dependencies | `@tidelight/sdk`, React, and React DOM |

## Development and validation

Run package commands from the canonical monorepo so the React package resolves its matching local SDK dependency:

```sh
npm ci --prefix sdk/typescript
npm test --prefix sdk/typescript
npm ci --prefix packages/react
npm test --prefix packages/react
```

See [CHANGELOG.md](./CHANGELOG.md) for release history. This project is licensed under Apache-2.0; see [LICENSE](./LICENSE).
