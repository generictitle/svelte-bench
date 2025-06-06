## Svelte 5 & SvelteKit Documentation

**How to Read These Instructions:**

*   **Goal:** Generate Svelte 5 / SvelteKit code following modern best practices.
*   **Svelte Version:** You **MUST** use Svelte 5 syntax (Runes, Snippets). Svelte 4 comparisons are included *only* to help you avoid outdated patterns.
*   **Directive Formatting:**
    *   **`USE [Correct Pattern] (Replaces S4 [Old Pattern])`**: Standard Svelte 5 way, noting S4 replacement. Prioritize `USE`.
    *   **`CORRECT: [Good Practice/Example]. AVOID: [Bad Practice/Example].`**: Highlights recommended (`CORRECT`) vs. discouraged (`AVOID`). Always implement `CORRECT`.
*   **Conciseness:** Information is condensed. Focus on patterns, examples, and keywords.

---

### Svelte 5 Documentation

**Core Principles & Setup:**

*   **Svelte 5 Mandate:** `MUST` use Svelte 5 API (Runes). Unchanged syntax (`{#if}`) reused from S4 knowledge.
*   **Runes Overview:** Built-in keywords (`$`) controlling compiler. `CORRECT: Use as language keywords. AVOID: Importing/calling runes like functions.` Valid in `.svelte`, `.svelte.js`, `.svelte.ts` files.
*   **`.svelte` Files:** Optional `<script>`, `<script module>`, `<style>`, markup sections.
    *   `<script>`: Runs per component instance. Use runes here.
    *   `<script module>`: Runs once per module load. Can export non-default bindings. Variables accessible in instance script, not vice-versa.
    *   `<style>`: CSS scoped to the component by default.
*   **`.svelte.js`/`.svelte.ts` Files:** Regular JS/TS modules supporting runes for reactive logic/state sharing.
    ```js
    // store.svelte.js
    export let count = $state(0); // Export reactive state
    export const increment = () => count++;
    ```
*   **DOM Event Syntax:** `USE onclick={...} (Replaces S4 on:click={...}).` No colon.

**State Management (`$state`)**

*   **`$state`:** Creates reactive state. `USE let count = $state(0); (Replaces S4 let count = 0; for reactivity).`
    ```svelte
    <script> let count = $state(0); </script>
    <button onclick={() => count++}>Clicked: {count}</button>
    ```
*   **Deep State:** Objects/arrays become deeply reactive proxies. Modifications trigger updates.
    ```js
    let obj = $state({ nested: { value: 1 } });
    obj.nested.value = 2; // UI updates
    let arr = $state([]);
    arr.push({ text: 'new' }); // UI updates, new item is reactive
    ```
    `CORRECT: Access properties directly: obj.nested.value. AVOID: Destructuring reactive proxies: let { value } = obj.nested; (breaks reactivity).`
*   **Direct Updates:** `CORRECT: Update $state variables directly: count++, arr.push({...}), obj.prop = value. AVOID: Wrapping state in unnecessary custom objects/stores for simple component state.`
*   **Classes:** Use `$state` in class fields. Handle `this` context for methods (use arrow functions or `() => instance.method()` in handlers).
    ```js
    class Todo { done = $state(false); text = $state(''); reset = () => { /*...*/ }; }
    ```
*   **Exporting State:** `CORRECT: Export stateful object: export const counter = $state({ count: 0 }); OR Export getters/actions: export function getCount() { return count; } export function increment() { count++; }. AVOID: Directly exporting reassigned state: export let count = $state(0);.`
*   **`$state.raw`:** Shallow, non-tracked state. Updates require reassignment.
    ```js
    let person = $state.raw({ name: 'Zeno' });
    person = { ...person, age: 71 }; // Correct update
    ```
    `CORRECT: Reassign entire object. AVOID: Mutating properties of raw state.`
*   **`$state.snapshot`:** Creates a non-reactive, plain object copy. `CORRECT: Use ONLY if external API needs plain object. AVOID: Overusing.`
*   **Passing State:** Pass by value semantics. `CORRECT: Pass state to functions via getters: fn(() => stateVal). AVOID: Passing raw state variables assuming live updates inside function.`

**Derived State (`$derived`)**

*   **`$derived`:** Computes reactive values from dependencies. `USE const doubled = $derived(count * 2); (Replaces S4 $: double = count * 2;).`
    ```svelte
    <script> let count = $state(0); const doubled = $derived(count * 2); </script>
    <p>{count} * 2 = {doubled}</p>
    ```
