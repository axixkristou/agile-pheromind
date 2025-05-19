# Project {{PROJECT_NAME_FROM_PHEROMONE_EN}} Design and UI/UX Conventions

**Version:** 1.0
**Last Updated:** {{TIMESTAMP_BY_PHEROMIND_AGENT}}
**Maintained by:** @architecture-advisor-agent (with human validation from Tech Lead/Design Lead)

## 1. Introduction

This document defines the English design principles, standards, and UI components to be used for the `{{PROJECT_NAME_FROM_PHEROMONE_EN}}` project. The goal is to create a consistent, intuitive, accessible, and aesthetically pleasing user experience (UX), aligned with the brand identity (if any) and user expectations. These conventions will guide frontend development (Angular) and the use of the styling library (Tailwind CSS) and component library (e.g., based on Shadcn/UI if chosen).

Adherence to these conventions is crucial for maintaining high UI/UX quality. This document is a key part of the project's `memoryBank` (referenced via `documentationRegistry`).

## 2. UX/UI Design Philosophy and Principles (English)

*   **User-Centered Design:** End-users' needs and goals are central to all design decisions.
*   **Simplicity and Clarity:** "Less is more." Clean interfaces, hierarchized information, clear English (UI text can be localized later). Avoid cognitive overload.
*   **Consistency:** UI elements, interactions, user flows must be consistent. Use components and styles defined here.
*   **Intuitive Design:** Easy to understand and use without complex explanations.
*   **User Feedback:** Provide clear, immediate feedback to user actions.
*   **Efficiency:** Enable users to accomplish tasks quickly. Optimize workflows.
*   **Accessibility (A11Y):** Design for everyone. Aim for WCAG 2.1 AA compliance.
*   **Aesthetic Integrity:** Visual design supports and enhances functionality. Aligned with project "vibe".
*   **Forgiveness (Error Tolerance):** Help users avoid and recover from errors easily.

## 3. Visual Identity (Branding - English descriptions of concepts)

*   **3.1. Logo:**
    *   [Space for primary logo and variations. AgilePheromind AI agent can suggest a placeholder or assist if requested.]
*   **3.2. Primary Color Palette (Tailwind CSS Tokens - English names):**
    *   `primary`: [e.g., `colors.blue['500']` - HEX: `#3B82F6`] - For main actions, important accents.
    *   `secondary`: [e.g., `colors.slate['600']` - HEX: `#475569`] - For supporting elements, secondary text.
    *   `accent`: [e.g., `colors.amber['400']` - HEX: `#FBBF24`] - For highlights, notifications.
    *   **Neutrals (English names):**
        *   `background`: [e.g., `colors.white` or `colors.slate['50']`]
        *   `foreground`: [e.g., `colors.slate['900']` or `colors.slate['700']`]
        *   `card / surface`: [e.g., `colors.white` or `colors.slate['100']`]
        *   `border / divider`: [e.g., `colors.slate['200']` or `colors.slate['300']`]
    *   **Semantic Colors (English names):**
        *   `destructive / error`: [e.g., `colors.red['500']`]
        *   `success`: [e.g., `colors.green['500']`]
        *   `warning`: [e.g., `colors.yellow['500']`]
        *   `info`: [e.g., `colors.sky['500']`]
    *   *Note: Configure in `tailwind.config.js`.*
*   **3.3. Typography (Tailwind CSS Tokens - English font names if specific):**
    *   **Primary Font (Body & UI):** [Font Name, e.g., "Inter"]. Weights (e.g., Regular 400, Medium 500, Bold 700). Fallback: `sans-serif`.
    *   **Display Font (Headings, optional):** [Font Name]. Weights.
    *   **Typographic Scale (English usage notes):** Tailwind classes or `theme.fontSize` config. Define appropriate `line-height`.
*   **3.4. Iconography (English style description):**
    *   **Style:** [e.g., Outline, Solid].
    *   **Recommended Library:** [e.g., Heroicons, Lucide Icons]. Ensure consistency.
    *   Base icon size: [e.g., 20px, 24px].
