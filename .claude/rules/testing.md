# Testing Conventions

- Framework: Vitest with `@vitest/coverage-v8`.
- Coverage: aim high but no enforced threshold — `vitest run --coverage` is available via `pnpm test:coverage`.
- Specs co-located: `parse.spec.ts` next to `parse.ts` in `src/`.
- `vi.fn()` and `vi.spyOn()` for mocks.
- `vi.stubGlobal('fetch', ...)` + `afterEach(() => vi.unstubAllGlobals())` for HTTP mocks.

## Rules

```
RULE-01  No redundant "should create" / "should be created" tests.
         Function/module will fail in subsequent tests if broken.
RULE-02  Nested describes group tests by state or scenario.
         Never flat -- always at least one level of nesting.
RULE-03  Each describe that changes state has its own beforeEach.
         beforeEach performs EXACTLY ONE action or state change.
RULE-04  "it" blocks contain ONLY expectations.
         No setup, no act -- those go in beforeEach or helper functions.
RULE-05  Input variations -> describe starts with "when..."
         Example: "when the tag is invalid", "when the XML is empty"
RULE-06  restoreMocks: true in vitest config -- no manual
         afterEach(vi.restoreAllMocks()) needed.
RULE-07  Prefer vi.spyOn over vi.mock. vi.mock is hoisted, global,
         and confusing. Only use for dynamic imports.
RULE-08  One logical assertion per "it". Related expects on the SAME
         subject are OK (e.g. text + attributes from same parse).
RULE-09  vi.fn() for mocks. Never third-party spy libraries.
RULE-10  Spec files use suffix *.spec.ts.
RULE-11  No comments in spec files unless logic is extremely complex.
         Use semantic function names and describe text instead.
RULE-12  Root describe contains ONLY the entity name under test.
         describe('readTag', () => { ... })
         NOT: describe('xml readTag', () => { ... })
RULE-13  Describe text uses semantic, human-readable names.
         "when the tag has attributes" NOT "when attrs !== null"
RULE-14  No conditional logic (if/switch) in spec files.
         Use hardcoded values and explicit assertions.
RULE-15  Avoid type casts (as Xxx) in specs -- type correctly upfront.
RULE-16  Each test must be independent -- no shared mutable state
         across it blocks.
RULE-17  Nested describe blocks start with "when...".
RULE-18  it blocks start with "should...".
```

## File Ordering Inside a Spec

```
1. Imports
2. describe('EntityName', () => {
3.   let variables
4.   Helper functions
5.   beforeEach
6.   Nested describes
7. })
```

## Spec Location

Test files live alongside source files in `src/` directory:

```
src/
  parse.ts
  parse.spec.ts
```
