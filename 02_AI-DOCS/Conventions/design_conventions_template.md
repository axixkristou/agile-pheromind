# Conventions de Design et UI/UX du Projet : {{NOM_PROJET_PAR_PHEROMIND}}

**Version:** 1.0
**Date de Dernière Mise à Jour:** {{TIMESTAMP_PAR_PHEROMIND}}
**Maintenu par:** `@architecture-advisor-agent` (avec validation humaine du Tech Lead/Design Lead)

## 1. Introduction

Ce document définit les principes, standards et composants de design à utiliser pour le projet `{{NOM_PROJET_PAR_PHEROMIND}}`. L'objectif est de créer une expérience utilisateur (UX) cohérente, intuitive, accessible, et esthétiquement plaisante, alignée avec l'identité de la marque et les attentes des utilisateurs. Ces conventions guideront le développement frontend (Angular) et l'utilisation de la librairie de style (Tailwind CSS) et de composants (ex: sur base de Shadcn/UI si retenu).

Le respect de ces conventions est crucial pour maintenir une haute qualité UI/UX.

## 2. Philosophie et Principes de Design UX/UI

*   **Centré sur l'Utilisateur (User-Centered Design):** Les besoins et les objectifs des utilisateurs finaux sont au cœur de toutes les décisions de design.
*   **Simplicité et Clarté:** "Moins, c'est plus." Interfaces épurées, informations hiérarchisées, langage clair. Éviter la surcharge cognitive.
*   **Cohérence (Consistency):** Les éléments UI, les interactions et les parcours utilisateurs doivent être cohérents à travers toute l'application. Utiliser les composants et styles définis ici.
*   **Intuitivité (Intuitive Design):** L'interface doit être facile à comprendre et à utiliser sans nécessiter d'explications complexes. Les utilisateurs doivent pouvoir anticiper le résultat de leurs actions.
*   **Feedback Utilisateur (Feedback):** Fournir un retour clair et immédiat aux actions de l'utilisateur (ex: états de chargement, messages de succès/erreur, confirmations visuelles).
*   **Efficacité (Efficiency):** Permettre aux utilisateurs d'accomplir leurs tâches rapidement et avec un minimum d'effort. Optimiser les flux de travail.
*   **Accessibilité (A11Y):** Concevoir pour tous. Viser au minimum la conformité WCAG 2.1 AA. (Voir section dédiée).
*   **Esthétique Fonctionnelle (Aesthetic Integrity):** Le design visuel doit soutenir et améliorer la fonctionnalité, sans la surcharger. L'esthétique doit être alignée avec la "vibe" du projet définie par le client/PO.
*   **Tolérance aux Erreurs (Forgiveness):** Aider les utilisateurs à éviter les erreurs et à les corriger facilement (ex: messages d'erreur clairs, options d'annulation).

## 3. Identité Visuelle (Branding - à définir avec le client/PO)

