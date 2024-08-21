> **ATTENTION** : This repository is a fork of github.com/AugustDev/langgraphgo which is a fork of github.com/tmc/langgraphgo. Nothing is added to this fork and i'm not the author. This fork only exist to allow us to use the AddConditionalEdge function added by AugustDev as a PR (not merged) on github.com/tmc/langgraphgo (https://github.com/tmc/langgraphgo/pull/2) with a different package name to avoid any conflict in the management of go dependencies.

> **TIME TO LIVE** : As soon as the AugustDev's PR is merged into github.com/tmc/langgraphgo (https://github.com/tmc/langgraphgo/pull/2), this fork will be removed. This fork is not for long-term use !

# 🦜️🔗 LangGraphGo

[![go.dev reference](https://img.shields.io/badge/go.dev-reference-007d9c?logo=go&logoColor=white&style=flat-square)](https://pkg.go.dev/github.com/tmc/langgraphgo)


## Quick Start


This is a simple example of how to use the library to create a simple chatbot that uses OpenAI to generate responses.

```go
import (
	"context"
	"errors"
	"fmt"
	"testing"

	"github.com/tmc/langchaingo/llms"
	"github.com/tmc/langchaingo/llms/openai"
	"github.com/tmc/langchaingo/schema"
	"github.com/tmc/langgraphgo/graph"
)

func main() {
	model, err := openai.New()
	if err != nil {
		panic(err)
	}

	g := graph.NewMessageGraph()

	g.AddNode("oracle", func(ctx context.Context, state []llms.MessageContent) ([]llms.MessageContent, error) {
		r, err := model.GenerateContent(ctx, state, llms.WithTemperature(0.0))
		if err != nil {
			return nil, err
		}
		return append(state,
			llms.TextParts(schema.ChatMessageTypeAI, r.Choices[0].Content),
		), nil

	})
	g.AddNode(graph.END, func(ctx context.Context, state []llms.MessageContent) ([]llms.MessageContent, error) {
		return state, nil
	})

	g.AddEdge("oracle", graph.END)
	g.SetEntryPoint("oracle")

	runnable, err := g.Compile()
	if err != nil {
		panic(err)
	}

	ctx := context.Background()
	// Let's run it!
	res, err := runnable.Invoke(ctx, []llms.MessageContent{
		llms.TextParts(schema.ChatMessageTypeHuman, "What is 1 + 1?"),
	})
	if err != nil {
		panic(err)
	}

	fmt.Println(res)

	// Output:
	// [{human [{What is 1 + 1?}]} {ai [{1 + 1 equals 2.}]}]
}
```
