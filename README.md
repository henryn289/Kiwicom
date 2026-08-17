# Kiwicom

A community hub for developers to discover, share and collaborate on open source projects. Built as the final group project of the Dev Academy Aotearoa Ngahuru 2026 cohort.

> **This is a fork of a team project. I did not build this alone.**
>
> The original repository is [Oscar-Wakefield/Kiwicom](https://github.com/Oscar-Wakefield/Kiwicom), owned by the team. Kiwicom was built by four people over two sprints, 25 June to 1 July 2026. I forked it here with the team's agreement so I could document my own contribution for portfolio purposes. All issues, pull requests and review history live in the original repository, and the team retains ownership of the project.
>
> Full team: Oscar Wakefield (Product Owner), Henry Nguyen (Agile Facilitator), Serina McFall (Git Keeper), Ivonne Valenzuela (Vibes Watcher).

---

## Status

Not deployed. There is no live demo. The project was built to a one week bootcamp deadline and deployment was never completed, so this repository is source code only.

The AI project summary feature described in the original planning documentation was a stretch goal. The schema reserves columns for it (`ai_summary`, `ai_summary_at`) and `ProjectPage.tsx` reads them, but no Anthropic client or summary endpoint was ever written. It was scoped for a third sprint the deadline never allowed.

It is the thing I most want to finish. Now that I have my own copy, building the summary endpoint against the schema the team already designed is the next piece of work on this repository.

---

## What I built

My work concentrated on authentication, the GitHub API integration, and the create project feature.

### Authentication

GitHub OAuth through Supabase, end to end, and mine alone.

- [`client/supabaseClient.ts`](client/supabaseClient.ts) and [`client/hooks/use-auth.ts`](client/hooks/use-auth.ts), the shared `useAuth` hook consumed across the app
- Signing in and out via `supabase.auth.signInWithOAuth({ provider: 'github' })`
- Automatic profile creation on first login: `upsertProfile` fires on the `SIGNED_IN` event so a user record exists from the first session without a separate registration step
- `ProtectedRoute`, plus the fix that stopped it redirecting before the session had finished loading
- The `upsertProfile` database function and `POST /api/v1/profiles` route behind it

### GitHub repository autofill

So a user adding a project does not have to retype what GitHub already knows. Also mine alone.

- [`server/routes/github.ts`](server/routes/github.ts), an Express endpoint that proxies to the GitHub API with Octokit, using the signed in user's own OAuth token rather than a shared application token
- The typeahead on the Add Project form, with debouncing implemented directly in [`client/pages/CreateProject.tsx`](client/pages/CreateProject.tsx) using `useRef` and `setTimeout` / `clearTimeout`
- Selecting a repository populates name, description, language, topics, stars, licence and open issue count in one action

### Create project and ownership

- The `ProjectCard` component
- The Add Project form and its `POST /api/v1/projects` route, plus the `addProject` database function
- The `owner_profile_id` migration and `getProjectsByOwner`, which is what makes a project belong to a person
- Wiring submitted projects into both `MyProfile` and `DeveloperProfile`

`CreateProject.tsx` was later worked on by Serina and Ivonne as well. The autofill and submission logic are mine; the validation and the visual pass are theirs.

### Agile facilitation

I was the team's Agile Facilitator. I set the two sprint structure, core MVP in sprint one and feature expansion in sprint two, ran standups and retros, translated scope into tickets, and managed the backlog. I reviewed and merged pull requests into `dev`, sharing merge duty with the team's Git Keeper.

My pull requests in the original repository: [`author:henryn289`](https://github.com/Oscar-Wakefield/Kiwicom/pulls?q=is%3Apr+author%3Ahenryn289)

## What I did not build

Bookmarking, project search and filtering on the home page, the single project page, form validation, and the editorial visual design were built by Serina, Ivonne and Oscar. The `useDebounce` hook in `client/hooks/` is Oscar's and powers the home page search, not my autofill.

---

## Stack as actually implemented

| Layer | Technology |
| --- | --- |
| Frontend | React, Vite, TypeScript |
| Styling | Tailwind CSS |
| Backend | Node.js, Express, TypeScript |
| Database | PostgreSQL, hosted on Supabase |
| Query builder | Knex |
| Auth | Supabase GitHub OAuth |
| GitHub integration | Octokit (`@octokit/rest`) |
| Tests | Vitest, React Testing Library |

Supabase is used for its hosted Postgres and for authentication only. All data access goes through Knex and Express, not the Supabase client library.

shadcn/ui appears in the team's original planning document but was never installed. The two components under `client/components/ui/` came from 21st.dev instead, built on Radix with `clsx` and `tailwind-merge`. Everything else is Tailwind written by hand.

---

## Running it locally

Verified from a clean clone as far as the database connection. `npm install`, the `.env` copy and both `knex` commands run correctly: the knexfile path resolves and knex reaches the connection step. The migration and seed steps then stop with `ECONNREFUSED` because `DATABASE_URL` is empty, which is the expected behaviour without a database. They have not been run against a live Supabase project.

You will need your own Supabase project.

```bash
npm install
cp .env.example .env
```

The committed `.env.example` covers only the two client variables, so a straight copy leaves the server without a database. `server/db/connection.ts` catches this and says so. Your `.env` needs all three:

```
VITE_SUPABASE_URL=
VITE_SUPABASE_ANON_KEY=
DATABASE_URL=
```

Migrations and seeds live under `server/db/`, and the knexfile is at `server/db/knexfile.js` rather than the project root:

```bash
npx knex migrate:latest --knexfile server/db/knexfile.js
npx knex seed:run --knexfile server/db/knexfile.js
npm run dev
```

No GitHub personal access token is required. The repository search uses each signed in user's own OAuth token, passed through from the client.

## Tests

Six Vitest suites exist, covering the home page, navbar, project card, project page, and the projects database functions and routes. I co-authored `server/db/functions/__tests__/projects.test.ts`; the rest are my teammates' work. Coverage is partial and there are no tests for the auth hook or the GitHub proxy route.

```bash
npm test
```

---

## A decision I would revisit

The `useAuth` hook stores the GitHub OAuth token in `sessionStorage`. Supabase only returns `provider_token` once, on the sign in event, so without stashing it the repository autofill broke every time someone refreshed the page. It works, and `sessionStorage` at least clears when the tab closes, but anything kept there is readable by any JavaScript running on the page. If the app had an XSS hole or a dependency were compromised, that token could be read and used against the GitHub API as that user.

With more time I would keep the token on the server and have the client call our own endpoint, so the browser never holds it at all. That was more plumbing than a one week sprint allowed. I took the shortcut knowingly rather than by accident.

---

## Credit

Kiwicom was a team effort. Oscar Wakefield set scope and priorities, Serina McFall kept the branch history healthy and shared review and merge duty, and Ivonne Valenzuela built validation and testing. Original repository and full history: [Oscar-Wakefield/Kiwicom](https://github.com/Oscar-Wakefield/Kiwicom).
