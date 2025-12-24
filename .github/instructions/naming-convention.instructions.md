---
applyTo: "**/*.{js,jsx,ts,tsx}"
---

# Naming Conventions for JavaScript/TypeScript Frontend

## General Principles

| Type             | Convention                           | Example                                |
| ---------------- | ------------------------------------ | -------------------------------------- |
| Components       | PascalCase                           | `UserProfile.tsx`, `DashboardCard.tsx` |
| Hooks            | camelCase with `use` prefix          | `useAuth.ts`, `useDebounce.ts`         |
| Utils/Services   | camelCase                            | `formatDate.ts`, `apiClient.ts`        |
| Constants        | UPPER_SNAKE_CASE                     | `API_BASE_URL`, `MAX_RETRY_ATTEMPTS`   |
| Types/Interfaces | PascalCase with `I` or `Type` prefix | `IUser`, `TypeProfile<T>`              |