*   **`$derived.by`:** For multi-line/complex logic. `USE $derived.by(() => {...}) when passing a function.`
    ```js
    const total = $derived.by(() => { let sum = 0; /*...*/ return sum; });
    ```
*   **Reactivity:** Values are not deep proxies. Updates trigger when dependencies change value. Uses push-pull reactivity (recalculated on read).
*   **Purity:** `CORRECT: Keep $derived pure (no side effects). AVOID: Assignments/API calls inside $derived.`
*   **Dependencies:** Tracks synchronous reads. Use `untrack()` to exclude.
*   **Overriding:** Temporarily assign new value (optimistic UI). Reverts on next dependency update. `CORRECT: Assign directly: likes = 1;. AVOID: Overriding via $effect.`
    ```svelte
    <script> let likes = $derived(post.likes); async function like() { likes++; /*...*/ } </script>
    ```

**Side Effects (`$effect`)**

*   **`$effect`:** Runs code reactively *after* DOM updates when dependencies change. Browser-only. `USE $effect(() => {...}); (Replaces S4 $: console.log(...) for side effects).`
    ```svelte
    <script> let size = $state(50); $effect(() => { console.log('Size:', size); }); </script>
    ```
*   **Usage:** `CORRECT: Use $effect for: logging, non-declarative DOM manipulation, external subscriptions (WebSocket, timers), analytics. AVOID: Using $effect for state sync/derivation (use $derived).`
*   **Lifecycle & Cleanup:** Can return a cleanup function (runs before re-run or on unmount). `CORRECT: Return cleanup for intervals, listeners, etc. AVOID: Forgetting cleanup.`
    ```js
    $effect(() => { const id = setInterval(...); return () => clearInterval(id); });
    ```
*   **Dependencies:** Tracks synchronous reads only. Async values (post-`await`, `setTimeout`) not tracked.
*   **`$effect.pre`:** Runs *before* DOM updates. `CORRECT: Use for pre-DOM tasks (reading scroll position). AVOID: Standard post-update effects.`
*   **`$effect.tracking`:** Debugging helper: returns `true` if inside reactive context.
*   **`$effect.root`:** Creates non-tracked scope needing manual cleanup via returned function. Niche use case.

**Component Inputs (`$props`)**

*   **`$props`:** Access component inputs. `USE let { foo = true, bar } = $props(); (Replaces S4 export let foo = true; export let bar;).`
    ```svelte
    <!-- MyComponent.svelte -->
    <script> let { adjective = 'great' }: { adjective?: string } = $props(); </script>
    <p>Component is {adjective}</p>
    ```
*   **Immutability:** `CORRECT: Treat props read-only. Use callbacks or $bindable for changes. AVOID: Directly modifying props: adjective = 'new';.` Reassigning props locally is allowed but usually discouraged; mutation has complex rules/warnings (see source doc if needed).
*   **Features:** Fallbacks (`prop = 'default'`), renaming (`{ class: className }`), rest (`{ a, ...rest }`).
*   **`$props.id()`:** Generates unique component instance ID string. `CORRECT: Use for unique IDs (labels, aria). AVOID: Manual ID generation.`
*   **Typing:** Use interfaces or inline types: `let { name }: { name: string } = $props();`. For generics: `<script lang="ts" generics="T"> ... let { item }: { item: T } = $props(); </script>`.

**Two-Way Binding (`$bindable`)**

*   **`$bindable`:** Explicitly marks prop for two-way binding (`bind:prop`). `USE let { value = $bindable('fallback') } = $props(); (Unlike S4 implicit bindable props).`
    ```svelte
    <!-- FancyInput.svelte -->
    <script> let { value = $bindable('') } = $props(); </script>
    <input bind:value={value} />
    ```
    ```svelte
    <!-- App.svelte -->
    <script> let name = $state('world'); </script>
    <FancyInput bind:value={name} />
    ```
    `CORRECT: Use when two-way binding is needed. AVOID: Overusing; prefer one-way flow + callbacks.`

**Snippets (Reusable Markup Chunks)**

*   **Concept:** Define reusable, parameterized **HTML markup** sections within the component **template area**. `USE {#snippet ...}` (Replaces S4 `<slot>`).
*   **Definition:** `{#snippet name(params)}...{/snippet}`. Parameters are optional, support defaults (`p='default'`), and destructuring. No rest params (`...args`).
    ```svelte
    <script>
      let message = 'Hello'; // Accessible by snippet below
    </script>

    <!-- Snippet defined in template markup -->
    {#snippet greet(name='world')}
      <p>{message} {name}!</p>
    {/snippet}
    ```
