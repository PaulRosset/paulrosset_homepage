---
title: "Setting up Server Side Auth with Supabase and NextJS"
date: 2021-10-12T14:46:15+02:00
draft: false
slug: "technical-writing/next-js-supabase-auth"
description: "Implementing Authentification system with NextJS and Supabase."
keywords:
  [
    "supabase",
    "nextjs",
    "authentification",
    "auth",
    "open source",
    "secure",
    "web",
  ]
toc: true
---

![Supabase X Nextjs](./supabase-x-nextjs.png)

# Introduction

When it comes to choosing your stack for your next SaaS project, NextJS is a must choice in the react ecosystem, NextJS also make a good match with [supabase.com/](https://supabase.com/) an Open source firebase alternative.
In this post, I will show you how to integrate the supabase auth logic in NextJS in an idiomatic way.

# Setting up a custom `App`

irst, we will need to create the `pages/_app.tsx`.
This will allow us to create reusable logic among our App.
In this React Component page, we will use the SDK provided by [supabase](https://github.com/supabase/supabase-js) to handle authentification.

```
// pages/_app.tsx
import { supabase } from "../../utils/supabase";

function MyApp({ Component, pageProps }: AppPropsWithLayout) {
 supabase.auth.onAuthStateChange((event, session) => {
  fetch("/api/auth", {
    method: "POST",
    headers: new Headers({ "Content-Type": "application/json" }),
    credentials: "same-origin",
    body: JSON.stringify({ event, session }),
  });

return (
  <Component {...pageProps} />
 )
}
```

This piece of code will allow performing a POST HTTP request to our future [API route](https://nextjs.org/docs/api-routes/dynamic-api-routes) `pages/api/auth.ts` with metadata `event` and `session` an object that contains key pieces of information (auth cookie) about the user who is logged or not.

We are listening to any changes that could happen related to authentication (sign in, sign out, etc…) thanks to the supabase SDK auth. Once an event happens, we will immediately tell the server-side that the auth state has changed, then the server-side will update his state accordingly.

The good news as said earlier is that supabase is open source, which means that you can check what's going on under the hood. For our use case, the logic happens [here](https://github.com/supabase/gotrue-js/blob/master/src/GoTrueApi.ts#L469).

# Setting up API Route Auth

Once the client is ready to send the `event` and `session` events, we need to create the function server-side that will receive that payload to interpret it.

```
// pages/pages/api/auth.ts
import { NextApiRequest, NextApiResponse } from "next";
import { supabase } from "../../utils/supabase";

export default function handler(req: NextApiRequest, res: NextApiResponse) {
 // Set the auth cookie.
 supabase.auth.api.setAuthCookie(req, res);
}
```

We are using again the `supabase` object that gives access to the set of API related to auth. `setAuthCookie` is using the under the hood the [Set-Cookie HTTP](https://developer.mozilla.org/fr/docs/Web/HTTP/Headers/Set-Cookie) header that allows the user agent to store a cookie that will be used in a near future.

We are already almost done!

# Protecting routes

We now have the ability to protect our route thanks to the [getServerSideProps](https://nextjs.org/docs/basic-features/data-fetching#getserversideprops-server-side-rendering) function of NextJS.

```
// pages/protectedRoute.tsx
import { GetServerSideProps } from "next";
import { supabase } from "../../utils/supabase";

const Protected = ({ user }: IProps) => {
  return <div>JSON.stringify(user)</div>
}

export const getServerSideProps: GetServerSideProps = async ({ req }) => {
 // Get our logged user
 const { user } = await supabase.auth.api.getUserByCookie(req);
 // Check if the user is logged
 if (user === null) {
  // Redirect if no logged in
  return { props: {}, redirect: { destination: "/auth/login" } };
 }
 // If logged return the user
 return { props: { user } };
};
export default Protected;
```

The function is really straightforward, it will look in the cookie of the request if there is any auth cookie available to consume.

If an auth cookie is present, the function will fetch the user associated with the cookie from the backend and return his metadata.

On the other hand, if no auth cookie is detected, the API will not be able to fetch that user and then return `null` value, to indicate it.

Then, once we have our result, we will use the special API that [getServerSideProps](https://nextjs.org/docs/basic-features/data-fetching#getserversideprops-server-side-rendering) is providing, by either redirecting the user to the login page before he could reach the client rendered or making him access to the page asked with the desired props.

Then, that [getServerSideProps](https://nextjs.org/docs/basic-features/data-fetching#getserversideprops-server-side-rendering) function should be replicated to every page your application logic ask for it.
You can even do the inverse, for example when an already authenticated user navigates to the login page. You can redirect him to your main protected page instead.

# Conclusion

It exists either the server-side way of doing it or the client-side way.
It's generally a better approach to do it server-side way whenever possible to avoid a few edge cases where the application could behave wrongly.

Everything on the client side could be reached, so it means we should never trust the client side.
On the other hand, on the server-side, it is way more secure and hard to tweak.

I hope you will find it useful!
