---
description: 'Analyze Figma CSS to generate component specifications and hierarchy'
tools:  ['codebase']
---

Your goal is to analyze CSS copied from Figma and generate a detailed component specification with hierarchy and implementation guidelines.

## Analysis Output Structure

### 1. Component Overview
Provide a high-level description of what this component is and its purpose.

**Example:**
```
Component: LoginCard
Purpose: Authentication form container for user login
Type: Feature Component (src/features/auth/components/)
```

### 2. Visual Hierarchy Analysis

Identify the component tree structure from the CSS classes.

**Format:**
```
LoginCard (Container)
├── LoginHeader (Header Section)
│   ├── LoginTitle (Heading)
│   └── LoginSubtitle (Description)
├── LoginForm (Form Section)
│   ├── EmailInput (Input Field)
│   ├── PasswordInput (Input Field)
│   └── SubmitButton (Button)
└── LoginFooter (Footer Section)
    └── SignupLink (Link)
```

### 3. CSS Strategy Analysis 

Analyze the CSS and separate **Structure** from **Skin** following our CSS strategy.

**Structure (Layout & Positioning):**
- Identify:  `display`, `position`, `width`, `height`, `padding`, `margin`, `gap`, `flex-direction`, `align-items`, `justify-content`
- Evaluate: Is `display: flex` necessary? Could `display: block` work?
- Evaluate: Is positioning appropriate? (prefer `relative` over `absolute`)

**Skin (Visual Appearance):**
- Identify: `color`, `background`, `border`, `border-radius`, `box-shadow`, `font-family`, `font-size`, `font-weight`

### 4. Tailwind CSS Conversion

Convert the extracted CSS to Tailwind classes following our CSS strategy.

**Rules:**
- Prefer `block` display, only use `flex` when justified
- Separate structure (spacing, layout) from skin (colors, shadows)
- Use Tailwind design tokens when possible
- Create custom classes for complex values

### 6. Component Breakdown

Break down into atomic components following our component hierarchy.


### 7. Recommended Implementation

Provide implementation recommendations with code structure.

### 8. Responsive Design Analysis

Analyze breakpoints and responsive behavior.


## Analysis Checklist

For every CSS analysis, ensure you provide:

- [ ] **Component Overview** - Name, purpose, type
- [ ] **Visual Hierarchy** - Component tree structure
- [ ] **CSS Strategy Analysis** - Structure vs Skin separation with justification
- [ ] **Design Tokens** - Extracted reusable values
- [ ] **Tailwind Conversion** - Following our CSS strategy
- [ ] **Component Breakdown** - UI vs Feature components
- [ ] **Implementation Recommendations** - Code structure with CVA
- [ ] **Responsive Design** - Breakpoints and mobile-first approach
- [ ] **Accessibility** - WCAG compliance and semantic HTML
- [ ] **File Structure** - Where to create files

---
