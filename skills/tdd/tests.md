# Good and Bad Tests

## Good Tests

**Integration-style**: Test through real interfaces, not mocks of internal parts.

```typescript
// GOOD: Tests observable behavior
test("user can checkout with valid cart", async () => {
  const cart = createCart();
  cart.add(product);
  const result = await checkout(cart, paymentMethod);
  expect(result.status).toBe("confirmed");
});
```

Characteristics:

- Tests behavior users/callers care about
- Uses public API only
- Survives internal refactors
- Describes WHAT, not HOW
- One logical assertion per test

## Bad Tests

**Implementation-detail tests**: Coupled to internal structure.

```typescript
// BAD: Tests implementation details
test("checkout calls paymentService.process", async () => {
  const mockPayment = jest.mock(paymentService);
  await checkout(cart, payment);
  expect(mockPayment.process).toHaveBeenCalledWith(cart.total);
});
```

Red flags:

- Mocking internal collaborators
- Testing private methods
- Asserting on call counts/order
- Test breaks when refactoring without behavior change
- Test name describes HOW not WHAT
- Verifying through external means instead of interface

```typescript
// BAD: Bypasses interface to verify
test("createUser saves to database", async () => {
  await createUser({ name: "Alice" });
  const row = await db.query("SELECT * FROM users WHERE name = ?", ["Alice"]);
  expect(row).toBeDefined();
});

// GOOD: Verifies through interface
test("createUser makes user retrievable", async () => {
  const user = await createUser({ name: "Alice" });
  const retrieved = await getUser(user.id);
  expect(retrieved.name).toBe("Alice");
});
```

**Tautological tests**: Expected value restates the implementation, so the test passes by construction.

```typescript
// BAD: Expected value is recomputed the way the code computes it
test("calculateTotal sums line items", () => {
  const items = [{ price: 10 }, { price: 5 }];
  const expected = items.reduce((sum, i) => sum + i.price, 0);
  expect(calculateTotal(items)).toBe(expected);
});

// GOOD: Expected value is an independent, known literal
test("calculateTotal sums line items", () => {
  expect(calculateTotal([{ price: 10 }, { price: 5 }])).toBe(15);
});
```

**Call-success tests**: The assertion checks that a call was accepted, not that it did anything. A plugin host's flush test is the classic case: the plugin returns success from the flush call whether or not it read the edit, so the test passes against a plugin that drops every edit.

```cpp
// BAD: Only checks that the plugin accepted the call
TEST(ParameterFlush, editorEditIsFlushed) {
  Host host(loadPlugin("Gain.vst3"));
  host.editParameter(kGain, 0.25);
  EXPECT_EQ(host.flushParameters(), kResultOk);
}

// GOOD: Checks the effect through the interface a caller uses, against a
// test plugin that misbehaves the way real ones were seen to
TEST(ParameterFlush, editorEditReachesSavedState) {
  Host host(loadPlugin("IgnoreParameterFlush.vst3"));
  host.editParameter(kGain, 0.25);
  EXPECT_EQ(decodeState(host.saveState()).gain, 0.25);
}
```

The same shape in Rust: `assert!(instance.set_parameter(id, 0.25).is_ok())` proves only that the request was accepted. Read the value back through the public API, or render audio and check the samples.

## Where tests live

**Rust**: put seam tests in the crate's `tests/` directory. Code there sees only the public API, so a test that reaches past the interface does not compile.

```rust
// tests/state.rs: only the crate's public items are visible here
#[test]
fn edit_survives_save_and_restore() {
    let mut first = Instance::spawn(&plugin(), &Configuration::default()).unwrap();
    first.set_parameter(GAIN, 0.25).unwrap();
    let saved = first.save_state().unwrap();

    let mut second = Instance::spawn(&plugin(), &Configuration::default()).unwrap();
    second.restore_state(&saved).unwrap();
    assert_eq!(second.parameter(GAIN).unwrap(), 0.25);
}
```

**C++**: include only the public headers in the test target. A test that needs `#define private public` or a `friend` declaration is testing an internal; move it to the seam, or move the seam.
