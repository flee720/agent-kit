# Verification adapters

pstack's `create-verification-skill` assumes an app you can launch locally and drive like a user. These notes say what launch, drive, evidence and isolation mean for each kind of project. Use the matching section while answering its interview questions. Everything still gets checked against the actual repo.

## Web app with shared services (Next.js, Supabase, a queue, workers)

- **Launch:** the repo's own dev command (`pnpm dev`, a turbo task) on a free port. "Ready" means a real page renders and a signed-in route loads, not just that the port answers.
- **Drive:** a real browser. Cloud sessions ship Chromium with Playwright: `require('playwright')` works, and `chromium.launch()` needs no options. For background work (jobs, workers, queues), submit through the UI or public API the way a user would, then follow the job to a terminal state.
- **Isolation, the hard part:** parallel agents sharing one dev database overwrite each other's schema and data. In order of preference: a database per agent (a Supabase branch, or the local Supabase stack where Docker exists), then a dedicated test project with seed data. The production project only if the repo's data policy allows it, and then read-only.
- **Auth:** a dedicated test account, with credentials in environment secrets, never in the repo.
- **Paid external APIs:** mock only where a production boundary already isolates the service, or make a capped real call. Write the cost per run in the skill.
- **Evidence:** screenshots of each state, the network responses behind them, database rows and job states before and after.

## Device firmware with a host simulator (ESP32, LVGL, embedded panels)

- **Launch:** there is no server. Build the host simulator once (usually CMake), then each drive is one simulator run that renders the UI to image files.
- **Drive:** the inputs the simulator exposes (profiles, layouts, sensor values, alarm and fault states), set the way the firmware would receive them. Render every state the feature has, including missing-sensor and alarm states.
- **Evidence:** the rendered frames before and after, a contact sheet if the repo has a tool for one, unit test output, and an emulator boot test if one exists (QEMU smoke).
- **What the simulator cannot prove:** touch calibration, real sensors, bus timing, power, anything physical. Mark each one `verified-unreachable` with the prerequisite ("needs the panel on the bench"). Add it to the repo's hardware test log for the owner. Never claim hardware behavior from simulator output.
- **Isolation:** simulator runs are stateless. Parallel runs are fine with separate output directories.

## Shopify theme

- **The live-store trap:** a branch connected through Shopify's GitHub integration publishes on every merge. Find out which branch feeds the published (role `MAIN`) theme and write it in the skill. Never run `shopify theme publish`, and never push to the published theme.
- **Launch:** push the branch to its own unpublished theme with the Shopify CLI (`shopify theme push --unpublished`, plus `--store` and a Theme Access password from the environment). Record the theme id and preview URL it returns. Check `shopify theme push --help` for the current flags.
- **Drive:** Playwright on `https://<store>/?preview_theme_id=<id>`. A preview theme renders the store's real products, collections and reviews, which is the production data you need for free. Cover home, a collection, a product (variant change and add to cart) and the cart, plus whatever the change touches.
- **Evidence:** screenshots at phone width (about 390 px) and desktop width (about 1440 px), console errors, and parsed JSON-LD for product and article pages. Add `shopify theme check` output for static problems.
- **Isolation:** one unpublished theme per agent or branch. The theme library holds at most 20, so the cleanup step deletes the theme (`shopify theme delete`).
- **Network:** the cloud environment must allow the store's domain and `cdn.shopify.com`.

## CLI or HTTP service

- **CLI or TUI:** build once, then each drive runs in its own PTY or tmux session. Evidence is the transcript and the exit code.
- **HTTP service:** start it on a free port and drive it with real requests. Evidence is the request, the response and any side effects.
