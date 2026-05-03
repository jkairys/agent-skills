# Tasks: <feature-name>

## Mini-specs

### Task 1: <title>
- **Depends on:** independent
- **Files likely touched:** `path/a`, `path/b`
- **Acceptance:** <observable criterion>
- **Verification:** <Playwright flow / dbt test / unit + integration test>
- **Notes:** <anything specific>

### Task 2: <title>
- **Depends on:** Task 1
- **Files likely touched:** `path/c`
- **Acceptance:** <observable criterion>
- **Verification:** <method>
- **Notes:**

## Parallelisation

**Can run in parallel from the start:**
- Task 1, Task 3 (touch disjoint files)

**Sequential chains:**
- Task 1 → Task 2 → Task 4
- Task 3 → Task 5

**Total mini-specs:** N
**Parallelisable from start:** M
**Longest chain length:** K
