# Project {{PROJECT_NAME_FROM_PHEROMIND}} Design and UI/UX Conventions

**Version:** 1.0
**Last Updated:** {{TIMESTAMP_BY_PHEROMIND_AGENT}}
**Maintained by:** @architecture-advisor-agent (with human validation from Tech Lead/Design Lead)

## 1. Introduction

This document defines the design principles, standards, and UI components to be used for the `{{PROJECT_NAME_FROM_PHEROMIND}}` project. The goal is to create a consistent, intuitive, accessible, and aesthetically pleasing user experience (UX), aligned with the brand identity and user expectations. These conventions will guide frontend development (Angular) and the use of the styling library (Tailwind CSS) and component library (e.g., based on Shadcn/UI if chosen).

Adherence to these conventions is crucial for maintaining high UI/UX quality. This document is a key part of the project's `memoryBank` and will be used by AI agents like `@developer-agent` and `@tester-ui-validator-agent`.

## 2. UX/UI Design Philosophy and Principles

*   **User-Centered Design:** The needs and goals of end-users are at the heart of all design decisions.
*   **Simplicity and Clarity:** "Less is more." Clean interfaces, hierarchized information, clear language. Avoid cognitive overload.
*   **Consistency:** UI elements, interactions, and user flows must be consistent throughout the application. Use components and styles defined herein.
*   **Intuitive Design:** The interface should be easy to understand and use without complex explanations. Users should be able to anticipate the outcome of their actions.
*   **User Feedback:** Provide clear and immediate feedback to user actions (e.g., loading states, success/error messages, visual confirmations).
*   **Efficiency:** Enable users to accomplish tasks quickly and with minimal effort. Optimize workflows.
*   **Accessibility (A11Y):** Design for everyone. Aim for at least WCAG 2.1 AA compliance. (See dedicated section).
*   **Aesthetic Integrity:** Visual design should support and enhance functionality, not overwhelm it. Aesthetics should align with the project "vibe" defined by the client/PO.
*   **Forgiveness (Error Tolerance):** Help users avoid errors and recover from them easily (e.g., clear error messages, undo options).

## 3. Visual Identity (Branding - to be defined with client/PO)

*   **3.1. Logo:**
    *   [Space for primary logo and its variations. AgilePheromind AI agent can suggest a placeholder or assist in generating one via an MCP if requested.]
*   **3.2. Primary Color Palette (Tailwind CSS Tokens):**
    *   `primary`: [Main brand color, e.g., `colors.blue['500']` - HEX: `#3B82F6`] - Used for main actions, important accents.
    *   `secondary`: [Secondary brand color, e.g., `colors.slate['600']` - HEX: `#475569`] - Used for supporting elements, secondary text.
    *   `accent`: [Accent color, e.g., `colors.amber['400']` - HEX: `#FBBF24`] - Used for highlights, notifications, badges.
    *   **Neutrals:**
        *   `background`: [Main background color, e.g., `colors.white` or `colors.slate['50']`]
        *   `foreground`: [Main text color on `background`, e.g., `colors.slate['900']` or `colors.slate['700']`]
        *   `card / surface`: [Color for containers, cards, e.g., `colors.white` or `colors.slate['100']`]
        *   `border / divider`: [Color for borders, dividers, e.g., `colors.slate['200']` or `colors.slate['300']`]
    *   **Semantic Colors:**
        *   `destructive / error`: [Color for destructive actions, errors, e.g., `colors.red['500']` - HEX: `#EF4444`]
        *   `success`: [Color for confirmations, success, e.g., `colors.green['500']` - HEX: `#22C55E`]
        *   `warning`: [Color for warnings, e.g., `colors.yellow['500']` - HEX: `#EAB308`]
        *   `info`: [Color for informational messages, e.g., `colors.sky['500']` - HEX: `#0EA5E9`]
    *   *Note: These colors must be configured in `tailwind.config.js`.*
*   **3.3. Typography (Tailwind CSS Tokens):**
    *   **Primary Font (Body & UI):** [Font Name, e.g., "Inter", "Roboto", "System UI"]. Specify weights to use (e.g., Regular 400, Medium 500, Semibold 600, Bold 700).
        *   Fallback: `sans-serif`.
    *   **Display Font (Headings, optional):** [Font Name, e.g., "Lexend", "Poppins"]. Specify weights.
    *   **Typographic Scale (Tailwind classes or `theme.fontSize` config):**
        *   `text-xs`, `text-sm`, `text-base` (body), `text-lg`, `text-xl`, `text-2xl`, `text-3xl`, `text-4xl` (Hx headings).
        *   Define appropriate `line-height` for each size.
*   **3.4. Iconography:**
    *   **Style:** [Choose a style, e.g., Outline, Solid, Two-tone].
    *   **Recommended Library:** [e.g., Heroicons, Lucide Icons, Font Awesome via SVG]. Ensure consistency.
    *   Base icon size: [e.g., 20px, 24px].
*   **3.5. Spacing and Grid (Tailwind CSS Tokens):**
    *   **Base Spacing Unit:** [e.g., 4px]. Tailwind's `spacing` scale will be based on this (e.g., `1` -> `0.25rem`, `2` -> `0.5rem`, `4` -> `1rem`).
    *   **Layout Grid:** Use Tailwind's grid classes (`grid`, `grid-cols-*`) or Flexbox (`flex`) for main layouts.
    *   Consistent margins and paddings for sections, cards, form elements.
