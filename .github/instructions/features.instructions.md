---
description: "Feature-based architecture guidelines for organizing domain-specific code"
applyTo: "src/features/**"
---

# Feature-Based Architecture Guidelines

> **Purpose:** Standards for organizing feature modules with domain-driven design  
> **Scope:** This file applies to all code in the `src/features/` directory

---

## 🎯 What is a Feature Module?

A feature is a **self-contained domain module** that encapsulates all code related to a specific business capability.  
Think: `auth`, `users`, `products`, `orders`, `payments`, `notifications`.

### Core Principles:

- ✅ **Domain-driven** - Organized by business capability, not technical layer
- ✅ **Self-contained** - Everything for the feature lives in its folder
- ✅ **Encapsulated** - Internal implementation details are private
- ✅ **Composable** - Features can use shared code and other features' public APIs
- ✅ **Testable** - Each feature can be tested in isolation

---

## 📁 Feature Module Structure

```
src/features/
├── auth/                      # Authentication feature
│   ├── api/                   # API calls (services)
│   │   ├── auth-api.ts       # Auth API functions
│   │   └── index.ts          # Public API exports
│   ├── assets/               # Feature-specific assets
│   │   ├── auth-bg.jpg
│   │   └── logo.svg
│   ├── components/           # Feature components
│   │   ├── LoginForm.tsx
│   │   ├── RegisterForm.tsx
│   │   └── PasswordReset.tsx
│   ├── hooks/                # Feature hooks
│   │   ├── useAuth.ts
│   │   ├── useLogin.ts
│   │   └── index.ts          # Public hook exports
│   ├── stores/               # Feature state
│   │   ├── auth-store.ts
│   │   └── index.ts
│   ├── types/                # Feature types
│   │   ├── auth. types.ts
│   │   └── index.ts
│   ├── utils/                # Feature utilities
│   │   ├── token.ts
│   │   ├── validation.ts
│   │   └── index.ts
│   └── index.ts              # Public feature API
│
├── products/                 # Products feature
│   ├── api/
│   ├── components/
│   │   ├── ProductCard.tsx
│   │   ├── ProductList.tsx
│   │   └── ProductDetail. tsx
│   ├── hooks/
│   ├── stores/
│   ├── types/
│   └── index.ts
│
└── orders/                   # Orders feature
    ├── api/
    ├── components/
    ├── hooks/
    ├── stores/
    ├── types/
    └── index.ts
```

## 📊 Feature vs Shared Code

### When to Put Code in a Feature:

**✅ Feature-specific code:**

- Components used only by this feature
- Business logic specific to this domain
- Types that represent domain concepts
- API calls for this feature's endpoints
- State management for this feature's data

**❌ Not feature code (use shared):**

- UI components used across features → `src/components/ui`
- Generic hooks (useDebounce, useMediaQuery) → `src/hooks`
- Generic utilities (formatDate, slugify) → `src/utils`
- App-wide state (theme, locale) → `src/lib` or root `stores`
- Shared types (ApiResponse, PaginatedData) → `src/types`

---

### Example: Products Feature

```
features/products/
├── api/
│   ├── products-api.ts      # CRUD operations for products
│   └── index.ts
├── components/
│   ├── ProductCard.tsx      # Display single product
│   ├── ProductList.tsx      # Display list of products
│   ├── ProductFilters.tsx   # Filter products
│   └── ProductDetail.tsx    # Product detail view
├── hooks/
│   ├── useProducts.ts       # Fetch products list
│   ├── useProduct.ts        # Fetch single product
│   ├── useCreateProduct.ts  # Create product
│   └── index.ts
├── types/
│   ├── product.types.ts     # Product, Category, etc.
│   ├── schemas.ts           # Zod schemas
│   └── index.ts
├── utils/
│   ├── price.ts             # Price calculations
│   └── index.ts
└── index.ts                 # Export ProductCard, ProductList, useProducts
```

---

## ✅ Feature Module Best Practices

### 1. Keep Features Independent

```tsx
// ✅ GOOD - Features communicate through public APIs
import { useAuth } from "@/features/auth";
import { useProducts } from "@/features/products";

// ❌ BAD - Direct coupling to internals
import { authStore } from "@/features/auth/stores/auth-store";
```

### 2. Export Minimal Public API

```tsx
// features/auth/index.ts

// ✅ GOOD - Minimal, intentional exports
export { useAuth, useLogin } from "./hooks";
export { LoginForm } from "./components/LoginForm";
export type { User } from "./types";

// ❌ BAD - Export everything
export * from "./components";
export * from "./hooks";
export * from "./api";
export * from "./stores";
```

---

## 🚫 Common Mistakes

### Mistake 1: Exposing Internal Implementation

```tsx
// ❌ BAD
// features/auth/index.ts
export * from "./stores/auth-store"; // Exposes store internals

// ✅ GOOD
// features/auth/index.ts
export { useAuth } from "./hooks/useAuth"; // Hook wraps store
```

### Mistake 2: Feature Importing Another Feature's Internals

```tsx
// ❌ BAD
import { userApi } from "@/features/users/api/user-api";

// ✅ GOOD
import { useUsers } from "@/features/users";
```

### Mistake 3: Putting Shared Code in Feature

```tsx
// ❌ BAD
// features/auth/utils/format-date.ts - Used by many features

// ✅ GOOD
// src/utils/format-date.ts - Shared utility
```

### Mistake 4: Too Much in One Feature

```tsx
// ❌ BAD - "users" feature handles everything user-related
features/users/
├── components/ (50 components)
├── hooks/ (30 hooks)
└── api/ (20 API files)

// ✅ GOOD - Split into focused features
features/
├── user-profile/
├── user-settings/
└── user-management/
```

---
