# tesa-shared-components

Shared UI components for the eight TeSA Cohort 6 capstone products. This repo is not deployed: copy the components you need into `/src/components/ui/` in your own project, then override `/src/styles/tokens.css` with your team's look.

> README owner: Ufoma. To fill in once components exist: a table of every component with its owner, props and one usage example; how to copy a component in; how to override tokens.

## Working rules

- Work on a branch, open a pull request, get one other interface engineer to review it.
- CI runs `npm run lint` and `npm run typecheck` on every pull request. Both must pass.
- No `any`. No hard-coded colours; use the variables in `tokens.css`.

## Setup

```
npm install
npm run lint
npm run typecheck
```
