---
name: testing
description: >
  Unit testing workflow for pure TypeScript functions and modules.
  Provides templates and patterns for writing tests. Rules and conventions
  are in .claude/rules/testing.md (always loaded).
---

# Testing Workflow

Rules and conventions are defined in `.claude/rules/testing.md` and are always enforced.

This skill provides templates for common test patterns.

## Pure Function Test Template

```typescript
import { describe, it, expect } from 'vitest';
import { parse } from './parse';

describe('parse', () => {
  describe('when the tag is well-formed', () => {
    it('should split the language tag into its subtags', () => {
      const result = parse('en-US');
      expect(result.language).toBe('en');
      expect(result.region).toBe('US');
    });
  });

  describe('when the tag has a script subtag', () => {
    it('should extract the script', () => {
      const result = parse('zh-Hant-TW');
      expect(result.script).toBe('Hant');
    });
  });

  describe('when the tag is malformed', () => {
    it('should return null', () => {
      expect(parse('not a tag')).toBeNull();
    });
  });
});
```

## Typed Helper Pattern

When multiple tests need to narrow a return type, extract a typed helper:

```typescript
import { describe, it, expect } from 'vitest';
import { parse } from './parse';

describe('parse', () => {
  function parseSuccessfully(tag: string) {
    const result = parse(tag);
    expect(result).not.toBeNull();
    return result!;
  }

  describe('when the tag includes a region', () => {
    it('should expose both language and region', () => {
      const result = parseSuccessfully('es-419');
      expect(result.language).toBe('es');
      expect(result.region).toBe('419');
    });
  });
});
```