*   **3.5. Spacing and Grid (Tailwind CSS Tokens - English unit notes):**
    *   **Base Spacing Unit:** [e.g., 4px]. Tailwind's `spacing` scale based on this.
    *   **Layout Grid:** Use Tailwind's grid or Flexbox. Consistent margins/paddings.
*   **3.6. Corner Radius (Border Radius - Tailwind CSS Tokens - English scale names):**
    *   Define scale (e.g., `rounded-sm`, `rounded-md`, `rounded-lg`). Consistent use.
*   **3.7. Shadows (Box Shadow - Tailwind CSS Tokens - English scale names):**
    *   Define subtle shadow scale for elevation.
*   **3.8. Overall Project "Vibe" (English keywords based on client/PO input):**
    *   [e.g., "Modern and clean", "Professional and reliable", "Playful and engaging"]. These English keywords guide aesthetics.

## 4. Standard UI Components (English Naming and Props - Based on Shadcn/UI or chosen library, customized)

*The `@architecture-advisor-agent` or `@developer-agent` documents key English-named components here, their variants, and English prop names/usage, ensuring adherence to this document's tokens.*

*   **4.1. Buttons (`<button>`, `<a>` styled - English variants):**
    *   Variants: `primary`, `secondary`, `destructive`, `outline`, `ghost`, `link`.
    *   States: `default`, `hover`, `focus`, `active`, `disabled`.
    *   Sizes: `sm`, `default` (md), `lg`, `icon`.
*   **4.2. Form Fields (`<input>`, `<textarea>`, `<select>` - English states/types):**
    *   Input types: `text`, `email`, `password`, `number`, `date`. States: `default`, `disabled`, `error`. Labels, help text, error messages (UI text can be localized, but component prop names for messages are English).
*   **4.3. Cards (English structure):** Base structure (header, content, footer). Variants.
*   **4.4. Modals (Dialogs - English structure):** Structure, behavior.
*   **4.5. Notifications (Toasts, Alerts - English types):** Types: `info`, `success`, `warning`, `error`.
*   **4.6. Navigation (English component names):** Main Nav, Sidebar, Breadcrumbs, Pagination, Tabs.
*   **4.7. Data Tables (English features):** Structure, sorting, basic filtering, row actions.
*   **4.8. Avatars, Badges, Tooltips, Dropdowns, Accordions, etc. (English names).**

## 5. Interaction Principles and Animations (English Descriptions)

*   **Immediate Feedback:** Visual feedback for every user action.
*   **Smooth Transitions:** Purposeful, non-distracting animations (e.g., Tailwind's `duration-200`, `ease-in-out`).
*   **Loading States:** Clear indicators (spinners, skeletons).
*   **Micro-interactions:** Subtle animations on hover, focus, clicks if well-dosed.

## 6. Accessibility (A11Y - English Guidelines based on WCAG)

*   **Color Contrast:** Adhere to WCAG AA ratios.
*   **Keyboard Navigation:** All interactive elements keyboard accessible/operable. Logical focus order.
*   **ARIA:** Use appropriately for complex components.
*   **Alternative Text for Images:** Descriptive `alt` for informative images. `alt=""` for decorative.
*   **Labels for Forms:** All fields must have associated labels.
*   **Semantic HTML:** Use tags semantically.
*   **Accessibility Testing:** Use automated tools (Axe DevTools) and manual testing.

## 7. Responsive Design (English Principles)

*   **Mobile-First Approach.**
*   **Tailwind CSS Breakpoints:** Use consistently.
*   Test on various screen sizes. Ensure readability/usability.

## 8. Tools and Resources (English List)

*   **UI Library:** [Name of base library].
*   **Icon Library:** [Name and link].
*   **Design Tool (if used for mockups):** [e.g., Figma, Penpot]. Links to master English mockups.
*   **Contrast Checker:** [Link].
*   **Browser Accessibility Extension:** [e.g., Axe DevTools].

## 9. Update Process (English)
This English document is living. `@architecture-advisor-agent` proposes updates. Changes validated by Tech Lead/Design Lead. Version and update date maintained.

---