*   **Rendering:** Use `{@render snippetName(args)}` **in the template** to display a snippet. `USE {@render ...}` (Replaces S4 `<slot ...>`).
    *   `CORRECT: Always invoke with parentheses: {@render greet()}, {@render greet('Svelte')}. AVOID: Omitting (): {@render greet}.`
    *   Render optional snippets: `{@render maybeSnippet?.()}`.
    ```svelte
    <!-- Rendering in template -->
    {@render greet('Developer')}
    ```
*   **Scope:** Snippets access variables from their outer `<script>` or template scope. Render within scope or pass down.
*   **Passing Snippets (in template):**
    *   **As Props:** Pass like values: `<ChildComponent {greet} />`.
    *   **Implicit Props:** Define inside component tag: `<Child>{#snippet greet ...}{/snippet}</Child>`. Becomes `greet` prop on `Child`.
    *   **`children` Snippet:** Direct content `<Child>Text</Child>` becomes `children` prop. Render via `{@render children()}` in `Child`.

**Template Syntax**

*   **Basic Markup:** Lowercase tags = HTML, Capitalized = Components. Attributes: Standard HTML, JS expressions `{value}`, boolean shorthand `{disabled}`, spread `{...attrs}`. Props: Similar syntax, shorthand `{prop}`, spread `{...props}`.
*   **Text Expressions:** `{expression}`. Escaped by default. `{@html rawHtmlString}` for unescaped HTML (use with caution!).
*   **Comments:** `<!-- HTML comment -->`. `<!-- svelte-ignore directive -->`. `<!-- @component Doc comment -->`.
*   **Logic Blocks:**
    *   `{#if condition}...{:else if condition}...{:else}...{/if}`
    *   `{#each items as item, index (item.id)}...{:else}...{/each}` (Key `(item.id)` is crucial for performance/animation). Supports destructuring.
    *   `{#await promise}...{:then value}...{:catch error}...{/await}` (Blocks optional).
    *   `{#key expression}...{/key}` (Destroys/recreates content when expression changes).
*   **`{@const ...}`:** Define local constant: `{@const area = width * height}`. Allowed inside blocks/components.
*   **`{@debug ...}`:** Debug helper: `{@debug var1, var2}` logs changes, pauses debugger. Dev-only.

**Debugging Reactivity (`$inspect`) (Development Only)**

*   **Purpose:** Observe state changes and trace dependencies during development. **`$inspect` is removed in production builds (no-op).**
*   **Log State Changes:** `$inspect(var1, var2, ...)` logs values when they change and pauses the debugger if devtools are open.
    ```svelte
    <script> let count = $state(0); $inspect(count); </script>
    <button onclick={() => count++}>Inc: {count}</button>
    ```
*   **Custom Inspection (`.with(callback)`):** Execute custom logic on change. `callback(type, value)` where `type` is 'init' or 'update'.
    ```js
    // Trigger debugger when count updates and exceeds 5
    $inspect(count).with((type, val) => { if (type === 'update' && val > 5) debugger; });
    ```
*   **Trace Dependencies (`.trace()`):** Inside `$effect` or `$derived`, use `$inspect.trace('optional label')` to log which specific dependencies triggered the re-run.
    ```js
    $effect(() => {
      $inspect.trace(); // Logs which dependency change triggered this effect
      console.log('Effect ran:', someState);
    });
    ```

**Data Binding (`bind:`)**

*   **Concept:** Two-way data binding. `bind:prop={value}` or shorthand `bind:value`.
*   **Common Inputs:** `bind:value` (text, number, range, select), `bind:checked` (checkbox), `bind:group` (radio/checkbox group), `bind:files` (file input). Use `defaultValue`/`defaultChecked` for form reset behavior.
*   **Media Elements:** `bind:currentTime`, `bind:paused`, `bind:duration` (audio/video), `bind:videoWidth/Height` (video), `bind:naturalWidth/Height` (img).
*   **Other Elements:** `bind:open` (details), `bind:innerHTML`/`textContent` (contenteditable), `bind:offsetWidth`/`Height`, `clientWidth`/`Height`, `scrollWidth`/`Height`, `scrollLeft`/`Top` (readonly dimensions/scroll).
*   **`bind:this`:** Get reference to DOM element or component instance.
*   **Component Binding:** Requires child prop marked with `$bindable`. `<Child bind:value={parentState} />`.
*   **Function Bindings:** Advanced control: `bind:value={() => getter, (v) => setter(v)}`. Readonly binding: `bind:clientWidth={null, redrawFn}`.

