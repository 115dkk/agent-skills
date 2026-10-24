# When to Mock

Mock at **system boundaries** only:

- External APIs (payment, email, etc.)
- Databases (sometimes - prefer test DB)
- Time/randomness
- File system (sometimes)

Don't mock:

- Your own classes/modules
- Internal collaborators
- Anything you control

## Designing for Mockability

At system boundaries, design interfaces that are easy to mock:

**1. Use dependency injection**

Pass external dependencies in rather than creating them internally:

```typescript
// Easy to mock
function processPayment(order, paymentClient) {
  return paymentClient.charge(order.total);
}

// Hard to mock
function processPayment(order) {
  const client = new StripeClient(process.env.STRIPE_KEY);
  return client.charge(order.total);
}
```

**2. Prefer SDK-style interfaces over generic fetchers**

Create specific functions for each external operation instead of one generic function with conditional logic:

```typescript
// GOOD: Each function is independently mockable
const api = {
  getUser: (id) => fetch(`/users/${id}`),
  getOrders: (userId) => fetch(`/users/${userId}/orders`),
  createOrder: (data) => fetch('/orders', { method: 'POST', body: data }),
};

// BAD: Mocking requires conditional logic inside the mock
const api = {
  fetch: (endpoint, options) => fetch(endpoint, options),
};
```

The SDK approach means:
- Each mock returns one specific shape
- No conditional logic in test setup
- Easier to see which endpoints a test exercises
- Type safety per endpoint

## Adapters that misbehave on purpose

At a seam onto code you don't control, a test adapter that only follows the contract proves little: the bugs come from counterparts that break it. Build the test adapter so it can reproduce each misbehaviour that real counterparts have been seen to commit, one named mode per misbehaviour, and record where each was found.

When the counterpart is loadable code (a plugin, a driver, a shared library), the adapter is a real module the production loader opens, not an in-process mock of the interface, so the test runs the same ABI path as production. Selecting the mode by file name keeps the loader untouched:

```text
TestPlugin modes, chosen by the module's file name
  TestPlugin.vst3            follows the contract
  IgnoreParameterFlush.vst3  drops parameter changes passed in a zero-sample
                             process call, as every DPF-built plugin does
                             (EqualizerAPO-XT#437)
  RetainHandler.vst3         keeps the host's component handler and calls it
                             after the host released the instance
                             (audit #348 TD-48)
```

Each mode is the reproduction for one defence in production code. The defence's comment names the violator and this mode, so the next reader, and the next audit, can see the defence is earned.
