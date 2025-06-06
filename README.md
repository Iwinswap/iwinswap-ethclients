ETHClients Go Library

A Go library providing foundational interfaces, mocks, and utility wrappers for Ethereum clients, designed to promote testable and decoupled application architecture.
Overview

When building Go applications that interact with the Ethereum network, it's crucial to write code that is modular and easily testable. This often means depending on interfaces rather than concrete client implementations.

The ethclients package provides these essential, foundational components:

    An ETHClient interface that standardizes interactions with any Ethereum node.

    A powerful TestETHClient mock implementation for clean and simple unit testing.

    A utility wrapper, ETHClientWithMaxConcurrentCalls, to easily add concurrency limiting to any client.

This package serves as the bedrock for other, more complex packages (like a client manager) or for direct use in your application to decouple your business logic from the underlying go-ethereum client.
Installation

go get [github.com/Iwinswap/iwinswap-ethclients](https://github.com/Iwinswap/iwinswap-ethclients)

Core Components
The ETHClient Interface

This is the central abstraction of the package. It defines the set of methods for interacting with an Ethereum node, mirroring the capabilities of go-ethereum's ethclient.Client and satisfying common interfaces like bind.ContractBackend.

By having your application depend on ethclients.ETHClient, you can easily swap the real implementation with a mock for testing or use different client wrappers without changing your application logic.
TestETHClient: A Built-in Mock for Easy Testing

The primary feature of this package is the TestETHClient mock. It allows you to test your application code that relies on an ETHClient without needing a live network connection. You can set handlers to mock the behavior of any interface method.

Example: Testing an application service

Imagine you have a service that needs to check the latest block number.

// your-app/service.go
package yourapp

import (
	"context"
	"[github.com/Iwinswap/iwinswap-ethclients](https://github.com/Iwinswap/iwinswap-ethclients)"
)

type BlockService struct {
	client ethclients.ETHClient
}

func NewBlockService(client ethclients.ETHClient) *BlockService {
    return &BlockService{client: client}
}

func (s *BlockService) GetLatestBlock(ctx context.Context) (uint64, error) {
	return s.client.BlockNumber(ctx)
}

You can test BlockService easily and reliably using TestETHClient:

// your-app/service_test.go
package yourapp

import (
	"context"
	"testing"

	"[github.com/Iwinswap/iwinswap-ethclients](https://github.com/Iwinswap/iwinswap-ethclients)" // Import the root package
	"[github.com/stretchr/testify/assert](https://github.com/stretchr/testify/assert)"
)

func TestBlockService_GetLatestBlock(t *testing.T) {
	// 1. Create an instance of the mock client.
	// Assumes NewTestETHClient is exported from the root 'ethclients' package.
	mockClient := ethclients.NewTestETHClient() 

	// 2. Set a handler for the method you expect to be called.
	expectedBlockNumber := uint64(12345678)
	mockClient.SetBlockNumberHandler(func(ctx context.Context) (uint64, error) {
		// This function is called when the code under test calls BlockNumber().
		return expectedBlockNumber, nil
	})

	// 3. Inject the mock client into your service.
	service := NewBlockService(mockClient) // Assuming a constructor

	// 4. Run the function and assert the results.
	blockNum, err := service.GetLatestBlock(context.Background())

	assert.NoError(t, err)
	assert.Equal(t, expectedBlockNumber, blockNum)
}

ETHClientWithMaxConcurrentCalls: A Concurrency Limiter

This is a utility decorator that wraps any existing ETHClient to limit the number of simultaneous RPC calls that can be in-flight. This is useful for protecting rate-limited RPC endpoints or controlling your application's resource consumption.

Example: Wrapping a real client

package main

import (
	"context"
	"log"

	"[github.com/ethereum/go-ethereum/ethclient](https://github.com/ethereum/go-ethereum/ethclient)" // The real client
	"[github.com/Iwinswap/iwinswap-ethclients](https://github.com/Iwinswap/iwinswap-ethclients)"   // Your library
)

func main() {
	// Assume 'realClient' is an already connected *ethclient.Client
	realClient, err := ethclient.Dial("[https://mainnet.infura.io/v3/YOUR_INFURA_PROJECT_ID](https://mainnet.infura.io/v3/YOUR_INFURA_PROJECT_ID)")
	if err != nil {
		log.Fatalf("Failed to connect to the Ethereum network: %v", err)
	}

	// Wrap the real client to limit concurrent calls to 10
	// Assumes NewETHClientWithMaxConcurrentCalls is exported from the root 'ethclients' package
	limitedClient, err := ethclients.NewETHClientWithMaxConcurrentCalls(realClient, 10) 
	if err != nil {
		log.Fatalf("Failed to create limited client: %v", err)
	}

	// Now use limitedClient just like any other ETHClient.
	// Any calls beyond the 10th concurrent call will block until a slot is free.
	blockNum, err := limitedClient.BlockNumber(context.Background())
	if err != nil {
		log.Fatalf("Failed to get block number: %v", err)
	}

	log.Printf("Latest block number: %d", blockNum)
}

Contributing

Contributions are welcome! Please feel free to submit a pull request or open an issue for bugs and feature requests.

Before submitting, please ensure your code is formatted and tested:

go fmt ./...
go test ./...

License

This project is licensed under the MIT License. See the LICENSE file for details.