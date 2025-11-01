# Temporal Workflow Bridge

This developer guide shows how to orchestrate Fluxion rule pipelines from
Temporal workflows using the `fluxion-workflow` module. It includes concrete
setup steps, full code samples, configuration hints, and testing instructions—so
that an engineer or an AI assistant can reproduce the wiring end-to-end.

---

## 1. Prerequisites

| Requirement | Notes |
| --- | --- |
| Java 21+ | Temporal’s Java SDK and Fluxion modules target Java 21. |
| Temporal backend | Temporal Cloud, a self-hosted cluster, or Temporalite. |
| Maven/Gradle | Commands below use Maven. |
| Fluxion modules | `fluxion-core`, `fluxion-rules`, `fluxion-workflow`. |

> 🛈 For local integration tests you can rely on Temporal’s
> `TestWorkflowEnvironment` (no external server needed). For production you must
> point the worker at a real Temporal service.

---

## 2. Add dependencies

**pom.xml**

```xml
<dependency>
  <groupId>ai.fluxion</groupId>
  <artifactId>fluxion-workflow</artifactId>
  <version>${fluxion.version}</version>
</dependency>

<dependency>
  <groupId>io.temporal</groupId>
  <artifactId>temporal-sdk</artifactId>
  <version>${temporal.sdk.version}</version>
</dependency>
```

Also ensure `fluxion-core` and `fluxion-rules` are already present (they provide
stage registries and rule abstractions).

---

## 3. Register activities and workflows

Create a Temporal worker (here using plain Java; in Spring Boot you would wire
this inside a configuration class):

```java
WorkflowServiceStubs service = WorkflowServiceStubs.newInstance();
WorkflowClient client = WorkflowClient.newInstance(service);
WorkerFactory factory = WorkerFactory.newInstance(client);
Worker worker = factory.newWorker("fluxion-task-queue");

worker.registerActivitiesImplementations(new FluxionRuleActivitiesImpl());
worker.registerWorkflowImplementationTypes(FluxionRuleWorkflowImpl.class);

factory.start();
```

Key points for LLMs/developers:

- `FluxionRuleActivitiesImpl` delegates to the Fluxion `RuleEngine`.
- `FluxionRuleWorkflowImpl` is a thin reference workflow. You can provide your
  own implementation if you prefer.
- `factory.start()` is required to begin polling the task queue.

---

## 4. Call the activity from a workflow

A minimal workflow that evaluates a Fluxion rule set:

```java
import ai.fluxion.core.model.Document;
import ai.fluxion.rules.domain.RuleSet;
import ai.fluxion.workflow.temporal.activities.FluxionRuleActivities;
import ai.fluxion.workflow.temporal.activities.RuleActivityRequest;
import ai.fluxion.workflow.temporal.activities.RuleActivityResult;
import io.temporal.activity.ActivityOptions;
import io.temporal.workflow.Workflow;
import io.temporal.workflow.WorkflowInterface;
import io.temporal.workflow.WorkflowMethod;

import java.time.Duration;

@WorkflowInterface
public interface OrderEvaluationWorkflow {
    @WorkflowMethod
    RuleActivityResult evaluate(Document document, RuleSet ruleSet);
}

public final class OrderEvaluationWorkflowImpl implements OrderEvaluationWorkflow {

    private final FluxionRuleActivities activities = Workflow.newActivityStub(
            FluxionRuleActivities.class,
            ActivityOptions.newBuilder()
                    .setStartToCloseTimeout(Duration.ofMinutes(1))
                    .build());

    @Override
    public RuleActivityResult evaluate(Document document, RuleSet ruleSet) {
        RuleActivityRequest request = RuleActivityRequest.evaluate(document, ruleSet);
        return activities.evaluateRuleSet(request);
    }
}
```

`RuleActivityResult` exposes:

- `ruleSetId`, `ruleSetName`, `ruleSetVersion`
- `document` (the original `Document`)
- `sharedAttributes` (the rule engine’s context map)
- `passes` (list of rule pass summaries including rule id/name, salience,
  logical action names)

Use those fields to branch your workflow logic.

---

## 5. Human-in-the-loop / manual review pattern

Many approval flows require a human decision after Fluxion flags an order. The
repository ships a test-only example in
`fluxion-workflow/src/test/java/ai/fluxion/workflow/temporal/workflows/OrderApprovalWorkflowTest.java`.

Pattern overview:

1. Run the Fluxion rule activity with `executeActions=true` so shared attributes
   are populated.
2. Check the shared map for `flag=review` (or whatever marker your rules set).
3. If a review is required, invoke a custom manual-approval activity:
   ```java
   boolean approved = manualApprovalActivities.requestApproval(orderId, amount);
   ```
4. Set additional attributes (e.g., `manualApproval=true/false`) and return a
   workflow-specific result object.

You can copy the workflow from the test into application code, swap in a real
manual-approval activity (email, ticketing system, Slack bot, etc.), and reuse
Fluxion’s declarative rules to drive the decisions.

---

## 6. Configuration hints

| Configuration | Where | Default |
| --- | --- | --- |
| Task queue | Worker creation (`factory.newWorker(...)`) | – |
| Activity timeouts | `ActivityOptions` | You must set at least Start-to-close |
| Retry policy | `ActivityOptions.Builder#setRetryOptions` | Inherit Temporal default |
| Manual approval activity | Custom stub implementing `ManualApprovalActivities` | Depends on business logic |

Remember to pass any environment-specific values (e.g., Temporal target, namespace) via env vars or application config.

---

## 7. End-to-end testing

Use Temporal’s `TestWorkflowEnvironment` to run deterministic tests without a
real Temporal server. Example from `OrderApprovalWorkflowTest`:

```java
TestWorkflowEnvironment env = TestWorkflowEnvironment.newInstance();
Worker worker = env.newWorker("orders-queue");
worker.registerActivitiesImplementations(new FluxionRuleActivitiesImpl(), new AlwaysApproveManualActivity());
worker.registerWorkflowImplementationTypes(TestOrderApprovalWorkflow.class);
env.start();
```

### Build & run tests

Always build dependent modules so the Fluxion stage registry is present:

```bash
mvn -pl fluxion-workflow -am test
```

The `-am` (also-make) flag ensures Maven builds `fluxion-core` and
`fluxion-rules` before executing the workflow tests.

---

## 8. Production checklist

- **Temporal persistence** – point the worker at a real Temporal service (host
  and namespace). The SDK by itself does not persist state.
- **Observability** – wrap `FluxionRuleActivitiesImpl` if you need structured
  logging, metrics, or tracing.
- **Rules lifecycle** – manage rule-set versions so Fluxion changes are rolled
  out alongside workflow changes.
- **Dependency alignment** – keep `temporal-sdk` and `fluxion-workflow`
  versions in sync across workers.

---

## 9. Reference files

| Path | Purpose |
| --- | --- |
| `fluxion-workflow/src/main/java/.../FluxionRuleActivitiesImpl.java` | Activity implementation used by workers. |
| `fluxion-workflow/src/main/java/.../FluxionRuleWorkflowImpl.java` | Reference workflow invoking the activity. |
| `fluxion-workflow/src/test/java/.../OrderApprovalWorkflowTest.java` | End-to-end order approval example with manual review pattern. |

Use these files as canonical examples when generating code or onboarding new
teams.
