# Decisions

One dated line per decision: what was decided, why, and who agreed.

<!-- PENDING: confirm after the interface engineers' meeting, then remove this comment -->
2026-10-22  Shared component set agreed by the four interface engineers.
Base: React + TypeScript + Tailwind CSS + Radix UI.
Shared via one public copy-in repo: tesa-shared-components (link).
Split: Muizz (setup, tokens, button, form inputs), Ehimare (select, checkbox, filter bar),
Victor (dialog, toast), Ufoma (card, badge, skeleton, empty and error states, README, skill). Due 23 Oct.
Why: brief requires one shared set (p.3); avoids building the same pieces eight times and keeps accessibility consistent.
Agreed by: Muizz Jamiu, Ehimare J. Okosun, Victor Oyeleke, Ufoma Onowho.

2026-10-24  Textarea character count, and --radius as an alias of --radius-md.
Textarea: showCount (with maxLength) shows "120 / 500" under the field as visible text, linked with aria-describedby.
It becomes a polite live region only in the last 20 characters.
--radius: defined in tokens.css as var(--radius-md). Dialog and Toast keep reading var(--radius) unchanged.
Why: announcing every keystroke interrupts screen reader users while they type, and the count only matters near the limit.
Dialog and Toast corners fell back to square because --radius was undefined; an alias fixes them without renaming a token
or editing those components, and products change one value for all corners.
Agreed by: Muizz Jamiu.
Informed: Ehimare J. Okosun, Victor Oyeleke, Ufoma Onowho.
