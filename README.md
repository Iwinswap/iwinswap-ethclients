# iwinswap-ethclients

*A Go library that provides foundational interfaces, mocks, and utility wrappers for Ethereum clients, designed to promote testable and decoupled application architecture.*

---

## Overview

When you build Go applications that interact with Ethereum, writing **modular, easily testable code** usually means depending on **interfaces** rather than concrete client implementations.

`iwinswap-ethclients` supplies those essentials:

| Component                                     | Purpose                                                                                                                                                                              |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **`ETHClient` interface**                     | A thin abstraction that mirrors the methods of `ethclient.Client` (and satisfies interfaces like `bind.ContractBackend`). Depend on this instead of the concrete go‑ethereum client. |
| **`TestETHClient` mock**                      | A powerful in‑memory mock so unit tests need **no live RPC endpoint**.                                                                                                               |
| **`ETHClientWithMaxConcurrentCalls` wrapper** | Drop‑in decorator that limits the number of in‑flight RPC calls — handy for rate‑limited endpoints.                                                                                  |

Use the package directly or as the bedrock for higher‑level helpers (e.g., a client manager).

---

## Installation

```bash
go get github.com/Iwinswap/iwinswap-ethclients
```

---

## Core Components

### 1. `ETHClient` Interface

The heart of the package. It standardises Ethereum‑node interactions:

```go
type ETHClient interface {
    BlockNumber(ctx context.Context) (uint64, error)
    // …other methods that match ethclient.Client and bind.ContractBackend
}
```

Write your business logic against `ETHClient`; swap in **real**, **mock**, or **wrapped** clients without touching your code.

---

### 2. `TestETHClient` — Built‑in Mock

Unit‑test without spinning up Ganache/Anvil/mainnet:

```go
mock := ethclients.NewTestETHClient()

expected := uint64(12345678)
mock.SetBlockNumberHandler(func(ctx context.Context) (uint64, error) {
    return expected, nil
})

svc := yourapp.NewBlockService(mock)

got, err := svc.GetLatestBlock(context.Background())
```

No RPC, no flakes — just pure Go tests.

---

### 3. `ETHClientWithMaxConcurrentCalls` — Concurrency Limiter

Protects rate‑limited endpoints or keeps CPU/RAM in check:

```go
raw, _ := ethclient.Dial("https://mainnet.infura.io/v3/YOUR_KEY")

limited, _ := ethclients.NewETHClientWithMaxConcurrentCalls(raw, 10) // max 10 concurrent calls

blockNum, _ := limited.BlockNumber(context.Background())
log.Printf("latest block: %d", blockNum)
```

Any goroutine over the 10th blocks until a slot is free.

---

## Example: Testing with `TestETHClient`

```go
package yourapp

import (
    "context"
    "testing"

    "github.com/Iwinswap/iwinswap-ethclients"
    "github.com/stretchr/testify/assert"
)

type BlockService struct{ c ethclients.ETHClient }

func NewBlockService(c ethclients.ETHClient) *BlockService { return &BlockService{c} }

func (s *BlockService) GetLatestBlock(ctx context.Context) (uint64, error) {
    return s.c.BlockNumber(ctx)
}

func TestBlockService(t *testing.T) {
    mock := ethclients.NewTestETHClient()

    want := uint64(12345678)
    mock.SetBlockNumberHandler(func(ctx context.Context) (uint64, error) { return want, nil })

    svc := NewBlockService(mock)

    got, err := svc.GetLatestBlock(context.Background())
    assert.NoError(t, err)
    assert.Equal(t, want, got)
}
```

---

## Contributing

Pull requests and issues are welcome!

```bash
go fmt ./...
go test ./...
```

---

## License

`iwinswap-ethclients` is released under the MIT License — see [`LICENSE`](./LICENSE) for full text.
