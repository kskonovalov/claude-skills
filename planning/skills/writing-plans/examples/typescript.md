# Task Structure Example — TypeScript / Vitest

```markdown
### Task N: [Component Name]

**Files:**
- Create: `src/exact/path/to/file.ts`
- Modify: `src/exact/path/to/existing.ts:123-145`
- Test: `src/exact/path/to/file.test.ts`

- [ ] **Step 1: Write the failing test**

```typescript
import { specificFunction } from './file';

it('should do specific behavior', () => {
  const result = specificFunction(input);
  expect(result).toEqual(expected);
});
```

- [ ] **Step 2: Run test to verify it fails**

Run: `npx vitest run src/exact/path/to/file.test.ts`
Expected: FAIL with "specificFunction is not a function"

- [ ] **Step 3: Write minimal implementation**

```typescript
export function specificFunction(input: InputType): OutputType {
  return expected;
}
```

- [ ] **Step 4: Run test to verify it passes**

Run: `npx vitest run src/exact/path/to/file.test.ts`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add src/exact/path/to/file.ts src/exact/path/to/file.test.ts
git commit -m "feat: add specific feature"
```
```