**Actions (`use:`)**

*   **`use:action={params}`:** Attach JS behavior to an element lifecycle.
    ```svelte
    <script>
      import { tick } from 'svelte'; // Use tick if needed inside action
      /** @type {import('svelte/action').Action<HTMLDivElement, { speed: number }>} */
      function longpress(node, params = { speed: 100 }) {
        // setup logic, e.g., event listeners
        console.log('Action mounted with speed', params.speed);
        return {
          // update(newParams) { /* logic if params change */ }, // Optional update function
          destroy() { /* cleanup logic */ }
        };
      }
    </script>
    <div use:longpress={{ speed: 50 }}>Long press me</div>
    ```
*   **Runs:** Only in browser, once on mount. `update` runs if params change. `destroy` runs on unmount.

**Transitions & Animations**

*   **`transition:fn={params}`:** Animate element in/out of DOM (bidirectional). From `svelte/transition`. Use `|global` modifier for non-local transitions.
*   **`in:fn={params}` / `out:fn={params}`:** Unidirectional transitions.
*   **`animate:fn={params}`:** Animate reordering within keyed `{#each}` blocks. From `svelte/animate`.
*   **Custom Functions:** Return object with `delay`, `duration`, `easing`, and `css` (preferred) or `tick`.

**Styling**

*   **`<style>` Block Usage:** Define component styles within a `<style>` tag. **The content inside `<style>` MUST be standard CSS syntax; it's processed at compile time, not generated dynamically with JS `{expressions}`.**
    ```svelte
    <script>
      // Svelte variable holding a color value
      let mainColor = $state('blue');
    </script>

    <!-- Set the CSS variable using the Svelte variable -->
    <p style:--p-color={mainColor}>Styled text</p>

    <style>
      /* Standard CSS using the variable */
      p {
        color: var(--p-color, black); /* Use CSS variable with fallback */
        font-weight: bold;
      }
    </style>
    ```
*   **Scoped Styles:** By default, CSS in `<style>` is **scoped** to the component (Svelte adds unique classes). Specificity is boosted. `@keyframes` also scoped.
*   **Global Styles:** `CORRECT: Use :global(.selector) or :global { ... } block. AVOID: Overusing.` Use `-global-` prefix for global `@keyframes`.
*   **Inline Styles (`style:`):** Set styles directly: `style:opacity={value}`, shorthand `style:color`, important `style:color|important="blue"`. Also used to set CSS custom properties (`style:--my-var={jsValue}`).
*   **CSS Classes (`class` / `class:`):**
    *   **`class` Attribute (Recommended):** `class="string"`, `class={expr}`, object `class={{ active: isActive }}`, array `class={[isActive && 'active']}`.
    *   `class:` Directive (Legacy): `class:name={bool}` or `class:name`. Prefer `class` attribute.
*   **CSS Custom Properties (`--*`):** Pass to components: `<Comp --bg-color="blue" />`. Access in CSS: `var(--bg-color, fallbackValue)`. Set dynamically using `style:--my-var={jsValue}`.

**Special Elements `<svelte:...>`**

*   **`<svelte:head>`:** Insert content into `document.head`.
*   **`<svelte:element this={tag} {...attrs}>`:** Render dynamic HTML element type. `bind:this` only.
*   **`<svelte:window bind:scrollY={y} on:event={handler}/>`:** Interact with `window`.
*   **`<svelte:document on:event={handler} use:action />`:** Interact with `document`.
*   **`<svelte:body on:event={handler} use:action />`:** Interact with `document.body`.
*   **`<svelte:boundary onerror={handler}>{#snippet failed(e,r)}</svelte:boundary>`:** Catch rendering errors. Provide `failed` snippet for fallback UI. `onerror` prop for programmatic handling. Catches render/effect errors, not async/event handler errors.

**Runtime API**

