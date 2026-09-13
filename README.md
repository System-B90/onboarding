# @system-b90/onboarding

An in-house guided-tour and help-panel engine for MUI apps: an anchor registry,
a step runner with a spotlight overlay, and a persistent help drawer.

**Generic by construction.** Nothing here knows what a curriculum, a syllabus or
a cut is. It imports `react` and `@mui/material` only, extracted from Bluz's
`ui/src/components/onboarding/` the same way
[`@system-b90/command-palette`](https://github.com/System-B90/command-palette)
was extracted before it.

## Install

```bash
npm install @system-b90/onboarding
```

## Quick Start

```tsx
// 1. Mount the provider once, above everything that contributes a tour or an
//    anchor, with the language you want.
import { OnboardingProvider } from "@system-b90/onboarding";
import { HE_LABELS } from "@system-b90/onboarding/he";

<OnboardingProvider labels={HE_LABELS} storageNamespace="myapp">
    {children}
</OnboardingProvider>;
```

```tsx
// 2. Mark the things a tour points at.
import { useTourAnchor } from "@system-b90/onboarding";

<Tabs ref={useTourAnchor("gantt.tabs")} />;
```

```tsx
// 3. Contribute the tour from the component that owns that screen.
import { useTour } from "@system-b90/onboarding";

const TOUR = {
    id: "gantt.intro",
    title: "סיור בגאנט",
    autoStart: true,
    steps: [
        {
            id: "tabs",
            title: "חמש תצוגות",
            body: "כל לשונית היא זווית אחרת על אותו גאנט.",
            anchor: "gantt.tabs",
            placement: "block-end",
        },
    ],
} as const satisfies Tour;

useTour(TOUR);
```

```tsx
// 4. Give returning users a way back in.
import { HelpButton } from "@system-b90/onboarding";
```

## The model

| Concept | What it is |
| --- | --- |
| **Anchor** | A stable id (`"gantt.tabs"`) mapped to a live element by `useTourAnchor`. Steps never hold selectors. |
| **Step** | Title, body, an optional anchor, a logical placement, and optional `beforeShow` / `optional` / `interactive` flags. |
| **Tour** | An ordered list of steps behind a stable `id`, registered by `useTour` and persisted by that id. |
| **Topic** | An entry in the help drawer, registered by `useHelpTopics`, optionally offering to replay a tour. |

Steps resolve at run time, one at a time:

1. `beforeShow` runs and is awaited — this is where a tour opens a drawer or
   switches to the tab holding the next anchor.
2. The runner waits (up to `anchorTimeoutMs`, 4s) for the anchor to register.
3. If it never arrives: an `optional` step is skipped, any other step degrades
   to a centred, anchor-less card rather than stalling the tour.

That is why a tour can safely describe UI that only exists in some states, and
why a step is never left pointing at nothing.

## Persistence

`storageNamespace` prefixes one `localStorage` key holding, per tour id, the
version seen and whether it was completed or dismissed. Dismissing counts as
seen — a tour the user walked out of does not ambush them next session.

- Rewrote a tour? Bump `Tour.version` and everyone sees it again.
- Need "seen" to follow the user across devices? Pass a `storage` prop backed by
  your own API — the seam is the whole of `OnboardingStorage`.

## Theme, RTL and accessibility

- Surfaces are MUI (`Paper`, `Popper`, `Drawer`, palette colors), so light/dark
  and the host's theme tokens apply with no styling of our own to keep in sync.
  The backdrop dim is derived from the palette rather than a fixed slate.
- Placements are **logical** (`inline-start`, `block-end`) and resolved against
  `theme.direction`, so a Hebrew UI needs no per-step overrides.
- The card is a `role="dialog"` labelled by its title. Focus moves into it on
  every step, `Tab` cycles inside it, and focus returns to where the user was
  when the tour ends. Each step is announced through a polite live region.
- `Esc` dismisses, `Enter` advances, and the arrow keys move in the reading
  direction of the theme.
- **Ending a tour is deliberate**: `Esc`, the close button or "skip". A click on
  the backdrop does nothing — a dismissal is remembered, so a misclick must not
  be able to spend it.
- The spotlight is four backdrop panes around a real hole, so an `interactive`
  step lets the user press the control being described — the step says so, the
  ring pulses, and the control joins the card's focus cycle.
- Below `sm` the card becomes a bottom sheet: a popper wide enough to read would
  cover the very thing it points at on a phone.
- `prefers-reduced-motion` turns off the pane transitions, the ring pulse, the
  card fade and smooth scrolling.

## Language

Two tables ship, one per subpath — import exactly one:

```tsx
import { EN_LABELS } from "@system-b90/onboarding/en";
import { HE_LABELS } from "@system-b90/onboarding/he";
```

Re-word individual strings with `withLabelOverrides(HE_LABELS, { … })` rather
than restating a table. A third language is just an `OnboardingLabels` object.
