```markdown
# funagency-miniapp-mockup Development Patterns

> Auto-generated skill from repository analysis

## Overview
This skill teaches the core development patterns and conventions used in the `funagency-miniapp-mockup` TypeScript repository. It covers file naming, import/export styles, commit conventions, and testing patterns. While no specific framework or automated workflows are detected, this guide ensures consistency and clarity for contributors.

## Coding Conventions

### File Naming
- **Pattern:** PascalCase
- **Example:**  
  ```plaintext
  UserProfile.ts
  MainScreen.ts
  ```

### Import Style
- **Pattern:** Relative imports
- **Example:**  
  ```typescript
  import { UserProfile } from './UserProfile';
  import { utils } from '../utils/helpers';
  ```

### Export Style
- **Pattern:** Named exports
- **Example:**  
  ```typescript
  // In UserProfile.ts
  export function UserProfile(props: Props) { ... }
  ```

### Commit Message Convention
- **Type:** Conventional Commits
- **Prefix:** `feat`
- **Example:**  
  ```
  feat: add user profile component with avatar support
  ```

## Workflows

### Creating a New Feature
**Trigger:** When adding a new component, utility, or feature  
**Command:** `/create-feature`

1. Create a new file using PascalCase (e.g., `NewFeature.ts`).
2. Use relative imports to include dependencies.
3. Export your component or function using named exports.
4. Write a test file named `NewFeature.test.ts` (see Testing Patterns).
5. Commit your changes using the `feat:` prefix and a descriptive message.

### Refactoring Code
**Trigger:** When improving or restructuring existing code  
**Command:** `/refactor-code`

1. Identify the file(s) to refactor.
2. Maintain PascalCase for file names.
3. Update imports/exports to remain relative and named.
4. Run or update corresponding test files.
5. Commit with a descriptive message, e.g., `feat: refactor UserProfile for performance`.

## Testing Patterns

- **Test File Naming:**  
  Test files follow the pattern `*.test.*` (e.g., `UserProfile.test.ts`).
- **Framework:**  
  No specific testing framework detected. Use your preferred TypeScript-compatible test runner.
- **Example:**  
  ```typescript
  // UserProfile.test.ts
  import { UserProfile } from './UserProfile';

  describe('UserProfile', () => {
    it('renders correctly', () => {
      // test implementation
    });
  });
  ```

## Commands
| Command          | Purpose                                      |
|------------------|----------------------------------------------|
| /create-feature  | Scaffold a new feature/component             |
| /refactor-code   | Refactor existing code following conventions |
```