*   **Stores (`svelte/store`):** For complex async/external state. `writable`, `readable`, `derived`, `readonly`, `get`. Access value via `$store` prefix in components. Prefer runes (`.svelte.js` state) for simpler cases.
*   **Context API (`svelte`):** Pass values down component tree without props. `setContext(key, value)`, `getContext(key)`, `hasContext(key)`. Useful for theme, user data.
*   **Lifecycle (`svelte`):** `onMount(() => cleanup?)` (browser-only, runs after mount), `onDestroy(() => ...)` (runs before unmount), `tick()` (promise resolving after updates). `$effect.pre`/`$effect` replace `before/afterUpdate`.
*   **Imperative API (`svelte`):** Less common for generation. `mount(Comp, { target, props })`, `unmount(instance)`, `hydrate(Comp, { target, props })`. SSR: `render(Comp, { props })` from `svelte/server`.

---

### SvelteKit Documentation

**Core Concepts**

*   **Framework:** Builds full-stack apps with Svelte components. Handles routing, data loading, build optimization, SSR/CSR/SSG.
*   **Setup:** `USE npx sv create my-app`. Needs Node.js.
*   **Project Structure:** Key items: `src/routes` (app pages/endpoints), `src/lib` (shared code, `$lib` alias), `src/app.html` (HTML shell), `static` (public assets), `svelte.config.js`, `vite.config.js`.

**Routing (Filesystem-based in `src/routes`)**

*   **Pages:** `+page.svelte`. `USE +page.svelte (Replaces S4 route.svelte).`
*   **Layouts:** `+layout.svelte`. Wrap pages/child layouts. Use `{@render children()}`. Nesting inherits.
*   **Data Loading:** `+page.js`/`+layout.js` (universal load), `+page.server.js`/`+layout.server.js` (server-only load & actions).
*   **API Endpoints:** `+server.js`. Export functions named `GET`, `POST`, etc.
*   **Error Pages:** `+error.svelte`. Catches errors in its subtree.
*   **Dynamic Routes:** `[param]`, rest `[...param]`, optional `[[param]]`. Matchers `[param=matcher]` (defined in `src/params/matcher.js`).
*   **Advanced Layouts:** Route groups `(group)` (no URL segment). Layout reset `@` (`+page@.svelte`).

**Data Loading (`load`)**

*   **Mechanism:** `load` function (exported from `+page/layout .js/.server.js`) returns data object. This object becomes the `data` prop in the corresponding `.svelte` file. Layout data merges downwards.
    ```js
    // +page.server.js
    export async function load({ params }) { return { post: await db.getPost(params.slug) }; }
    ```
    ```svelte
    // +page.svelte
    <script> let { data } = $props(); </script> <h1>{data.post.title}</h1>
    ```
*   **Universal (`.js`) vs. Server (`.server.js`):** Server load for private data (DB, secrets), must return serializable data. Universal load runs server then client, can return non-serializable data (functions, classes).
*   **`fetch`:** `CORRECT: Use load's fetch argument. AVOID: Global fetch in load.`
*   **`$app/state` (`page`):** Reactive store with current page info (`url`, `params`, `data` from all loads, `error`, `status`, `form`).
*   **Streaming:** Return promises from server `load`, use `{#await}` in component.
*   **Invalidation:** `depends('key')` to track, `invalidate('key')`/`invalidateAll()` to rerun.

**Forms & Actions (`+page.server.js`)**

*   **Actions:** `export const actions = { default: async ({ request, cookies, locals }) => {}, name: async => {} };`. Handle `<form method="POST">` (`action="?/name"` for named).
*   **Return Values:**
    *   `fail(status, data)`: For validation errors. Populates `form` prop.
    *   `redirect(status, location)`: For redirects.
    *   Success: Reruns `load`, merges return value into `form` prop.
*   **Progressive Enhancement:** `USE use:enhance ($app/forms) on <form>`. Handles AJAX, state updates, errors, redirects. Customize with callback if needed.

**Page Options (Export `const` from `+page/layout .js/.server.js`)**

*   **`prerender = true`:** Static Site Generation (SSG). Needs `adapter-static`. Dynamic routes need `entries()`.
*   **`ssr = false`:** Client-Side Rendering only (SPA mode).
*   **`csr = false`:** No JS hydration (static HTML).
*   **`trailingSlash = 'always' | 'never' | 'ignore'`:** URL format.

**Other Core Features**

