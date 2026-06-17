---
name: test-driven-development
description: Use when implementing any feature or bugfix, before writing implementation code
---

# Use-Case-Driven Development: Integration Tests First

## Overview

Define the use case flow first.
Then define the core data structures and interface contracts.
Then write an integration test that proves the flow works end to end.
Finally, implement the minimum amount of code required to make this use case pass.

**Core principle:** You do not need to write fine-grained unit tests for every internal function, but you must control the core data structures, interface boundaries, and complete flow of the use case.

The goal of testing is not to prove that “a certain function is implemented correctly,” but to prove that:

> The input, state changes, interface calls, data flow, and output of a real use case are coherent.

## When to Use

Use this for:

- New features
- Bug fixes
- Core flow refactoring
- Interface definition changes
- Data structure changes

Fine-grained unit tests may still be kept when:

- A bug is hard to reproduce reliably through integration tests
- A module is core infrastructure reused by many flows

## Iron Rule

Do not start implementation before defining the use case flow and interface contracts.

Before implementation, you must clearly define:

- What input is received after the trigger?
- Which functions or interfaces does the input pass through?
- What structures is it parsed into at each step of the flow?
- How is the parsed result eventually consumed?
- What function-level or interface-level capabilities need to be implemented or reused?
- What are the core data structures?
- What is the responsibility of each interface?

## Development Loop

**PLAN → CONTRACT → INTEGRATION TEST → IMPLEMENT → VERIFY → REFACTOR**

------

## 1. PLAN — Define the Use Case

First, write a minimal but complete use case.

A good use case should include:

- User or system trigger
- Initial input
- Key state
- Expected output
- Important side effects
- Failure scenarios or boundary cases

Example:

```markdown
## Use Case: Create an agent session after the user submits a prompt

### Trigger

- The user presses Enter to submit a prompt

### Input

- The prompt entered by the user
- The list of available tools

### Flow

1. The UI submits the prompt
2. A session is created
3. A UserMessage is saved
4. AgentRuntime is called
5. AgentRunner decides whether to call tools based on the tool schema
6. An AssistantMessage is written
7. The UI receives the session update

### Expected Result

- A session is created
- The user message is persisted
- AgentRuntime is called
- An assistant message is appended
- The UI can render the complete conversation based on the session state
```

------

## 2. CONTRACT — Define Core Data Structures and Interfaces

Before implementation, define the boundaries instead of jumping directly into internal logic.

You need to clarify:

- Core domain models
- DTOs / state shapes
- Interface inputs and outputs
- State transitions
- Dependencies between modules

Do not rush into implementation. First, make the interfaces clearly express how the flow is connected. Must clearly provide a detailed definition of the capability will be implemented

------

## 3. INTEGRATION TEST — Write Use-Case-Level Tests

The test should cover a complete flow, not an internal function.

No compilation is required; it only provides semantic guidance. 

A good test focuses on:

- The real entry point
- Real core modules
- As few mocks as possible
- Mocking external uncontrollable systems, not your own business logic
- Verifying final state and key side effects

```ts
test('creates a session and generates an assistant message after the user submits a prompt', async () => {
  const app = createTestApp({
    agentRunner: fakeAgentRunner({
      response: 'Hello, I can help you.'
    })
  });

  const result = await app.submitPrompt({
    workspaceId: 'workspace-1',
    prompt: 'Hello'
  });

  const session = await app.sessions.getById(result.sessionId);

  expect(session.workspaceId).toBe('workspace-1');
  expect(session.status).toBe('completed');
  expect(session.messages).toEqual([
    { role: 'user', content: 'Hello' },
    { role: 'assistant', content: 'Hello, I can help you.' }
  ]);
});
```

This test verifies the complete use case: entry point, session creation, runner invocation, message writing, and final state.

```ts
test('appendMessage works', async () => {
  const repo = mockRepo();
  await appendMessage(repo, 'session-1', { role: 'user', content: 'hi' });
  expect(repo.appendMessage).toHaveBeenCalled();
});
```

This test only verifies that a mock was called. It does not prove that the flow actually works.

------

## 4. IMPLEMENT — Write the Minimum Code

Only implement the code required to make the current use case integration test pass.

Do not implement ahead of time:

- Parameters that may be needed in the future
- Complex abstractions
- Extra configuration options
- Branches not covered by the current use case
- Premature framework-style abstractions

It is acceptable to start with a simple, direct, replaceable implementation.

------

## 5. VERIFY — Validate That the Flow Passes

Run the integration test for the corresponding use case.

Confirm that:

- The integration test passes
- Core data structures are not temporarily hacked
- Interface boundaries do not leak implementation details
- Failures can be located at a specific stage in the flow
- The test does not over-mock your own business code

If the test fails, fix the implementation first.
If the test is hard to write, first check whether the interface design or data structures are unclear.

------

## 6. REFACTOR — Refactor After Green

Refactoring is only allowed after the integration test passes.

Allowed:

- Extract helpers
- Improve naming
- Split modules
- Simplify data structures
- Adjust interface boundaries
- Remove duplicate logic

Not allowed:

- Adding new behavior without adding a new use case test
- Breaking interface contracts for implementation convenience
- Turning an integration test into a test that only verifies mock calls

## Test Granularity Principle

The default testing granularity is:

> One integration test group per use case

Instead of:

> One test per function

## Mocking Principle

Only mock system boundaries. Do not mock the core business flow.

## Final Rules

The core data structures are clear.
The interface contracts are clear.
The use case flow is clear.
The integration test proves that the flow works.

Otherwise, it is not done.
