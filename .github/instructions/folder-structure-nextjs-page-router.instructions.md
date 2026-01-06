---
description: "Folder structure and architecture guidelines for Next.js Page Router applications"
applyTo: "src/**"
---

# 🗄️ Folder Structure - Next.js Page Router

> **Purpose:** Define scalable folder structure and unidirectional code flow for Next.js Page Router apps  
> **Scope:** Applies to all code in the `src/` directory and `pages/` directory

---

## 🎯 Core Architecture Principles

### Unidirectional Code Flow

Code flows in **one direction**: `shared → features → pages`

```
shared modules (components, hooks, lib, types, utils)
    ↓
features (auth, users, products, etc.)
    ↓
pages (Next.js page components with routing)
```

**Rules:**

- ✅ Shared modules can be used **anywhere**
- ✅ Features can import **only from shared modules**
- ✅ Pages can import from **features and shared modules**
- ❌ **NO cross-feature imports** - compose features at page level
- ❌ **NO upward imports** - features cannot import from pages, shared cannot import from features

---

## 📁 Project Structure

```
root/
├── public/                   # Static assets
│   ├── images/
│   ├── fonts/
│   └── favicon.ico
│
└── src/
    ├── pages/               # Next.js pages (routing)
    │   ├── _app.tsx        # Custom App component
    │   ├── _document.tsx   # Custom Document
    │   ├── index.tsx       # / route (Home page)
    │   ├── about.tsx       # /about route
    │   ├── dashboard/      # /dashboard routes
    │   │   ├── index.tsx   # /dashboard route
    │   │   └── settings.tsx # /dashboard/settings route
    │   └── api/            # API routes
    │       └── hello.ts    # /api/hello endpoint
    │
    ├── assets/              # Non-static assets (imported in code)
    │   ├── icons/
    │   └── images/
    │
    ├── components/          # Shared components
    │   └── ui/             # Design system components
    │       ├── Button.tsx
    │       ├── Input.tsx
    │       └── Card.tsx
    │
    ├── config/              # Global configuration
    │   ├── env.ts          # Environment variables
    │   └── constants.ts    # App constants
    │
    ├── features/            # Feature modules (domain-driven)
    │   ├── auth/           # Authentication feature
    │   │   ├── api/
    │   │   ├── components/
    │   │   ├── hooks/
    │   │   ├── stores/
    │   │   ├── types/
    │   │   ├── utils/
    │   │   └── index.ts    # Public API
    │   ├── users/
    │   └── products/
    │
    ├── hooks/               # Shared custom hooks
    │   ├── useDebounce.ts
    │   └── useMediaQuery.ts
    │
    ├── lib/                 # Preconfigured libraries
    │   ├── utils.ts        # cn() utility
    │   ├── api-client.ts   # API client config
    │   └── query-client.ts # React Query config
    │
    ├── providers/           # React context providers
    │   ├── ThemeProvider.tsx
    │   └── AuthProvider.tsx
    │
    ├── stores/              # Global state stores
    │   └── theme-store.ts  # Theme state
    │
    ├── testing/             # Test utilities
    │   ├── test-utils.tsx
    │   └── mocks/
    │
    ├── types/               # Shared TypeScript types
    │   ├── api.types.ts
    │   └── common.types.ts
    │
    └── utils/               # Shared utility functions
        ├── format.ts
        └── validation.ts
```

---

## 🚦 Next.js Page Router Specifics

### Pages Location

**All routes live in `src/pages/` folder**

```
src/
├── pages/         # Next.js routing ✅
│   ├── index.tsx  # / route
│   └── about.tsx  # /about route
└── components/    # Application code
```

**We use `src/pages/` for consistency with the rest of our codebase.**

### Page File Structure

```tsx
// src/pages/index.tsx - Home page

import type { NextPage } from "next";
import { ProductList } from "@/features/products";
import { Button } from "@/components/ui/Button";

const HomePage: NextPage = () => {
  return (
    <div className="container mx-auto py-8">
      <h1 className="text-4xl font-bold mb-6">Welcome</h1>
      <Button>Get Started</Button>
      <ProductList />
    </div>
  );
};

export default HomePage;
```

```tsx
// src/pages/_app.tsx - Custom App component

import type { AppProps } from "next/app";
import { QueryClientProvider } from "@tanstack/react-query";
import { queryClient } from "@/lib/query-client";
import "@/styles/globals.css";

export default function App({ Component, pageProps }: AppProps) {
  return (
    <QueryClientProvider client={queryClient}>
      <Component {...pageProps} />
    </QueryClientProvider>
  );
}
```

```tsx
// src/pages/dashboard/index.tsx - Nested route

import type { NextPage } from "next";
import { useAuth } from "@/features/auth";
import { DashboardStats } from "@/features/dashboard";

const DashboardPage: NextPage = () => {
  const { user } = useAuth();

  if (!user) {
    return <div>Please log in</div>;
  }

  return (
    <div>
      <h1>Dashboard</h1>
      <DashboardStats userId={user.id} />
    </div>
  );
};

export default DashboardPage;
```

### API Routes

```tsx
// src/pages/api/hello.ts - API endpoint

import type { NextApiRequest, NextApiResponse } from "next";

type ResponseData = {
  message: string;
};

export default function handler(
  req: NextApiRequest,
  res: NextApiResponse<ResponseData>
) {
  res.status(200).json({ message: "Hello from Next.js!" });
}
```

### Layout Pattern

Since Page Router doesn't have built-in layouts, use component composition:

