# @supabase/auth-helpers-qwik

This submodule provides convenience helpers for implementing user authentication in Qwik applications.

## Installation

Using [npm](https://npmjs.org):

```sh
npm install @supabase/auth-helpers-qwik @thedeveloper/qwik-supabase
```

Using [yarn](https://yarnpkg.com/):

```sh
yarn add @supabase/auth-helpers-qwik @thedeveloper/qwik-supabase
```

This library supports the following tooling versions:

- Node.js: `^10.13.0 || >=12.0.0`

- Next.js: `>=10`

## Getting Started

### Configuration

Set up the following env vars in `.env`:

```bash
# Find these in your Supabase project settings > API
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key
```

### Basic Setup

Add `SupabaseProvider` to your `src/root.tsx`:

```tsx
import { $ } from '@builder.io/qwik';
import { createBrowserSupabaseClient } from '@supabase/auth-helpers-qwik';
import { SupabaseProvider } from '@thedeveloper/qwik-supabase';

export default component$(() => {
  const supabaseUrl = import.meta.env.VITE_SUPABASE_URL;
  const supabaseKey = import.meta.env.VITE_SUPABASE_ANON_KEY;

  /**
   * The root of a QwikCity site always start with the <QwikCity> component,
   * immediately followed by the document's <head> and <body>.
   *
   * Dont remove the `<head>` and `<body>` elements.
   */
  return (
    <QwikCity>
      <head>
        <meta charSet="utf-8" />
        <RouterHead />
      </head>
      <body lang="en">
        <SupabaseProvider client$={$(() => createBrowserSupabaseClient(supabaseUrl, supabaseKey))}>
          <RouterOutlet />
        </SupabaseProvider>
        <ServiceWorkerRegister />
      </body>
    </QwikCity>
  );
});
```

Example `src/routes/index.tsx`:

```tsx

import { component$, Resource, useClientEffect$, useStore } from "@builder.io/qwik";
import { useEndpoint } from "@builder.io/qwik-city";
import { QRLSupaBase, useSupabase } from "@thedeveloper/qwik-supabase";
import { createServerSupabaseClient } from "@supabase/auth-helpers-qwik";

export async function googleLogin(e: Event, getSupabase: QRLSupaBase) {
  const supabase = await getSupabase();

  try {
    const { user, session, error } = await supabase.auth.signInWithOAuth({
      provider: 'google'
    });

    console.log(user, session, error);
  } catch (e) {
    console.log(e)
  }
}

export async function onGet(req) {
  const supabaseUrl = import.meta.env.VITE_SUPABASE_URL;
  const supabaseKey = import.meta.env.VITE_SUPABASE_ANON_KEY;
  const opts = { req: req.request, res: req.response };
  const supabase = createServerSupabaseClient(supabaseUrl, supabaseKey, opts);
  return await supabase.auth.getUser();
}

export default component$(() => {
  const getSupabase = useSupabase();
  const store = useStore({ session: {} });

  // invoke supabase on client to process auth cookie
  useClientEffect$(async () => {
    const supabase = await getSupabase();
    const auth = supabase.auth;
    const session = await auth.getSession();
    // do something with session e.g. set on store
    store.session = session;
  });

  const data = useEndpoint();

  return (
    <>
      {JSON.stringify(store)}
      <div onClick$={e => googleLogin(e, getSupabase)}>sign in</div>
      <Resource
        value={data}
        onResolved={data => (
          <div>{JSON.stringify(data)}</div>
        )}
      />
    </>
  )
})

```
