---
name: svelte-skill
description: Assists with Svelte 5 and SvelteKit development using runes, snippets, and modern patterns. Use when creating Svelte components, SvelteKit routes, data loading, form actions, API endpoints, or any Svelte 5 code.
allowed-tools: Read, Grep, Glob, Edit, Write, Bash, WebFetch
---

# Svelte 5 & SvelteKit Development Guide

## Documentation

- Svelte: https://svelte.dev/docs/svelte/llms.txt
- SvelteKit: https://svelte.dev/docs/kit/llms.txt

---

# PART 1: SVELTE 5 CORE

---

## RUNES (Reactivity System)

Runes are built-in Svelte keywords prefixed with `$`. They are language keywords—do NOT import them.

### $state

```svelte
<script lang="ts">
  let count = $state(0);
  let user = $state({ name: 'John', age: 30 });
  let items = $state<string[]>([]);
</script>
```

**Deep Reactivity:** Arrays/objects become deeply reactive proxies.

```typescript
let todos = $state([{ text: 'Learn', done: false }]);
todos[0].done = true;
todos.push({ text: 'Build', done: false });
```

**WARNING:** Never destructure reactive proxies:
```typescript
let { done } = todos[0];
```

### $state.raw

Shallow state requiring full reassignment:

```typescript
let items = $state.raw([1, 2, 3]);
items = [...items, 4];
```

### $state.snapshot

Plain object copy for external APIs:

```typescript
const plain = $state.snapshot(reactiveData);
```

### $state in Classes

```typescript
class Todo {
  done = $state(false);
  text = $state('');

  constructor(text: string) {
    this.text = text;
  }

  toggle = () => {
    this.done = !this.done;
  };
}
```

---

### $derived

```typescript
let count = $state(0);
let doubled = $derived(count * 2);
let quadrupled = $derived(doubled * 2);
```

### $derived.by

```typescript
let filtered = $derived.by(() => {
  return items.filter(i => i.active).sort((a, b) => a.name.localeCompare(b.name));
});
```

---

### $effect

Runs after DOM updates when dependencies change:

```typescript
$effect(() => {
  console.log(`Count: ${count}`);
  return () => console.log('Cleanup');
});
```

### $effect.pre

Runs BEFORE DOM updates:

```typescript
$effect.pre(() => {
  scrollPosition = container.scrollHeight;
});
```

### $effect.tracking

```typescript
console.log($effect.tracking());
```

### $effect.root

Manual cleanup scope:

```typescript
const cleanup = $effect.root(() => {
  $effect(() => { /* ... */ });
  return () => { /* cleanup */ };
});
```

---

### $props

```svelte
<script lang="ts">
  interface Props {
    title: string;
    count?: number;
    onUpdate?: (value: number) => void;
  }

  let { title, count = 0, onUpdate }: Props = $props();
  let { class: className, ...rest } = $props();
</script>
```

### $props.id

```typescript
const id = $props.id();
```

---

### $bindable

```svelte
<script lang="ts">
  let { value = $bindable('') } = $props();
</script>

<input bind:value />
```

Parent:
```svelte
<Child bind:value={parentValue} />
```

---

### $inspect

Development-only (removed in production):

```typescript
$inspect(count);
$inspect(count).with(console.trace);
```

---

## SNIPPETS & RENDERING

Snippets replace slots in Svelte 5.

### Defining Snippets

```svelte
{#snippet card(title, content)}
  <div class="card">
    <h2>{title}</h2>
    <p>{content}</p>
  </div>
{/snippet}

{@render card('Hello', 'World')}
```

### Snippets as Props

```svelte
<script lang="ts">
  import type { Snippet } from 'svelte';

  interface Props {
    header: Snippet;
    row: Snippet<[item: any, index: number]>;
    children: Snippet;
  }

  let { header, row, children }: Props = $props();
</script>

{@render header()}
{#each items as item, i}
  {@render row(item, i)}
{/each}
{@render children()}
```

Parent:
```svelte
<Table>
  {#snippet header()}<th>Name</th>{/snippet}
  {#snippet row(item, i)}<td>{item.name}</td>{/snippet}
  <p>Default children</p>
</Table>
```

---

## EVENT HANDLING

Svelte 5 uses `onclick` not `on:click`:

```svelte
<button onclick={() => count++}>Click</button>
<button onclick={handleClick}>Click</button>

<input
  oninput={(e) => value = e.currentTarget.value}
  onkeydown={(e) => e.key === 'Enter' && submit()}
/>
```

