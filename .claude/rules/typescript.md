# TypeScript Conventions

- NEVER: **Comments** in any files unless the logic is extremely complex and cannot be clarified through naming. Use semantic function names, variable names, and describe text for readability.
- DO: `Array<T>` not `T[]`.
- DO: **`readonly`** on every field that is never reassigned. Enforced by `@typescript-eslint/prefer-readonly`.
- DO: **Type safety** -- avoid primitive types (`string`, `number`) where a union or domain type exists. Define a union type when a finite set of values is known.
- NEVER: **`any`** -- no `any`, `as any`, `as unknown`, `as unknown as T`. Use typed alternatives.
- DO: **Explicit types on external returns** -- when a function from an external dependency returns `any`, add an explicit type annotation to the variable.
- PREFER: **String union types over `enum`** -- use `type Foo = 'a' | 'b'` always.
- DO: **Always `async`/`await`** -- never use `.then()`/`.catch()` chains. Always `await` in an `async` function. Never ignore a `Promise`. Any function that returns a `Promise` must be declared `async` -- never return a bare `Promise<T>` without the `async` modifier.
- PREFER: **`undefined` over `null` for absent return values** -- use `T | undefined` when a function may not produce a result. `undefined` is idiomatic in this codebase.
- DO: **Readability over cleverness** -- no one-liner clever solutions (chained `reduce` + `??`, nested ternaries, etc.). Prefer explicit control flow (`if`, `for`, early returns) that reads top-to-bottom.
- DO: **Early returns first** -- handle fast/error/empty cases at the top with early returns. The main logic follows unindented.
- NEVER: **Inline if + return** -- always use braces and break `if`/`return` into separate lines. Never `if (x) return y;` on one line.
- DO: **Empty catch blocks** -- add `// noop` comment to clarify intent.
- DO: **Only export what has external consumers** -- internal implementation details stay unexported.
- DO: **Export input/output types of exported functions** -- when a function is exported, its parameter interfaces and return type interface must also be exported.
- DO: **`const` by default**, `let` only when reassignment is needed.
- DO: **`import type` for type-only imports** (verbatimModuleSyntax).
- DO: **Explicit return types on exported functions**.
- DO: **`noImplicitOverride: true`** -- always use the `override` keyword on overridden members.
- DO: **Explicit member accessibility** -- `public`, `protected`, or `private` on every class member. Exception: constructors.
- NEVER: **Backwards-compatibility shims** -- no re-exports, aliases, or wrappers when renaming code. Update all consumers directly.
- NEVER: **File extensions in imports** -- no `.js` or `.ts` extensions on any import specifier. A post-build script adds `.js` to compiled output for Node.js ESM resolution.
- DO: **`index.ts` as the single package entry point** -- each package has one `src/index.ts` re-exporting its public API. No other barrel files.
- DO: **`interfaces/` folder** -- group interface/type-only files under an `interfaces/` directory (not `types/`). Name these files with a `.interface.ts` suffix: `eutils-config.interface.ts`, `search-params.interface.ts`.
- NEVER: **Redundant `| undefined` on optional properties** -- `readonly prop?: string` is sufficient. Never write `readonly prop?: string | undefined`.

| Instead of           | Use                                       |
| -------------------- | ----------------------------------------- |
| `any` param/return   | Interface, inline type, or generic `<T>`  |
| `as SomeType`        | Type annotation `: SomeType`              |
| `as unknown as T`    | Fix the type design                       |
| Cast to narrow union | Type guard (`instanceof`, `in`, `typeof`) |

## Naming

| Target          | Convention                                                     | Example                 |
| --------------- | -------------------------------------------------------------- | ----------------------- |
| Files           | `kebab-case.ts` or `kebab-case.interface.ts` (type-only files) | `pubtator.interface.ts` |
| Interfaces      | `PascalCase`                                                   | `BioDocument`           |
| Functions       | `camelCase`                                                    | `readTag`               |
| Constants       | `UPPER_SNAKE_CASE`                                             | `MAX_RETRIES`           |
| Private members | `_camelCase`                                                   | `_cache`                |

## Semantic Naming

**Every name must be immediately understandable by a human reader.** The name alone should tell you what something is, what it does, or what it holds -- without needing to read the implementation.

### Files

Name files after what they contain, not their role. A file name should tell you the domain concept inside it.

| Bad            | Good                       | Why                                         |
| -------------- | -------------------------- | ------------------------------------------- |
| `helpers.ts`   | `article-field-parsers.ts` | Describes what the functions actually parse |
| `utils.ts`     | `month-conversion.ts`      | Describes the specific utility              |
| `common.ts`    | `xml-reader.ts`            | Describes the shared logic                  |
| `constants.ts` | `ncbi-endpoints.ts`        | Describes the domain of the constants       |

NEVER use generic file names: `helpers`, `utils`, `common`, `shared`, `misc`, `lib`, `tools`, `stuff`.

### Functions

Name functions as `verb + domain noun`. The name should describe the transformation or action, not just that it "processes" something.

| Bad                | Good                                 | Why                           |
| ------------------ | ------------------------------------ | ----------------------------- |
| `process(xml)`     | `parseAbstract(articleXml)`          | Specific verb + specific noun |
| `handle(data)`     | `extractArticleIds(pubmedDataBlock)` | Describes input->output       |
| `getData()`        | `fetchPubmedArticles()`              | Describes what data           |
| `transform(input)` | `readTag(xml, tagName)`              | Describes the operation       |

### Variables and Parameters

No single-letter variables. No cryptic abbreviations. No generic names like `data`, `result`, `item`, `value`, `input`, `output`, `temp`, `info`, `obj`. Name after the domain concept they represent.

```ts
scopes.map((scope) => scope.value); // not (s) => s.value
entries.find((entry) => entry.path === ""); // not (r) => r.path === ''
```

| Bad      | Good                                              |
| -------- | ------------------------------------------------- |
| `data`   | `articleXml`, `meshHeadings`, `grantList`         |
| `result` | `parsedArticle`, `searchResponse`, `convertedIds` |
| `item`   | `meshHeading`, `author`, `keyword`                |
| `input`  | `rawXml`, `medlineText`, `citationString`         |
| `temp`   | `accumulatedBuffer`, `pendingChunk`               |

## Imports

- `import type` for type-only when possible.
- No blank lines between imports.
- Order: third-party packages -> `@ncbijs/*` packages -> relative imports.
- No file extensions (`.js`, `.ts`) on any import specifier. A post-build script adds `.js` to compiled output.
- Import directly from the source file within a package, never through an intermediate barrel.

## Formatting

Single quotes, trailing commas (`all`), 100 char width, 2-space indent, LF.