*   **3.1. Logo:**
    *   [Emplacement pour le logo principal et ses variations. L'agent IA peut suggérer un placeholder ou aider à en générer un via un MCP si demandé.]
*   **3.2. Palette de Couleurs Principale (Tokens Tailwind CSS):**
    *   `primary`: [Couleur principale, ex: `colors.blue['500']` - HEX: `#3B82F6`] - Utilisée pour les actions principales, les accents importants.
    *   `secondary`: [Couleur secondaire, ex: `colors.slate['600']` - HEX: `#475569`] - Utilisée pour les éléments de support, textes secondaires.
    *   `accent`: [Couleur d'accent, ex: `colors.amber['400']` - HEX: `#FBBF24`] - Utilisée pour les points d'attention, notifications, badges.
    *   **Neutres:**
        *   `background`: [Couleur de fond principale, ex: `colors.white` ou `colors.slate['50']`]
        *   `foreground`: [Couleur de texte principale sur `background`, ex: `colors.slate['900']` ou `colors.slate['700']`]
        *   `card / surface`: [Couleur pour les conteneurs, cartes, ex: `colors.white` ou `colors.slate['100']`]
        *   `border / divider`: [Couleur pour les bordures, séparateurs, ex: `colors.slate['200']` ou `colors.slate['300']`]
    *   **Sémantiques:**
        *   `destructive / error`: [Couleur pour les actions destructrices, erreurs, ex: `colors.red['500']` - HEX: `#EF4444`]
        *   `success`: [Couleur pour les confirmations, succès, ex: `colors.green['500']` - HEX: `#22C55E`]
        *   `warning`: [Couleur pour les avertissements, ex: `colors.yellow['500']` - HEX: `#EAB308`]
        *   `info`: [Couleur pour les messages informatifs, ex: `colors.sky['500']` - HEX: `#0EA5E9`]
    *   *Note: Ces couleurs doivent être configurées dans `tailwind.config.js`.*
*   **3.3. Typographie (Tokens Tailwind CSS):**
    *   **Police Principale (Body & UI):** [Nom de la police, ex: "Inter", "Roboto", "System UI"]. Spécifier les graisses à utiliser (ex: Regular 400, Medium 500, Semibold 600, Bold 700).
        *   Fallback: `sans-serif`.
    *   **Police d'Affichage (Titres, optionnel):** [Nom de la police, ex: "Lexend", "Poppins"]. Spécifier les graisses.
    *   **Échelle Typographique (Classes Tailwind ou configuration `theme.fontSize`):**
        *   `text-xs`, `text-sm`, `text-base` (corps), `text-lg`, `text-xl`, `text-2xl`, `text-3xl`, `text-4xl` (titres Hx).
        *   Définir les `line-height` appropriés pour chaque taille.
*   **3.4. Iconographie:**
    *   **Style:** [Choisir un style, ex: Outline, Solid, Two-tone].
    *   **Librairie Recommandée:** [ex: Heroicons, Lucide Icons, Font Awesome via SVG]. S'assurer de la cohérence.
    *   Taille de base des icônes: [ex: 20px, 24px].
*   **3.5. Espacement et Grille (Tokens Tailwind CSS):**
    *   **Unité d'Espacement de Base:** [ex: 4px]. L'échelle d'espacement de Tailwind (`spacing`) sera basée sur cette unité (ex: `1` -> `0.25rem`, `2` -> `0.5rem`, `4` -> `1rem`).
    *   **Grille de Mise en Page:** Utiliser les classes de grille de Tailwind (`grid`, `grid-cols-*`) ou Flexbox (`flex`) pour les mises en page principales.
    *   Marges et Paddings cohérents pour les sections, cartes, éléments de formulaire.
*   **3.6. Arrondis (Border Radius - Tokens Tailwind CSS):**
    *   Définir une échelle d'arrondis (ex: `rounded-sm`, `rounded`, `rounded-md`, `rounded-lg`, `rounded-xl`, `rounded-full`).
    *   Utilisation cohérente pour les boutons, cartes, inputs.
*   **3.7. Ombres (Box Shadow - Tokens Tailwind CSS):**
    *   Définir une échelle d'ombres subtiles pour l'élévation (ex: `shadow-sm`, `shadow`, `shadow-md`, `shadow-lg`).
*   **3.8. "Vibe" Générale du Projet (Rappel des mots-clés du client/PO):**
    *   [Ex: "Moderne et épuré", "Professionnel et fiable", "Ludique et engageant"]. Ces mots-clés doivent guider les choix esthétiques globaux.

## 4. Composants UI Standards (Basés sur Shadcn/UI ou librairie choisie, personnalisés)

*L'agent `@architecture-advisor-agent` ou `@developer-agent` (guidé par le design) doit documenter ici les composants clés, leurs variantes, et leur utilisation, en s'assurant qu'ils respectent la personnalisation définie par les tokens de ce document.*

*   **4.1. Boutons (`<button>`, `<a>` stylés):**
    *   Variantes: `primary`, `secondary`, `destructive`, `outline`, `ghost`, `link`.
    *   États: `default`, `hover`, `focus`, `active`, `disabled`.
    *   Tailles: `sm`, `default` (md), `lg`, `icon`.
*   **4.2. Champs de Formulaire (`<input>`, `<textarea>`, `<select>`):**
    *   Types d'input: `text`, `email`, `password`, `number`, `date`, etc.
    *   États: `default`, `hover`, `focus`, `disabled`, `error`, `success`.
    *   Labels, messages d'aide, messages d'erreur.
    *   Checkbox, Radio buttons.
*   **4.3. Cartes (Cards):**
    *   Structure de base (header, content, footer).
    *   Variantes (ex: avec image, actions).
*   **4.4. Modales (Dialogs):**
    *   Structure (titre, contenu, actions).
    *   Comportement (fermeture, focus trapping).
*   **4.5. Notifications (Toasts, Alerts):**
    *   Types: `info`, `success`, `warning`, `error`.
    *   Positionnement, durée d'affichage.
*   **4.6. Navigation:**
    *   Barre de navigation principale (header).
    *   Navigation latérale (sidebar, si applicable).
    *   Fil d'Ariane (Breadcrumbs).
    *   Pagination.
    *   Tabs.
*   **4.7. Tables de Données:**
    *   Structure, tri, filtrage (basique), actions par ligne.
*   **4.8. Avatars, Badges, Tooltips, Dropdowns, Accordions, etc.**
    *   [Documenter les autres composants réutilisables nécessaires.]

## 5. Principes d'Interaction et Animations

*   **Feedback Immédiat:** Toute action utilisateur doit avoir un retour visuel (même subtil).
*   **Transitions Fluides:** Animations et transitions doivent être douces (ex: `duration-200`, `ease-in-out` de Tailwind) et servir à guider l'œil ou à indiquer un changement d'état, sans être distrayantes.
*   **États de Chargement (Loading States):** Utiliser des indicateurs de chargement clairs (spinners, skeletons) pour les opérations asynchrones. Les boutons doivent avoir un état de chargement.
*   **Micro-interactions:** Des animations subtiles sur les hovers, focus, ou clics peuvent améliorer l'expérience si bien dosées.

## 6. Accessibilité (A11Y)

*   **Contraste des Couleurs:** Respecter les ratios de contraste WCAG AA (4.5:1 pour texte normal, 3:1 pour grand texte). Utiliser des outils de vérification.
*   **Navigation au Clavier:** Tous les éléments interactifs doivent être accessibles et opérables au clavier. L'ordre de focus doit être logique.
*   **ARIA (Accessible Rich Internet Applications):** Utiliser les attributs ARIA de manière appropriée pour améliorer la sémantique des composants complexes (modales, onglets, menus déroulants).
*   **Texte Alternatif pour Images:** Toutes les images informatives doivent avoir un attribut `alt` descriptif. Les images décoratives peuvent avoir `alt=""`.
*   **Labels pour Formulaires:** Tous les champs de formulaire doivent avoir des labels associés (`<label for="...">`).
*   **Sémantique HTML:** Utiliser les balises HTML de manière sémantique (`<nav>`, `<main>`, `<aside>`, `<article>`, `<button>`, etc.).
*   **Tests d'Accessibilité:** Utiliser des outils automatisés (ex: Axe DevTools) et effectuer des tests manuels (navigation clavier, lecteur d'écran basique).

## 7. Responsive Design

*   **Approche Mobile-First:** Concevoir d'abord pour les petits écrans, puis adapter pour les plus grands.
*   **Points de Rupture (Breakpoints) Tailwind CSS:** Utiliser les breakpoints par défaut de Tailwind (`sm`, `md`, `lg`, `xl`, `2xl`) de manière cohérente.
*   Tester sur des tailles d'écran et appareils réels ou simulés.
*   Assurer la lisibilité et la facilité d'interaction sur toutes les tailles d'écran.

## 8. Outils et Ressources

*   **Librairie UI:** [Nom de la librairie de base si autre que Tailwind/Shadcn pur, ex: Angular Material mais stylé avec Tailwind].
*   **Librairie d'Icônes:** [Nom et lien, ex: Lucide Icons].
*   **Outil de Design (si utilisé pour mockups):** [Ex: Figma, Penpot]. Fournir des liens vers les maquettes maîtresses.
*   **Vérificateur de Contraste:** [Lien vers un outil, ex: WebAIM Contrast Checker].
*   **Extension d'Accessibilité Navigateur:** [Ex: Axe DevTools].

## 9. Processus de Mise à Jour

*   Ce document est vivant. Les propositions de nouveaux composants ou de modifications aux conventions existantes doivent être discutées avec le Tech Lead / Design Lead.
*   L'agent `@architecture-advisor-agent` est responsable de maintenir ce document à jour après validation des changements.

---