Event modifiers:
```svelte
<script lang="ts">
  import { preventDefault, stopPropagation } from 'svelte/events';
</script>

<form onsubmit={preventDefault(handleSubmit)}>
```

---

## TEMPLATE SYNTAX

### Control Flow

```svelte
{#if condition}
  <p>True</p>
{:else if other}
  <p>Other</p>
{:else}
  <p>False</p>
{/if}

{#each items as item, index (item.id)}
  <li>{index}: {item.name}</li>
{:else}
  <li>No items</li>
{/each}

{#await promise}
  <p>Loading...</p>
{:then value}
  <p>{value}</p>
{:catch error}
  <p>{error.message}</p>
{/await}

{#key expression}
  <Component />
{/key}
```

### Bindings

```svelte
<input bind:value />
<input type="checkbox" bind:checked />
<select bind:value>
<textarea bind:value />
<div bind:clientWidth bind:clientHeight />
<div bind:this={element} />
```

### Class & Style

```svelte
<div class:active={isActive}>
<div class:active>
<div class={{ active: isActive, disabled }}>

<div style:color="red">
<div style:--custom={value}>
```

---

## SPECIAL ELEMENTS

### <svelte:boundary>

```svelte
<svelte:boundary onerror={(e) => console.error(e)}>
  {#snippet failed(error, reset)}
    <p>Error: {error.message}</p>
    <button onclick={reset}>Retry</button>
  {/snippet}

  {#snippet pending()}
    <p>Loading...</p>
  {/snippet}

  <AsyncComponent />
</svelte:boundary>
```

### Others

```svelte
<svelte:element this={tag} {...attrs}>Content</svelte:element>
<svelte:window onkeydown={handleKey} bind:innerWidth />
<svelte:document onvisibilitychange={handle} />
<svelte:head><title>{title}</title></svelte:head>
<svelte:options customElement="my-el" />
```

---

## CONTEXT API

```typescript
import { setContext, getContext, createContext } from 'svelte';

const [getUser, setUser] = createContext<User>();
setUser({ name: 'John' });
const user = getUser();
```

With reactive state:
```typescript
setContext('counter', {
  count: $state(0),
  increment() { this.count++ }
});
```

---

## TRANSITIONS

```svelte
<script lang="ts">
  import { fade, fly, slide, scale } from 'svelte/transition';
  import { flip } from 'svelte/animate';
</script>

{#if visible}
  <div transition:fade>Fades</div>
  <div in:fly={{ y: 200 }} out:fade>Fly in, fade out</div>
{/if}

{#each items as item (item.id)}
  <div animate:flip={{ duration: 300 }}>{item.name}</div>
{/each}
```

---

# PART 2: SVELTEKIT WITH SVELTE 5

---

## PROJECT STRUCTURE

```
src/
├── lib/                    # $lib alias
│   ├── components/
│   ├── server/             # Server-only (import protection)
│   └── utils/
├── routes/
│   ├── +page.svelte
│   ├── +page.ts            # Universal load
│   ├── +page.server.ts     # Server load & actions
│   ├── +layout.svelte
│   ├── +layout.server.ts
│   ├── +server.ts          # API endpoint
│   ├── +error.svelte
│   ├── (group)/            # Route group (no URL effect)
│   ├── [slug]/             # Dynamic param
│   ├── [category]/[id]/    # Multiple params
│   ├── [[optional]]/       # Optional param
│   └── [...rest]/          # Rest params
├── hooks.server.ts
├── hooks.client.ts
├── app.html
└── app.d.ts
static/
svelte.config.js
```

---

## ROUTE FILES

| File | Purpose |
|------|---------|
| `+page.svelte` | Page component |
| `+page.ts` | Universal load (server + client) |
| `+page.server.ts` | Server-only load & form actions |
| `+layout.svelte` | Layout wrapper |
| `+layout.ts` | Layout universal load |
| `+layout.server.ts` | Layout server load |
| `+server.ts` | API endpoint |
| `+error.svelte` | Error boundary |

---

## DATA LOADING

### Universal Load (+page.ts)

Runs on server (SSR) and client (navigation):

```typescript
import type { PageLoad } from './$types';

export const load: PageLoad = async ({ fetch, params, url, depends }) => {
  depends('app:items');

  const res = await fetch(`/api/items/${params.id}`);
  if (!res.ok) throw error(404, 'Not found');

  return {
    item: await res.json(),
    query: url.searchParams.get('q')
  };
};
```