*   **Adapters:** Configure in `svelte.config.js` (`adapter-node`, `adapter-static`, `adapter-vercel`, etc.). Tailor build output for deployment.
*   **Hooks (`hooks.server.js`, `hooks.client.js`):** `handle` (middleware, modify `event.locals`), `handleError` (catch unexpected errors), `handleFetch` (intercept server fetch).
*   **Server-only Modules:** Use `$lib/server/`, `*.server.js`, `$env/.../private`. Build fails if imported into client code.
*   **State Management:** `CORRECT: Pass server data via load/props. Use context/stores/URL for client state. AVOID: Shared mutable server variables.`
*   **Error Handling:** `error(status, data)` helper throws expected error -> `+error.svelte`. Unexpected errors -> `handleError`.
*   **Link Options (`<a>` attributes):** `data-sveltekit-preload-data/code`, `data-sveltekit-reload`, etc. Tune navigation/preloading.
*   **Service Workers (`src/service-worker.js`):** Offline support via `$service-worker` module.
*   **Snapshots (`snapshot` export):** Preserve ephemeral component state across navigation.
*   **Shallow Routing (`$app/navigation`):** `pushState`/`replaceState` to change URL/state without navigation. Access via `page.state`.

**Best Practices**

*   **Auth:** Use `handle` hook to check cookies, set `event.locals.user`. Use libraries like Lucia.
*   **Images:** Vite handles static imports. `@sveltejs/enhanced-img` (`<enhanced:img>`) for optimization. Use CDNs for dynamic images. Add `alt` text.
*   **SEO:** Use `<svelte:head>` for metadata (`title`, `description`). Create `sitemap.xml` endpoint. Keep SSR enabled for public pages.

**Key Modules Reference**

*   **`@sveltejs/kit`:** `error`, `fail`, `redirect`, `json`, `text`.
*   **`$app/forms`:** `enhance`, `applyAction`, `deserialize`.
*   **`$app/navigation`:** `goto`, `invalidate`, `invalidateAll`, `preloadData`, `preloadCode`, `beforeNavigate`, `afterNavigate`, `pushState`, `replaceState`.
*   **`$app/paths`:** `base`, `assets`.
*   **`$app/server` (Server-only):** `read`, `getRequestEvent`.
*   **`$app/state`:** `page`, `navigating`, `updated`.
*   **`$env/...`:** Access environment variables.
*   **`$lib`:** Alias for `src/lib`.

**Import Requirements**

Understanding what needs an `import` statement is crucial.

**1. DO NOT Import (Globally Available or Implicit):**

*   **Runes:** All runes (`$state`, `$derived`, `$effect`, `$props`, `$bindable`, `$inspect`, `$host`) are compiler keywords. **Never** import them.
*   **Template Blocks/Directives:** All template syntax (`{#if}`, `{#each}`, `{#await}`, `{#snippet}`, `{@render}`, `{@html}`, `{@debug}`, `{@const}`, `bind:`, `use:`, `transition:`, `in:`, `out:`, `animate:`, `style:`, `class:`) are part of the Svelte template language. **No import needed.**
*   **Special Elements:** All `<svelte:...>` elements (`<svelte:head>`, `<svelte:window>`, `<svelte:document>`, `<svelte:body>`, `<svelte:element>`, `<svelte:options>`, `<svelte:boundary>`) are part of the Svelte language. **No import needed.**

**2. Imports Required When Needed:**

*   **From `svelte`:**
    *   Lifecycle functions: `onMount`, `onDestroy`, `tick`.
    *   Context API: `setContext`, `getContext`, `hasContext`, `getAllContexts`.
    *   Imperative API: `mount`, `unmount`, `hydrate`.
*   **From `svelte/transition`, `svelte/animate` and `svelte/easing`:**
    *   Built-in transitions, animations and easing functions
*   **From `@sveltejs/kit`:**
    *   API/Action helpers: `error`, `fail`, `redirect`, `json`, `text`.
*   **From `$app/...` Modules (SvelteKit):**
    *   `$app/forms`: `enhance`, `applyAction`, `deserialize`.
    *   `$app/navigation`: `goto`, `invalidate`, `invalidateAll`, `beforeNavigate`, `afterNavigate`, `preloadData`, `preloadCode`, `pushState`, `replaceState`, etc.
    *   `$app/paths`: `base`, `assets`.
    *   `$app/server` (Server-only): `read`, `getRequestEvent`.
    *   `$app/state`: `page`, `navigating`, `updated`.
    *   `$app/environment`: `dev`, `browser`, `building`.
*   **From `$env/...` Modules (SvelteKit):**
    *   Environment variables (`$env/static/public`, `$env/dynamic/private`, etc.).