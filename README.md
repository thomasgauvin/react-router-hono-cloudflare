# React-Router + Hono template, on Cloudflare

If this is you...

- You like building your frontend applications with React + React-Router
- You like building your APIs with Hono
- You want both to live in the same repo to share code
- You want to run both using Vite for fast developement
- You want to deploy to Cloudflare?

Then, you've come to the right place! This is a template React-Router, Hono, Vite, Cloudflare project, ready for you to get started! Just run this command to get started with a new project:

```
npm create cloudflare@latest -- --template thomasgauvin/react-router-hono-cloudflare
```

That's it! Now, run your project locally with:

```
npm run dev
```

And deploy with:

```
npm run deploy
```

---

### Why put this template together?

There's a lot of templates out there showing you how to run React-Router and Hono but most of them are outdated (as of June 2025) because React-Router, Hono and Cloudflare Vite support have evolved a lot over the past 6 months.

There's been great work put into [react-router-hono-server](https://github.com/rphlmr/react-router-hono-server) and [remix-hono](https://github.com/sergiodxa/remix-hono). But, as of June 2025, those solutions aren't ideal for hosting on Cloudflare Workers.

A comment by [agcty](https://github.com/agcty) in one of these repos actually pointed out that recent developments in Cloudflare, Vite and React Router have made it much easier to configure this set up. This repository makes it easier to get a project started with this stack and makes it (hopefully) easier to discover that this is how to set this up.

This project is very simple. It's just the [original deployment template for Cloudflare and React Router ](https://github.com/remix-run/react-router-templates/tree/main/cloudflare) with a small change to use Hono to serve the React Router project, allowing Hono routes to take effect and fallback to React Router:

```diff
+ import { Hono } from 'hono';
import { createRequestHandler } from "react-router";

declare module "react-router" {
  export interface AppLoadContext {
    cloudflare: {
      env: Env;
      ctx: ExecutionContext;
    };
  }
}

const requestHandler = createRequestHandler(
  () => import("virtual:react-router/server-build"),
  import.meta.env.MODE
);

- export default {
-   async fetch(request, env, ctx) {
-     return requestHandler(request, {
-       cloudflare: { env, ctx },
-     });
-   },
- } satisfies ExportedHandler<Env>;

+ const app = new Hono<{ Bindings: Env }>();

+ app.all('*', (c) => {
+   return requestHandler(c.req.raw, {
+     cloudflare: { env: c.env, ctx: c.executionCtx as ExecutionContext },
+   });
+ });

+ export default app;
```