### Server Load (+page.server.ts)

Server-only with access to cookies, locals, private env:

```typescript
import type { PageServerLoad } from './$types';
import { error } from '@sveltejs/kit';
import { SECRET_KEY } from '$env/static/private';

export const load: PageServerLoad = async ({ params, locals, cookies, setHeaders }) => {
  if (!locals.user) throw error(401, 'Unauthorized');

  setHeaders({ 'cache-control': 'max-age=60' });

  const item = await db.items.findUnique({ where: { id: params.id } });
  if (!item) throw error(404, 'Not found');

  return { item };
};
```

### Layout Load (+layout.server.ts)

```typescript
import type { LayoutServerLoad } from './$types';

export const load: LayoutServerLoad = async ({ locals }) => {
  return { user: locals.user };
};
```

### Using Load Data in Components

```svelte
<script lang="ts">
  import type { PageData } from './$types';

  let { data }: { data: PageData } = $props();
</script>

<h1>{data.item.title}</h1>
```

### Parent Data Access

```typescript
export const load: PageLoad = async ({ parent }) => {
  const { user } = await parent();
  return { canEdit: user?.role === 'admin' };
};
```

### Invalidation

```typescript
import { invalidate, invalidateAll } from '$app/navigation';

invalidate('app:items');
invalidate('/api/items');
invalidateAll();
```

---

## FORM ACTIONS

### Server Actions (+page.server.ts)

```typescript
import type { Actions } from './$types';
import { fail, redirect } from '@sveltejs/kit';

export const actions: Actions = {
  default: async ({ request, locals }) => {
    const data = await request.formData();
    const email = data.get('email')?.toString();
    const password = data.get('password')?.toString();

    if (!email) return fail(400, { email, missing: true });
    if (!password) return fail(400, { email, passwordMissing: true });

    const user = await db.users.findUnique({ where: { email } });
    if (!user) return fail(400, { email, invalid: true });

    locals.user = user;
    throw redirect(303, '/dashboard');
  },

  logout: async ({ cookies }) => {
    cookies.delete('session', { path: '/' });
    throw redirect(303, '/login');
  }
};
```

### Form Component

```svelte
<script lang="ts">
  import { enhance } from '$app/forms';
  import type { ActionData } from './$types';

  let { form }: { form: ActionData } = $props();
</script>

<form method="POST" use:enhance>
  <input name="email" type="email" value={form?.email ?? ''} />
  {#if form?.missing}<p class="error">Email required</p>{/if}
  {#if form?.invalid}<p class="error">Invalid credentials</p>{/if}

  <input name="password" type="password" />
  {#if form?.passwordMissing}<p class="error">Password required</p>{/if}

  <button>Login</button>
</form>

<form method="POST" action="?/logout" use:enhance>
  <button>Logout</button>
</form>
```

### Custom Enhance

```svelte
<form
  method="POST"
  use:enhance={() => {
    return async ({ result, update }) => {
      if (result.type === 'success') {
        await update();
      }
    };
  }}
>
```

---

## API ENDPOINTS (+server.ts)

```typescript
import type { RequestHandler } from './$types';
import { json, error } from '@sveltejs/kit';

export const GET: RequestHandler = async ({ params, url, locals }) => {
  if (!locals.user) throw error(401, 'Unauthorized');

  const limit = Number(url.searchParams.get('limit')) || 10;
  const items = await db.items.findMany({ take: limit });

  return json(items);
};

export const POST: RequestHandler = async ({ request, locals }) => {
  if (!locals.user) throw error(401, 'Unauthorized');

  const body = await request.json();
  const item = await db.items.create({ data: body });

  return json(item, { status: 201 });
};

export const PUT: RequestHandler = async ({ params, request }) => {
  const body = await request.json();
  const item = await db.items.update({
    where: { id: params.id },
    data: body
  });

  return json(item);
};

export const DELETE: RequestHandler = async ({ params }) => {
  await db.items.delete({ where: { id: params.id } });
  return new Response(null, { status: 204 });
};
```

---

## HOOKS

### Server Hooks (hooks.server.ts)

