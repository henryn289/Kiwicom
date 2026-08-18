# CLAUDE.md

Guidance for Claude Code and other agents working in this repository.

## What this repository is

Kiwicom was the final group project of the Dev Academy Aotearoa Ngahuru 2026 cohort, built by four people over two sprints in June and July 2026.

**This copy is Henry's fork, kept as portfolio evidence.** The team owns the original at [Oscar-Wakefield/Kiwicom](https://github.com/Oscar-Wakefield/Kiwicom). All issues, pull requests and review history live there, not here.

Never describe this project as solo work. The README's attribution block is deliberate and must not be softened or removed.

## Git

- **The default branch is `dev`, not `main`.** README changes must land on `dev` or they will not render.
- `origin` is `henryn289/Kiwicom`. `upstream` is the team's repository, fetch only; its push URL is deliberately set to `DISABLED`.
- **Do not run "Sync fork" or merge `upstream/dev`.** Both sides have rewritten `README.md` independently, so it will conflict, and the upstream commits contain nothing this fork needs. Being behind upstream is the intended state.
- **When opening a pull request, check the base repository.** GitHub defaults the base of a PR from a fork to the *upstream* repo. It must read `henryn289/Kiwicom` on both sides. Disabling the upstream push URL does not prevent this; a pull request is not a push.
- Feature branches must be merged back into `dev` to be visible.

## Architecture

- **Data access is Knex and Express only.** `supabase-js` is used for authentication, never for queries. Do not introduce Supabase client queries.
- Supabase provides hosted Postgres and GitHub OAuth. Nothing else.
- Production build: Vite builds the client, esbuild bundles the server, Express serves both when `NODE_ENV=production`.
- The GitHub repository autofill uses the *signed in user's own* OAuth token, passed from the browser. There is no server side `GITHUB_TOKEN` and none is needed.
- The AI project summary feature was scaffolded only: `ai_summary` and `ai_summary_at` columns exist and `ProjectPage.tsx` reads them, but no Anthropic client or summary endpoint was ever written. No Anthropic key is required to run this app.

## Environment

`.env.example` is incomplete. It carries only the two client variables. The server also needs `DATABASE_URL` or it will refuse to start:

```
VITE_SUPABASE_URL=
VITE_SUPABASE_ANON_KEY=
DATABASE_URL=
```

The knexfile is at `server/db/knexfile.js`, not the project root, so knex commands need the path:

```bash
npx knex migrate:latest --knexfile server/db/knexfile.js
npx knex seed:run --knexfile server/db/knexfile.js
```

Verified working up to the database connection. Migrations and seeds have never been run against a live Supabase project.

## Working style

- This was a one week bootcamp build. Keep suggestions simple and incremental. Do not over-engineer or propose rewrites.
- Ask Henry to reason through a problem before supplying a solution. He is learning, and prefers hints and questions to finished answers unless he asks directly.
- Flag vague or inaccurate commit messages.

## Documentation accuracy

The README makes specific claims about what Henry built and what he did not. Every one was verified against the code, not inherited from the team's original planning document, which was wrong in several places.

If you change the README:

- Verify claims by reading the implementation, not by inferring from file names, route definitions or component names.
- Keep completed, in progress and planned work clearly separated.
- **No em-dashes.** Minimise hyphens in compound modifiers: "open source", "one week", "full stack", "autofill".
- Do not restore the "shadcn/ui" claim. It was planned and never installed. The two components in `client/components/ui/` came from 21st.dev.
- The debounce in `CreateProject.tsx` is Henry's own `useRef` and `setTimeout`. The `useDebounce` hook is Oscar's and powers the home page search. Do not conflate them.
- Merge and review work is described generically on purpose. Never phrase it as "most" or "the majority" of the team's pull requests.

## Longer context

`10 Projects/Kiwicom.md` in Henry's Obsidian vault holds the full project record, including provenance, verification notes and open threads. Ask him to grant access with `--add-dir` if you need it.
