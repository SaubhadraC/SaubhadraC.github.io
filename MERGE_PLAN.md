# Merge Plan: Upstream V1 → This Repo's V1

> **Upstream:** [`samyaroy/FolioForge-vue-portfolio-website`](https://github.com/samyaroy/FolioForge-vue-portfolio-website) branch `V1`
> **Target:** [`saubhadrac/saubhadrac.github.io`](https://github.com/saubhadrac/saubhadrac.github.io) branch `V1`
> **Generated:** 2026-03-15

---

## Overview

This document outlines a plan to merge **logical (code/structural) changes** from the upstream `V1` branch into this repository's `V1` branch, while **excluding personal data changes** (profile content, images, domain names, etc.).

A full `git diff origin/V1 upstream/V1` reveals **49 changed files** spanning both code logic and personal data. This plan categorizes every change and provides a file-by-file merge strategy.

---

## Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Change Categories](#2-change-categories)
3. [Logical Changes to Merge](#3-logical-changes-to-merge)
4. [Personal / Data Changes to Skip](#4-personal--data-changes-to-skip)
5. [Step-by-Step Merge Procedure](#5-step-by-step-merge-procedure)
6. [Risk Assessment & Conflict Areas](#6-risk-assessment--conflict-areas)
7. [Post-Merge Verification](#7-post-merge-verification)

---

## 1. Prerequisites

- [ ] Add upstream remote: `git remote add upstream https://github.com/samyaroy/FolioForge-vue-portfolio-website.git`
- [ ] Fetch upstream branches: `git fetch upstream V1`
- [ ] Create a working branch from this repo's V1: `git checkout -b merge/upstream-v1-logical origin/V1`

---

## 2. Change Categories

| Category | Description | Action |
|---|---|---|
| **Bug Fix** | Missing conditional guards, broken rendering | Cherry-pick / apply |
| **Feature** | New capabilities (feature flags, Credly, TypeScript) | Cherry-pick / apply |
| **Refactor** | Component restructuring, code organisation | Cherry-pick / apply |
| **Formatting** | Whitespace, indentation, trailing newlines | Apply if trivial |
| **Personal Data** | Profile content, images, domains, names | **Skip** |

---

## 3. Logical Changes to Merge

### 3.1 Bug Fixes

#### 3.1.1 Missing `v-if` guard on Google Scholar link in Footer

- **Files:** `src/components/Footer/Index.vue`, `src/components/Footer/index.vue`
- **Issue:** The Google Scholar `<li>` renders even when no URL is provided, showing a broken/empty link.
- **Fix:** Add `v-if="google_scholar"` to the `<li>` element.
- **Priority:** High

```diff
- <li>
+ <li v-if="google_scholar">
    <a :href="google_scholar" target="_blank">Google Scholar</a>
  </li>
```

#### 3.1.2 Whitespace bug in Contact page class attribute

- **File:** `src/views/Contact.vue`
- **Issue:** `class ="flex` has a stray space before `=`, which although functional, is inconsistent.
- **Fix:** Change `class ="flex` → `class="flex`.
- **Priority:** Low

```diff
- <div v-if="github" class ="flex items-center gap-4 ...">
+ <div v-if="github" class="flex items-center gap-4 ...">
```

### 3.2 New Features

#### 3.2.1 Feature Flags for Home Page Sections

- **Files:** `src/config/featureFlags.js`, `src/views/Home/index.vue`
- **Description:** Adds a `showHome` section to the feature flags config, allowing each Home page section (HeroSection, ResearchInterests, Experience, Education, Awards) to be toggled independently.
- **Priority:** High

Changes in `src/config/featureFlags.js`:
```diff
  const DEFAULT_FEATURE_FLAGS = Object.freeze({
+   showHome: {
+     showHeroSection: true,
+     showResearchInterests: true,
+     showExperience: true,
+     showEducation: true,
+     showAwards: false,
+   },
+
    showProjectsPublications: {
```

Changes in `src/views/Home/index.vue`:
```diff
- <HeroSection />
+ <HeroSection v-if="homeFlags.showHeroSection" />
  <!-- (same for ResearchInterests, Experience, Education, Awards) -->
```

Plus the corresponding `<script setup>` import and flag resolution.

> **Note:** Do **not** change the default flag values for existing sections (e.g. `showArticles`, `showOngoingProjects`, etc.) — those reflect the upstream author's personal preferences, not template defaults. Only merge the **new `showHome` structure** and set values appropriate for this repo.

#### 3.2.2 Credly Badge Display in InternshipCertification Page

- **Files:** `src/views/InternshipCertification/Index.vue`, `src/views/InternshipCertification/index.vue`
- **Description:** Adds a conditional Credly badge section above certifications, reading from `config.socials.credly`.
- **Priority:** Medium

```diff
+ <div v-if="credly" class="max-w-4xl mx-auto mb-6">
+   <div class="bg-white rounded-lg shadow-md flex items-center overflow-hidden">
+     <div class="w-[20%] flex items-start justify-center ...">
+       <img src="/icons/Credly.png" alt="Credly logo" ... />
+     </div>
+     <div class="w-[80%] py-2 pl-6 pr-6">
+       <p class="text-sm text-[#4e7397]">
+         View my verified badges on
+         <a :href="credly" target="_blank" rel="noopener noreferrer" ...>Credly</a>
+       </p>
+     </div>
+   </div>
+ </div>
```

And in script:
```diff
+ const credly = config.socials.credly
```

> **Note:** Ensure `src/profile_info.yml` has a `socials.credly` field (can be empty/null to hide the section). A Credly icon image (`/icons/Credly.png`) would also need to be added to `public/icons/` if this feature is wanted.

#### 3.2.3 SmartLink TypeScript Conversion

- **File:** `src/components/SmartLink.vue`
- **Description:** Converts the `<script setup>` block to TypeScript with `lang="ts"`, adds a `LinkItem` interface, and uses typed `defineProps`.
- **Priority:** Low (non-breaking improvement)

```diff
- <script setup>
+ <script setup lang="ts">
  import { computed } from 'vue'
  import links from '@/metadata/hyperlinkMetadata.yml'

+ interface LinkItem {
+     Name?: string
+     Website?: string
+     Link?: string
+ }

- const props = defineProps({
-     text: { type: String, required: true },
-     type: { type: String, default: 'Institute' },
-     href: { type: String, default: null }
- })
+ const props = defineProps<{
+     text: string
+     type?: string
+     href?: string | null
+ }>()

- const resolvedUrl = computed(() => {
-     const candidates = links[props.type]
+ const resolvedUrl = computed<string | null>(() => {
+     const candidates = (links as Record<string, LinkItem[]>)[props.type ?? 'Institute']
```

### 3.3 Refactoring

#### 3.3.1 Cocurricular Leadership Component Restructuring

- **Files:** `src/views/Cocurricular/Index.vue`, `src/views/Cocurricular/index.vue`, `src/views/Cocurricular/components/Leadership.vue`
- **Description:** The `<Leadership>` component previously contained its own wrapper (`<div class="bg-white rounded-lg shadow-sm p-8">`) and section header. The refactoring moves the wrapper and header into the parent `Index.vue`, and `Leadership.vue` now renders only the individual entry (border-left card). This enables proper `v-for` rendering of multiple leadership entries inside a single container.
- **Priority:** Medium

**Parent (`Index.vue`):** Wrap `<Leadership>` in a container with the section header:
```diff
- <Leadership v-if="showLeadershipSection"
-   v-for="(leadership, index) in leadershipRoles"
-   :key="index" :leadership="leadership" />
+ <div v-if="showLeadershipSection" class="bg-white rounded-lg shadow-sm p-8">
+   <h2 class="text-2xl font-bold ...">
+     <svg ...>...</svg>
+     Leadership & Organizations
+   </h2>
+   <Leadership v-for="(leadership, index) in leadershipRoles"
+     :key="index" :leadership="leadership" />
+ </div>
```

**Child (`Leadership.vue`):** Remove the outer wrapper and header, keep only the entry template.

#### 3.3.2 Volunteering Component Null-Safety Improvements

- **File:** `src/views/Cocurricular/components/Volunteering.vue`
- **Description:** Adds a computed `fieldEntries` property that filters out null/empty field array entries. Adds `v-if` guards and uses `String()` wrapping for safe `.split()` calls.
- **Priority:** Medium

Key changes:
```diff
+ import { computed } from 'vue'
- defineProps({ ... })
+ const props = defineProps({ ... })
+
+ const fieldEntries = computed(() => {
+   if (!Array.isArray(props.volunteering.field)) return []
+   return props.volunteering.field.filter((entry) => {
+     if (!entry) return false
+     return Boolean(entry['sub-field'] || entry.time_period)
+   })
+ })
```

And in the template, use `fieldEntries` instead of `volunteering.field`, plus `v-if="f.time_period"` guard.

### 3.4 UI/Styling

#### 3.4.1 Header Profile Icon Sizing and Rounding

- **File:** `src/components/Header.vue`
- **Description:** Changes the profile icon container from `size-7` to `size-4` and adds `rounded-full` for a circular appearance.
- **Priority:** Low (cosmetic)

```diff
- <div class="size-7">
-   <img src="/profile-icon.png" alt="Profile Icon" class="w-full h-full object-cover" />
+ <div class="size-4">
+   <img src="/profile-icon.png" alt="Profile Icon" class="w-full h-full object-cover rounded-full" />
```

### 3.5 Formatting / Minor Cleanup

These are non-functional formatting changes. They can be applied optionally for consistency.

| File | Change |
|---|---|
| `src/views/Home/components/experience/Index.vue` | Remove trailing blank line before closing `</div>` |
| `src/views/Home/components/experience/index.vue` | Same as above |
| `src/views/Cocurricular/Index.vue` | Fix indentation of "No volunteering roles" `<div>` |
| `src/views/InternshipCertification/Index.vue` | Reformat multi-line `v-for` button binding |

---

## 4. Personal / Data Changes to Skip

The following changes are specific to the upstream author's personal data and **must NOT be merged** into this repo:

| File(s) | Reason to Skip |
|---|---|
| `.env` | Contains `VITE_USER_NAME` (upstream: "Samyabrata Roy") |
| `CNAME`, `public/CNAME` | Domain-specific (upstream: codeium.xyz) |
| `public/SamyabrataRoy2.jpg`, `public/SaubhadraC.jpg` | Personal photos |
| `public/profile-icon.png` | Personal profile icon |
| `public/logo/*` (IEM, ISBR, WBSETCL vs IITM, SNU, IDEAS, MSRKAV) | Institution-specific logos |
| `public/icons/Credly.png`, `credly.png` | Only needed if Credly feature is adopted |
| `public/googlef62b25008a7b041d.html` | Google site verification (personal) |
| `public/robots.txt` | References upstream's sitemap URL |
| `src/profile_info.yml` | All personal profile data |
| `src/metadata/hyperlinkMetadata.yml` | Personal hyperlink metadata |
| `src/metadata/logo/*` | Personal institution logos |
| `src/metadata/people/profile-icon.png` | Personal icon |
| `src/views/Home/components/HeroSection.vue` | Personal photo reference |
| `src/components/Footer/index.vue` (copyright name, logos array) | Personal name and logo references |
| `src/views/Home/components/researchInterests/*` (icon removals) | Specific to upstream author's research interests |
| `src/config/featureFlags.js` (default value changes) | Only merge the **structure** (`showHome`), not the toggled defaults |
| `vite.config.js` (hostname, sitemap options) | Domain-specific configuration |
| `BUGFIXES.md` | Upstream's internal documentation about their own PR |
| Flag Counter uncommenting in Footer | The Flag Counter widget is tied to a specific tracking ID |

---

## 5. Step-by-Step Merge Procedure

Since a direct `git merge` would bring in all personal data changes and create extensive conflicts, a **selective cherry-pick / manual patch** approach is recommended.

### Step 1: Set Up Working Branch

```bash
git checkout V1
git checkout -b merge/upstream-v1-logical
git remote add upstream https://github.com/samyaroy/FolioForge-vue-portfolio-website.git
git fetch upstream V1
```

### Step 2: Apply Bug Fixes

- [ ] **Footer Google Scholar guard:** Edit `src/components/Footer/Index.vue` and `src/components/Footer/index.vue` — add `v-if="google_scholar"` to the Google Scholar `<li>` element
- [ ] **Contact whitespace fix:** Edit `src/views/Contact.vue` — fix `class ="flex` → `class="flex"`

### Step 3: Apply Feature — Home Page Feature Flags

- [ ] Edit `src/config/featureFlags.js` — add the `showHome` section at the top of `DEFAULT_FEATURE_FLAGS` (keep existing this-repo default values for other flags unchanged)
- [ ] Edit `src/views/Home/index.vue` — import `isFeatureEnabled`, create `homeFlags` object, add `v-if` bindings to each section component

### Step 4: Apply Feature — Credly Badge (Optional)

This feature requires a Credly URL in `profile_info.yml` and a Credly icon image. If desired:

- [ ] Add `credly` field under `socials` in `src/profile_info.yml` (set to your Credly URL or leave empty)
- [ ] Add `public/icons/Credly.png` image asset
- [ ] Edit `src/views/InternshipCertification/Index.vue` and `index.vue` — add the Credly badge section and `const credly = config.socials.credly`

### Step 5: Apply Feature — SmartLink TypeScript (Optional)

- [ ] Edit `src/components/SmartLink.vue` — convert to TypeScript with typed props and interface

### Step 6: Apply Refactoring — Cocurricular Leadership

- [ ] Edit `src/views/Cocurricular/components/Leadership.vue` — remove outer wrapper div and section header
- [ ] Edit `src/views/Cocurricular/Index.vue` and `index.vue` — add the wrapper div and section header around the `v-for` loop

### Step 7: Apply Refactoring — Volunteering Null-Safety

- [ ] Edit `src/views/Cocurricular/components/Volunteering.vue` — add `computed` import, convert to `const props`, add `fieldEntries` computed, update template to use `fieldEntries` with guards

### Step 8: Apply UI Change — Header Icon

- [ ] Edit `src/components/Header.vue` — change `size-7` to `size-4`, add `rounded-full` to img

### Step 9: Apply Formatting (Optional)

- [ ] Minor indentation fixes in `Index.vue` / `index.vue` files as listed in Section 3.5

### Step 10: Test & Verify

- [ ] Run `npm run build` to confirm no build errors
- [ ] Run `npm run dev` and manually verify each page
- [ ] Confirm feature flags work (toggle values and check rendering)
- [ ] Confirm null-safety improvements (remove data fields and verify no crashes)

### Step 11: Merge to V1

```bash
git checkout V1
git merge merge/upstream-v1-logical
git push origin V1
```

---

## 6. Risk Assessment & Conflict Areas

| Risk | Likelihood | Mitigation |
|---|---|---|
| Feature flags break existing sections | Low | New `showHome` flags default to `true` for all existing sections |
| Leadership refactoring breaks layout | Medium | Test with multiple leadership entries; verify single-container rendering |
| Volunteering `fieldEntries` filters out valid data | Low | Test with edge cases (empty arrays, null fields) |
| SmartLink TS conversion causes build errors | Low | Verify TypeScript support in Vite config; test with existing links |
| Credly section without icon causes broken image | Medium | Only add if `Credly.png` asset is available |
| Duplicate `Index.vue` / `index.vue` files diverge | High | Both files in each directory should receive identical changes |

---

## 7. Post-Merge Verification

- [ ] `npm run build` succeeds without errors
- [ ] `npm run lint` passes
- [ ] All pages render correctly on desktop and mobile
- [ ] Feature flags can toggle Home page sections on/off
- [ ] Google Scholar link in Footer only shows when URL is configured
- [ ] Volunteering section handles missing/null field entries gracefully
- [ ] Leadership entries render inside a single container card
- [ ] Profile icon in Header appears smaller and circular
- [ ] No upstream personal data (names, images, domains) leaked into this repo