```typescript
import type { Handle, HandleFetch, HandleServerError } from '@sveltejs/kit';
import { sequence } from '@sveltejs/kit/hooks';

const auth: Handle = async ({ event, resolve }) => {
  const session = event.cookies.get('session');

  if (session) {
    event.locals.user = await db.users.findUnique({
      where: { sessionId: session }
    });
  }

  return resolve(event);
};

const logger: Handle = async ({ event, resolve }) => {
  const start = Date.now();
  const response = await resolve(event);
  console.log(`${event.request.method} ${event.url.pathname} ${Date.now() - start}ms`);
  return response;
};

export const handle = sequence(auth, logger);

export const handleFetch: HandleFetch = async ({ request, fetch }) => {
  if (request.url.startsWith('https://api.internal/')) {
    request.headers.set('Authorization', `Bearer ${API_KEY}`);
  }
  return fetch(request);
};

export const handleError: HandleServerError = async ({ error, event }) => {
  console.error(error);
  return { message: 'Internal Error' };
};
```

### Client Hooks (hooks.client.ts)

```typescript
import type { HandleClientError } from '@sveltejs/kit';

export const handleError: HandleClientError = async ({ error }) => {
  console.error(error);
  return { message: 'Something went wrong' };
};
```

---

## PAGE STATE WITH SVELTE 5 ($app/state)

```svelte
<script lang="ts">
  import { page, navigating, updated } from '$app/state';

  let currentPath = $derived(page.url.pathname);
  let userId = $derived(page.params.id);
  let isLoading = $derived(navigating.to !== null);
  let searchQuery = $derived(page.url.searchParams.get('q'));
</script>

{#if isLoading}
  <p>Loading...</p>
{/if}

<p>Path: {currentPath}</p>
```

---

## NAVIGATION

```typescript
import { goto, invalidate, invalidateAll, preloadData } from '$app/navigation';

goto('/dashboard');
goto('/search', { replaceState: true });
goto(`?tab=settings`, { noScroll: true });

preloadData('/expensive-page');
```

### Shallow Routing

```typescript
import { pushState, replaceState } from '$app/navigation';
import { page } from '$app/state';

pushState('', { showModal: true });

let showModal = $derived(page.state.showModal);
```

---

## PAGE OPTIONS

```typescript
export const prerender = true;
export const ssr = false;
export const csr = true;
export const trailingSlash = 'never';
```

---

## ENVIRONMENT VARIABLES

```typescript
import { PUBLIC_API_URL } from '$env/static/public';
import { SECRET_KEY, DATABASE_URL } from '$env/static/private';
import { env } from '$env/dynamic/private';

const runtime = env.RUNTIME_SECRET;
```

---

## ERROR HANDLING

### Throwing Errors

```typescript
import { error } from '@sveltejs/kit';

throw error(404, 'Not found');
throw error(401, { message: 'Unauthorized', code: 'AUTH_REQUIRED' });
```

### Error Page (+error.svelte)

```svelte
<script lang="ts">
  import { page } from '$app/state';
</script>

<h1>{page.status}</h1>
<p>{page.error?.message}</p>
```

---

## AUTHENTICATION PATTERN

### hooks.server.ts

```typescript
export const handle: Handle = async ({ event, resolve }) => {
  const session = event.cookies.get('session');

  if (session) {
    const user = await db.sessions.findUnique({
      where: { id: session },
      include: { user: true }
    });

    if (user && user.expiresAt > new Date()) {
      event.locals.user = user.user;
    }
  }

  if (event.url.pathname.startsWith('/dashboard') && !event.locals.user) {
    throw redirect(303, '/login');
  }

  return resolve(event);
};
```

### app.d.ts

```typescript
declare global {
  namespace App {
    interface Locals {
      user: { id: string; email: string; role: string } | null;
    }
    interface Error {
      message: string;
      code?: string;
    }
  }
}

export {};
```

---

## ADAPTERS

```javascript
import adapter from '@sveltejs/adapter-auto';
import adapter from '@sveltejs/adapter-node';
import adapter from '@sveltejs/adapter-static';
import adapter from '@sveltejs/adapter-vercel';
import adapter from '@sveltejs/adapter-cloudflare';

export default {
  kit: {
    adapter: adapter()
  }
};
```

---

## CLI

```bash
npx sv create my-app
npx sv add tailwindcss
npx sv add drizzle
npx sv add lucia
npx sv check
npx sv migrate svelte-5
```

---

## SECURITY

1. Validate ALL inputs server-side in `+page.server.ts`
2. Use `$env/static/private` for secrets
3. Use `$lib/server` to prevent client imports
4. CSRF protection enabled by default
5. Sanitize dynamic HTML
6. Never expose stack traces in `fail()` responses
7. Use `httpOnly` and `secure` flags on cookies
8. Validate session tokens on every request