*   **3.6. Corner Radius (Border Radius - Tailwind CSS Tokens):**
    *   Define a scale (e.g., `rounded-sm`, `rounded`, `rounded-md`, `rounded-lg`, `rounded-xl`, `rounded-full`).
    *   Consistent use for buttons, cards, inputs.
*   **3.7. Shadows (Box Shadow - Tailwind CSS Tokens):**
    *   Define a subtle shadow scale for elevation (e.g., `shadow-sm`, `shadow`, `shadow-md`, `shadow-lg`).
*   **3.8. Overall Project "Vibe" (Reminder of client/PO keywords):**
    *   [e.g., "Modern and clean", "Professional and reliable", "Playful and engaging"]. These keywords must guide overall aesthetic choices.

## 4. Standard UI Components (Based on Shadcn/UI or chosen library, customized)

*The `@architecture-advisor-agent` or `@developer-agent` (guided by design) should document key components here, their variants, and usage, ensuring they adhere to the customization defined by this document's tokens.*

*   **4.1. Buttons (`<button>`, `<a>` styled):**
    *   Variants: `primary`, `secondary`, `destructive`, `outline`, `ghost`, `link`.
    *   States: `default`, `hover`, `focus`, `active`, `disabled`.
    *   Sizes: `sm`, `default` (md), `lg`, `icon`.
*   **4.2. Form Fields (`<input>`, `<textarea>`, `<select>`):**
    *   Input types: `text`, `email`, `password`, `number`, `date`, etc.
    *   States: `default`, `hover`, `focus`, `disabled`, `error`, `success`.
    *   Labels, help text, error messages.
    *   Checkbox, Radio buttons.
*   **4.3. Cards:**
    *   Base structure (header, content, footer).
    *   Variants (e.g., with image, actions).
*   **4.4. Modals (Dialogs):**
    *   Structure (title, content, actions).
    *   Behavior (closing, focus trapping).
*   **4.5. Notifications (Toasts, Alerts):**
    *   Types: `info`, `success`, `warning`, `error`.
    *   Positioning, display duration.
*   **4.6. Navigation:**
    *   Main navigation bar (header).
    *   Side navigation (sidebar, if applicable).
    *   Breadcrumbs.
    *   Pagination.
    *   Tabs.
*   **4.7. Data Tables:**
    *   Structure, sorting, basic filtering, row actions.
*   **4.8. Avatars, Badges, Tooltips, Dropdowns, Accordions, etc.**
    *   [Document other necessary reusable components.]

## 5. Interaction Principles and Animations

*   **Immediate Feedback:** Every user action must have visual feedback (even subtle).
*   **Smooth Transitions:** Animations and transitions should be smooth (e.g., Tailwind's `duration-200`, `ease-in-out`) and purposeful (guide attention, indicate state change), not distracting.
*   **Loading States:** Use clear loading indicators (spinners, skeletons) for asynchronous operations. Buttons must have loading states.
*   **Micro-interactions:** Subtle animations on hovers, focus, or clicks can enhance the experience if well-dosed.

## 6. Accessibility (A11Y)

*   **Color Contrast:** Adhere to WCAG AA contrast ratios (4.5:1 for normal text, 3:1 for large text). Use verifier tools.
*   **Keyboard Navigation:** All interactive elements must be keyboard accessible and operable. Focus order must be logical.
*   **ARIA (Accessible Rich Internet Applications):** Use ARIA attributes appropriately to enhance semantics of complex components (modals, tabs, dropdowns).
*   **Alternative Text for Images:** All informative images must have a descriptive `alt` attribute. Decorative images can have `alt=""`.
*   **Labels for Forms:** All form fields must have associated labels (`<label for="...">`).
*   **Semantic HTML:** Use HTML tags semantically (`<nav>`, `<main>`, `<aside>`, `<article>`, `<button>`, etc.).
*   **Accessibility Testing:** Use automated tools (e.g., Axe DevTools) and perform manual testing (keyboard navigation, basic screen reader).

## 7. Responsive Design

*   **Mobile-First Approach:** Design for small screens first, then adapt for larger ones.
*   **Tailwind CSS Breakpoints:** Use Tailwind's default breakpoints (`sm`, `md`, `lg`, `xl`, `2xl`) consistently.
*   Test on real or simulated screen sizes and devices.
*   Ensure readability and ease of interaction on all screen sizes.

## 8. Tools and Resources

*   **UI Library:** [Name of base library if other than pure Tailwind/Shadcn, e.g., Angular Material styled with Tailwind].
*   **Icon Library:** [Name and link, e.g., Lucide Icons].
*   **Design Tool (if used for mockups):** [e.g., Figma, Penpot]. Provide links to master mockups.
*   **Contrast Checker:** [Link to tool, e.g., WebAIM Contrast Checker].
*   **Browser Accessibility Extension:** [e.g., Axe DevTools].

## 9. Update Process

*   This is a living document. Proposals for new components or changes to existing conventions must be discussed with the Tech Lead / Design Lead.
*   The `@architecture-advisor-agent` is responsible for keeping this document up-to-date after changes are validated.

---