---
title: $agentCall
---

# `$agentCall`

Use `$agentCall` to send the current document (or a projection) to a registered AI/decision agent and merge the structured response back into the pipeline.

```json
{
  "$agentCall": {
    "agent": "order-approval@1.2.0",
    "input": {
      "orderId": "$orderId",
      "amount": "$amount",
      "customerTier": "$customer.tier",
      "items": "$items"
    },
    "prompt": "Evaluate whether this order should be auto-approved or escalated.",
    "responseField": "agentDecision",
    "metadata": {
      "policyGroup": "north-america"
    },
    "onError": "skip"
  }
}
```

## When to use it

- You need an AI agent (LLM policy, custom decision service, etc.) to score a document and return structured data (decision, explanation, confidence).
- You want stage-level error policies (`inherit`, `bubble`, `skip`) rather than allowing a single expression to break the entire projection.
- You need to attach prompts, metadata, or versioned agent references.

## Parameters

| Field | Type | Description |
| --- | --- | --- |
| `agent` | string | Required. Agent name with optional version (`name@version`). Example: `order-approval@1.2.0`. |
| `input` | object / expression | Projection evaluated per document. Defaults to `$$CURRENT`. All values are resolved via the expression evaluator, so you can reference `$field`, `$$ROOT`, etc. |
| `prompt` | string | Optional natural-language guidance sent to the agent. Useful to capture policy instructions. |
| `metadata` | object | Optional static metadata forwarded to the agent (e.g., `policyGroup`, `requestId`). |
| `responseField` | string | Field where the agent response is stored. Default `agentDecision`. Use blank to merge the response at the root level. |
| `onError` | string | Error policy: `inherit` (default) to propagate, `bubble` to wrap the error with more context, `skip` to keep the original document when the agent fails. |

## Agent response shape

Agents return a JSON object. For example:

```json
{
  "decision": "ESCALATE",
  "confidence": 0.78,
  "notes": "High-value order requires manual review",
  "explanation": "Customer tier is bronze and amount exceeds limit"
}
```

If `responseField` is set to `agentDecision`, the document will now include:

```json
{
  "orderId": "A-104",
  "amount": 2150.75,
  "agentDecision": {
    "decision": "ESCALATE",
    "confidence": 0.78,
    "notes": "High-value order requires manual review"
  }
}
```

## Registering agents

Implement `ai.fluxion.core.agent.AgentExecutor` and provide it via `META-INF/services/ai.fluxion.core.agent.AgentContributor`. Executors receive an `AgentCall` (resolved input, prompt, metadata) and return an `AgentResponse` (map of values). Register multiple versions by overriding `version()`.

Example executor (simplified):

```java
public final class OrderApprovalAgent implements AgentExecutor {
    @Override
    public String name() {
        return "order-approval";
    }

    @Override
    public String version() {
        return "1.2.0";
    }

    @Override
    public AgentResponse execute(AgentCall call) {
        Map<String, Object> payload = call.payload();
        double amount = ((Number) payload.getOrDefault("amount", 0)).doubleValue();
        String decision = amount < 1000 ? "APPROVE" : "ESCALATE";
        return new AgentResponse(Map.of(
            "decision", decision,
            "confidence", amount < 1000 ? 0.92 : 0.64,
            "notes", call.prompt()
        ));
    }
}
```

## Example pipeline segment

```json
[
  { "$addFields": { "total": { "$sum": "$lineItems.amount" } } },
  { "$agentCall": {
      "agent": "order-approval@1.2.0",
      "input": {
        "orderId": "$orderId",
        "amount": "$total",
        "customerTier": "$customer.tier",
        "lineItems": "$lineItems"
      },
      "prompt": "Decide whether the order should auto-approve.",
      "metadata": { "source": "checkout-stream" }
  }},
  { "$match": { "agentDecision.decision": "ESCALATE" } }
]
```

This pipeline:
1. Adds a computed `total`.
2. Calls the agent using `$agentCall` and stores the response under `agentDecision`.
3. Filters for escalated orders.