```tsx
// src/components/layouts/MainLayout.tsx

import type { ReactNode } from "react";
import { Header } from "./Header";
import { Footer } from "./Footer";

interface MainLayoutProps {
  children: ReactNode;
}

export const MainLayout = ({ children }: MainLayoutProps) => {
  return (
    <div className="flex min-h-screen flex-col">
      <Header />
      <main className="flex-1 container mx-auto px-4 py-6">{children}</main>
      <Footer />
    </div>
  );
};
```

```tsx
// src/pages/index.tsx - Using layout

import type { NextPage } from "next";
import { MainLayout } from "@/components/layouts/MainLayout";
import { ProductList } from "@/features/products";

const HomePage: NextPage = () => {
  return (
    <MainLayout>
      <h1>Home Page</h1>
      <ProductList />
    </MainLayout>
  );
};

export default HomePage;
```

---

## 📊 Import Rules

### ✅ Allowed Imports

```tsx
// src/components/ui/Button.tsx - Shared component
import type { ButtonProps } from "@/types/common.types"; // ✅ Shared → Shared

// src/features/auth/components/LoginForm.tsx - Feature component
import { Button } from "@/components/ui/Button"; // ✅ Feature → Shared
import { useForm } from "@/hooks/useForm"; // ✅ Feature → Shared
import { useLogin } from "../hooks/useLogin"; // ✅ Feature internal

// src/pages/dashboard/index.tsx - Page component
import { useAuth } from "@/features/auth"; // ✅ Page → Feature
import { ProductList } from "@/features/products"; // ✅ Page → Feature
import { Card } from "@/components/ui/Card"; // ✅ Page → Shared
```

### ❌ Forbidden Imports

```tsx
// ❌ BAD - Shared importing from Feature
// src/components/ui/Button.tsx
import { useAuth } from "@/features/auth"; // ❌ Shared → Feature

// ❌ BAD - Shared importing from Pages
// src/hooks/useDebounce.ts
import { HomePage } from "@/pages/index"; // ❌ Shared → Pages

// ❌ BAD - Feature importing from another Feature
// src/features/products/components/ProductCard.tsx
import { useAuth } from "@/features/auth"; // ❌ Feature → Feature

// ❌ BAD - Feature importing from Pages
// src/features/auth/components/LoginForm.tsx
import { DashboardPage } from "@/pages/dashboard"; // ❌ Feature → Pages
```

---

## 🎨 Feature Module Structure

For detailed feature module structure and best practices, see [features.instructions.md](./features.instructions.md).

---

## 🎯 Shared Hooks Structure

For shared hooks structure and best practices, see [hooks.instructions.md](./hooks.instructions.md).

---

## 🎯 Shared Utils Structure

For shared utility function structure and best practices, see [utils.instructions.md](./utils.instructions.md).

---

## 🚫 Common Mistakes

### Mistake 1: Cross-Feature Imports

```tsx
// ❌ BAD - Product feature importing from Auth feature
// src/features/products/components/ProductCard.tsx
import { useAuth } from "@/features/auth";

// ✅ GOOD - Compose at page level
// src/pages/products/index.tsx
import { useAuth } from "@/features/auth";
import { ProductCard } from "@/features/products";

const ProductsPage: NextPage = () => {
  const { user } = useAuth(); // Get auth state

  return <div>{user && <ProductCard />}</div>;
};
```

### Mistake 2: Too Much in One Feature

```
❌ BAD - Bloated "users" feature
features/users/
├── components/ (50 components)
├── hooks/ (30 hooks)
└── api/ (20 files)

✅ GOOD - Split into focused features
features/
├── user-profile/
├── user-settings/
└── user-management/
```

---

## 📝 Folder Decision Tree

**"Where should I put this file?"**

```
Is it used by multiple features?
├─ Yes → Is it UI-related?
│  ├─ Yes → src/components/ui/
│  └─ No → Is it a hook?
│     ├─ Yes → src/hooks/
│     └─ No → Is it a utility?
│        ├─ Yes → src/utils/
│        └─ No → src/lib/ or src/types/
│
└─ No → Is it specific to one feature?
   ├─ Yes → src/features/<feature-name>/
   └─ No → Is it a page/route?
      ├─ Yes → src/pages/
      └─ No → Is it app-level config?
         └─ Yes → src/config/ or src/providers/
```

---

## 🛡️ Enforce with ESLint

Prevent architectural violations with ESLint rules:

```js
// .eslintrc.js
'import/no-restricted-paths': [
  'error',
  {
    zones: [
      // 🚫 NO cross-feature imports
      {
        target: './src/features/auth',
        from: './src/features',
        except: ['./auth'],
      },
      {
        target: './src/features/products',
        from: './src/features',
        except: ['./products'],
      },

      // 🚫 NO upward imports (shared → features/pages)
      {
        target: [
          './src/components',
          './src/hooks',
          './src/lib',
          './src/types',
          './src/utils',
        ],
        from: ['./src/features', './src/pages'],
      },

      // 🚫 NO features → pages imports
      {
        target: './src/features',
        from: './src/pages',
      },
    ],
  },
]
```

---

## 📚 Summary

### Key Takeaways

1. **Pages** (routing) → `src/pages/` folder
2. **Application code** → `src/` folder
3. **Feature code** → `src/features/<feature-name>/`
4. **Shared code** → `src/components/`, `src/hooks/`, `src/utils/`
5. **Unidirectional flow** → `shared → features → pages`
6. **No cross-feature imports** → Compose at page level
7. **Minimal exports** → Only export what's needed from features
8. **Data fetching** → Use Next.js data fetching methods in pages
9. **Layouts** → Use component composition pattern
10. **API routes** → Keep in `src/pages/api/`

---
