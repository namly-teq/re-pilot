---
description: "Generate a new pure, reusable UI component for the design system"
name: "ui"
tools: ["codebase"]
---

Your goal is to generate a new UI component in `src/components/ui/` that follows our design system standards.

Requirements for UI components (see `.github/copilot-instructions/ui. md` for full guidelines):

**Pure & Presentational:**

- No business logic, API calls, or state management
- Can ONLY import from `@/components/ui`, `@/hooks`, `@/utils`, `@/lib`, `@/types`, `@/assets`
- CANNOT import from `@/components/features`, `@/components/layouts`, `@/app`

**CSS Strategy (OOCSS - Separation of Structure and Skin):**

_Structural Classes (Layout & Positioning):_

- **Prefer `display: block`** - Use the natural document flow when possible
- Only use `display: flex` or `display: grid` when you need:
  - Items arranged horizontally
  - Space distribution between elements
  - Alignment that can't be achieved with block layout
- **Prefer `position: relative`** over `absolute` or `fixed`
  - Use relative positioning for minor adjustments
  - Only use `absolute` when element must be removed from document flow (dropdowns, tooltips, overlays)
  - Only use `fixed` for persistent UI (headers, modals, toast notifications)
- Structural properties define layout: `width`, `height`, `padding`, `margin`, `position`, `display`

_Skin Classes (Visual Appearance):_

- Skin properties define appearance: `color`, `background`, `border`, `font-family`, `font-size`, `box-shadow`
- Variants should primarily change skin properties, not structure
- Structure should remain consistent across variants

_Example - Button Component:_

```tsx
// ❌ BAD - Mixing structure and skin, overusing flex and Violates OOCSS Principles

interface BadButtonProps {
  variant?: "primary" | "secondary" | "ghost" | "outline";
  // ❌ No size prop - structure is hardcoded in variants
  children: React.ReactNode;
  onClick?: () => void;
}

const badButtonVariants = cva(
  // ❌ Base:  Unnecessary flex, absolute positioning by default
  "flex items-center justify-center absolute top-0 left-0 font-bold cursor-pointer",
  {
    variants: {
      variant: {
        // ❌ Each variant has different structure (padding, width, position)
        primary:
          "bg-blue-500 text-white px-8 py-4 w-full rounded-lg static shadow-lg border-2 border-blue-500",

        secondary:
          "bg-gray-200 text-gray-900 px-4 py-2 w-auto rounded-md relative shadow-sm border border-gray-300",

        ghost:
          "bg-transparent text-gray-700 px-2 py-1 w-fit rounded-none fixed top-10 right-0 border-none",

        outline:
          "bg-white text-blue-600 px-6 py-3 w-64 rounded-full absolute bottom-0 border-2 border-blue-600 shadow-xl",
      },
    },
    defaultVariants: {
      variant: "primary",
    },
  }
);

const BadButton: React.FC<BadButtonProps> = ({
  variant,
  children,
  onClick,
}) => {
  return (
    // ❌ Inline styles mixed with classes
    <button
      className={badButtonVariants({ variant })}
      onClick={onClick}
      style={{
        zIndex: variant === "ghost" ? 9999 : 1,
        transform: variant === "outline" ? "scale(1.1)" : "none",
      }}
    >
      {/* ❌ Icon hardcoded in component instead of being composable */}
      {variant === "primary" && <span className="mr-2">🚀</span>}
      {children}
    </button>
  );
};
export default BadButton;

// ✅ GOOD - Separated structure and skin, minimal flex
const buttonVariants = cva(
  // Base Structure (Layout & Positioning) - Always applied
  "inline-block text-center border border-solid transition-colors duration-200",
  {
    variants: {
      variant: {
        // Skin only (Visual Appearance) - No structural properties
        primary:
          "bg-blue-600 text-white border-blue-600 hover:bg-blue-700 hover:border-blue-700",
        secondary:
          "bg-gray-200 text-gray-900 border-gray-300 hover:bg-gray-300",
        ghost:
          "bg-transparent text-gray-700 border-transparent hover:bg-gray-100",
      },
      size: {
        // Structure variations (Spacing & Typography size only)
        sm: "px-3 py-1.5 text-sm",
        md: "px-4 py-2 text-base",
        lg: "px-6 py-3 text-lg",
      },
    },
    defaultVariants: {
      variant: "primary",
      size: "md",
    },
  }
);

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: "primary" | "secondary" | "ghost";
  size?: "sm" | "md" | "lg";
  children: React.ReactNode;
}

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ variant, size, className, children, ...props }, ref) => {
    return (
      <button
        ref={ref}
        className={cn(buttonVariants({ variant, size }), className)}
        {...props}
      >
        {children}
      </button>
    );
  }
);

Button.displayName = "Button";
```

**Styling with CVA:**

- Use `class-variance-authority` for variants: [cva documentation](https://cva.style/docs)
- Base classes always applied, variants for modifications
- Define `defaultVariants` for all variant groups

**TypeScript:**

- Extend appropriate HTML element props (`HTMLAttributes<HTMLElement>`)
- Include `VariantProps<typeof componentVariants>`
- JSDoc comments for all props with `@example` usage
- Use `forwardRef` if wrapping native elements

**Accessibility:**

- Use semantic HTML elements (`<button>`, `<nav>`, not `<div onClick>`)
- Ensure keyboard navigation (Tab, Enter, Escape)

**Component Structure:**

```tsx
import { type FC, forwardRef, type HTMLAttributes } from "react";
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "@/lib/utils";

const componentVariants = cva("base-classes", {
  variants: {
    variant: {
      default: "variant-classes",
    },
    size: {
      default: "size-classes",
    },
  },
  defaultVariants: {
    variant: "default",
    size: "default",
  },
});

interface ComponentNameProps
  extends HTMLAttributes<HTMLDivElement>,
    VariantProps<typeof componentVariants> {
  /** Prop description */
  customProp?: string;
}

export const ComponentName = forwardRef<HTMLDivElement, ComponentNameProps>(
  ({ variant, size, className, children, ...props }, ref) => {
    return (
      <div
        ref={ref}
        className={cn(componentVariants({ variant, size }), className)}
        {...props}
      >
        {children}
      </div>
    );
  }
);

ComponentName.displayName = "ComponentName";
```

Generate the component following these standards.
