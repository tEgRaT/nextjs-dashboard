# Learn Next.js

## Next.js App Router Course - Starter

This is the starter template for the Next.js App Router Course. It contains the starting code for the dashboard application.

For more information, see the [course curriculum](https://nextjs.org/learn) on the Next.js Website.

## Getting Started

### Creating a new project

```bash
npm i -g pnpm
npx create-next-app@latest nextjs-dashboard --example "https://github.com/vercel/next-learn/tree/main/dashboard/starter-example" --use-pnpm
cd nextjs-dashboard
pnpm i
pnpm dev
```

## CSS Styling

### Global styles

/app/ui/global.css can be used to add CSS rules to all the routes - such as css reset rules, site-wide styles for HTML elements like links, and more.

Then, it can be imported into the top level component - the root layout - `/app/layout.tsx` for this project.

```tsx
// app/layout.tsx
import '@/app/ui/global.css';
```

```css
/* app/ui/global.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## Optimizing Fonts and Images

Next.js automatically optimizes fonts in the application when you use the `next/font` module. It downloads font files at build time and hosts them with your other static assets.

### Adding a font

Create `fonts.ts` in `/app/ui`, then add the following code to it:

```js
import { Inter } from 'next/font/google';

export const inter = Inter({ subsets: ['latin'] });
```

Finally, import the font to the `<body>` element in `/app/layout.tsx`.

### The `<Image>` component

```js
...
import Image from 'next/image';
...
<Image
    src="/hero-desktop.png"
    className="hidden md:block"
    alt="Screenshots of the dashboard project showing desktop version"
    width={10000}
    height={760}
/>
...
```

It's good practice to set the width and height of the images to avoid layout shift, these should be an aspect ratio **identical** to the source image.

The class `hidden` is to remove the image from the DOM on mobile screens, and `md:block` to show the image on desktop screens.

## Creating Layouts and Pages

### Nested routing

Create a new route segment using a folder, and add a `page` file inside it.

```js
// /app/dashboard/page.tsx
export default function Page() {
  return <p>Dashboard Page</p>;
}
```

### Layouts

One benefit of using layouts in Next.js is that on navigation, only the page components update while the layout won't re-render. This is called [partial rendering](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#4-partial-rendering).

A [root layout](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts#root-layout-required) is required.

## Navigating Between Pages

### The `<Link>` component

```js
import Link from 'next/link';

<Link href="/dashboard">Dashboard</Link>;
```

To improve the navigation experience, Next.js automatically code splits the application by segments.

Splitting code by routes means that pages become isolated. If a certain page throws an error, the rest of the application will still work.

Furthermore, in production, whenever `Link` components appear in the browser's viewport, Next.js automatically **prefetches** the code for the linked route in the background.

### Get the current path

```js
import { usePathname } from 'next/navigation';

const pathname = usePathname();
```

Since `usePathname()` is a hook, in must be called inside a Client Component, so we have to add React's "use client" directive at the top of the file.

## Setting Up the Database

In `/app/seed` folder, create a file called `route.ts`, in this file, do the following:

- Connect to the database
- Create Tables, load the data from `/app/lib/placeholder-data.ts` and insert into the tables
- Import bcrypt to hash the passwords when inserting users into the database
- If error occurs, perform `ROLLBACK` to undo the changes

Start the local development server with `pnpm run dev` and navigate to localhost:3000/seed in the browser. Once completed, delete this file.

## Fetching Data

### API layer

APIs are an intermediary layer between the application code and database. Use an API if:

- You're using 3rd party services that provide an API.
- You're fetching data from the client, you want to have an API layer that runs on the server to avoid exposing your database secrets to the client.

### Database queries

When creating a full-stack application, you'll also need to write logic to interact with your database. Use database queries if:

- You need to write logic to interact with your database when creating your API endpoints.
- You are using React Server Components (fetching data on the server), you can skip the API layer, and query your database directly without risking exposing your database secrets to the client.

### Using Server Components to fetch data

Benefits of using Server Components:

- They support promises, so you can use `async/await` syntax without reaching for `useEffect`, `useState` or data fetching libraries.
- They execute on the server, so you can keep expensive data fetches and logic on the server and send only the result to the client.
- You can query the database directly without an additional API layer.

### Parallel data fetching

A common way to avoid waterfalls is to initiate all data requests at the same time - in parallel.

In JavaScript, you can use the `Promise.all()` or `Promise.allSettled()` functions to initiate all promises at the same time.

## Static and Dynamic Rendering

### Static Rendering

With static rendering, data fetching and rendering happens on the server at build time (when you deploy) or when revalidating data.

Whenever a user visits your application, the cached result is served. There are a couple of benefits with static rendering:

- **Faster Websites** - Prerendered content can be cached and globally distributed.
- **Reduced Server Load** - Your server does not have to dynamically generate content for each user request.
- **SEO** - Prerendered content is easier for search engine crawlers to index, as the content is already available when the page loads.

Static rendering is useful for **UI with no data** or **data that is shared across users**, such as a static blog post or a product page.

### Dynamic Rendering

With dynamic rendering, content is rendered on the server for each user at **request time** (when the user visits the page). There are a couple of benefits of dynamic rendering:

- **Real-Time Data** - Dynamic rendering allows your application to display real-time or frequently updated data. This is ideal for applications where data changes often.
- **User-Specific Content** - It's easier to serve personalized content, such as dashboards or user profiles, and update the data based on user interaction.
- **Request Time Information** - Dynamic rendering allows you to access information that can only be known at request time, such as cookies or the url search parameters.

## Streaming

Streaming is a data transfer technique that allows you to break down a route into smaller "chunks" and progressively stream them from the server to the client as they become ready.

By streaming, you can prevent slow data requests from blocking your whole page. This allows the user to see and interact with parts of the page without waiting for all the data to load before any UI can be shown to the user.

Each component can be considered a _chunk_.

There are two ways you implement streaming in Next.js:

1. At the page level, with the `loading.tsx` file.
1. For specific components, with `<Suspence>`.

### Streaming a whole page with `loading.tsx`

```jsx
export default function Loading() {
  return <div>Loading...</div>;
}
```

#### Adding loading skeletons

```tsx
// /app/dashboard/loading.tsx
import DashboardSkeleton from '@/app/ui/skeletons';

export default function Loading() {
  return <DashboardSkeleton />;
}
```

#### [Route Groups](https://nextjs.org/docs/app/building-your-application/routing/route-groups)

Create a new folder called `/(overview)` inside the dashboard folder. Then, move `loading.tsx` and `page.tsx` files inside the folder.

Route groups allow you to organize files into logical groups without affecting the URL path structure.

By moving `loading.tsx` and `page.tsx` into the `/app/dashboard/(overview)` folder, you can ensure `loading.tsx` only applies to the dashboard overview page. However, you can also use route groups to separate the application into sections (e.g. `(marketing)` routes and `(shop)` routes) or by team for larger applications.

#### Streaming a component using `<Suspense />`

```tsx
import { Suspense } from 'react';
import { RevenueChartSkeleton } from '@/app/ui/skeletons';
import { RevenueChart } from '@/app/ui/dashboard/revenue-chart';

export default async function Page() {
    ...
    return (
        ...
        <Suspense fallback={<RevenueChartSkeleton />}>
            <RevenueChart />
        </Suspense>
        ...
    );
}
```

## Partial Prerendering

### What is Partial Prerendering?

Next.js 14 introduced an experimental version of **Partial Prerendering** - a new rendering modal that allows you to combine the benefits of static and dynamic rendering in the same route.

When an user visits a route:

- A static route shell that includes the navbar and product information is served, ensuring a fast initial load.
- The shell leaves holes where dynamic content like the cart and recommended products will load in asynchronously.
- The async holes are streamed in parallel, reducing the overall load time of the page.

### How does Partial Prerendering work?

Partial Prerendering uses React's _Suspense_ to defer rendering parts of your application until some condition is met (e.g. data is loaded).

Wrapping a component in Suspense doesn't make the component itself dynamic, but rather Suspense is used as a boundary between your static and dynamic code.

### Implementing Partial Prerendering

Enable PPR for your Next.js app by adding the `ppr` option to your _next.config.mjs_ file:

```js
/** @type {import('next').NextConfig} */

const nextConfig = {
  experimental: {
    ppr: 'incremental',
  },
};

export default nextConfig;
```

The `'incremental'` value allows you to adopt PPR for specific routes.

Next, add the `experimental_ppr` segment config option to your dashbord layout:

```tsx
import SideNav from "@/app/ui/dashboard/sidenav";

export const experimental_ppr = true;

...
```

That's it. You may not see a difference in your application in development, but you should notice a performance improvement in production. Next.js will prerender the static parts of your route and defer the dynamic parts until the user requests them.

The greating thing about Partial Prerendering is that you don't need to change your code to use it. As long as you're using Suspense to wrap the dynamic parts of your route, Next.js will know which parts of your route are statiic and which are dynamic.

## Adding Searching and Pagination

### Starting code

```tsx
// app/dashboard/layout.tsx
import Pagination from '@/app/ui/invoices/pagination';
import Search from '@/app/ui/search';
import Table from '@/app/ui/invoices/table';
import { CreateInvoice } from '@/app/ui/invoices/buttons';
import { lusitana } from '@/app/ui/fonts';
import { InvoicesTableSkeleton } from '@/app/ui/skeletons';
import { Suspense } from 'react';

export default async function Page() {
  return (
    <div className="w-full">
      <div className="flex w-full items-center justify-between">
        <h1 className={`{lusitana.className} text-2xl`}>Invoices</h1>
      </div>

      <div className="mt-4 flex items-center justify-between gap-2 md:mt-8">
        <Search placeholder="Search Invoices..." />
        <CreateInvoice />
      </div>

      {/* <Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton />}>
                <Table query={query} currentPage={currentPage} />
            </Suspense> */}

      <div className="mt-5 flex w-full justify-center">
        {/* <Pagination totalPages={totalPages} /> */}
      </div>
    </div>
  );
}
```

### Why use URL search params?

- **Bookmarkable and Shareable URLs**
- **Server-Side Rendering and Initial Load**: URL parameters can be directly consumed on the server to render the initial state, making it easier to handle server rendering.
- **Analytics and Tracking**: Having search queries and filters directly in the URL makes it easier to track user behavior without requiring additional client-side logic.

### Adding the search functionality

- **`useSearchParams`** - Allows you to access the parameters of the current URL.
- **`usePathname`** - Lets you read the current URL's pathname.
- **`useRouter`** - Enables navigation between routes within client components programatically.

Here's a quick overview of the implementation step:

1. Capture the user's input.
1. Update the URL with the search params.
1. Keep the URL in sync with the input field.
1. Update the table to reflect the search query.

####

1. #### Capture the user's input

   Create a new `handleSearch` function, and add an `onChange` listener to the `input` element.

   ```tsx
   // app/ui/search.tsx
   'use client'; // This is a Client Component, which means you can use event listeners and hooks

   import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';

   export default function Search({ placeholder }: { placeholder: string }) {
     function handleSearch(term: string) {
       console.log(term);
     }

     return (
       <div className="relative flex flex-1 flex-shrink-0">
         <label htmlFor="search" className="sr-only">
           Search
         </label>
         <input
           className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
           placeholder={placeholder}
           onChange={(e) => {
             handleSearch(e.target.value);
           }}
         />
         <MagnifyingGlassIcon className="absolute left-3 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-gray-500 peer-focus:text-gray-900" />
       </div>
     );
   }
   ```

1. #### Update the URL with the search params

   Import the `useSearchParams` hook from `'next/navigation'`, and assign it to a variable. Inside `handleSearch`, create a new `URLSearchParams` instance using your new `SearchParams` variable.

   `URLSearchParams` is a Web API that provides utility methods for manipulating the URL query parameters. Instead of creating a complex string literal, use can use it to get the params string like `?page=1&query=a`.

   Next, set the params string base on the user's input. If the input is empty, you want to delete it.

   Import `useRouter` and `usePathname` from `'next/navigation'`, and use the `replace` method from `useRouter` inside `handleSearch`.

   ```tsx
   // app/ui/search.tsx
   'use client';

   import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
   import { useSearchParams, useRouter, usePathname } from 'next/navigation';

   export default function Search({ placeholder }: { placeholder: string }) {
       const searchParams = useSearchParams();
       const pathname = usePathname();
       const { replace } = useRouter();

       function handleSearch(term: string) {
           const params = new URLSearchParams(searchParams);

           if (term) {
               params.set('query', term);
           } else {
               params.delete('query');
           }

           replace(`${pathname}?${params.toString()}`);
       }
       // ...
   ```

   - `${pathname}` is the current path, in this case, `"/dashboard/invoices"`.
   - As the user types into the search bar, `params.toString()` translates this input into an URL-friendly format.
   - `` replace(`${pathname}?${params.toString()}`) `` updates the URL with the user's search data. For example, `/dashboard/invoices?query=lee` if the user searches for "Lee".
   - The URL is updated without reloading the page, thanks to Next.js's client-side navigation.

According to Claude 3.5 sonnet in Cursor:

> You're correct that `replace` is a method provided by the `useRouter` hook from Next.js. The `replace` method is used to navigate to a new URL without adding a new entry to the browser's history stack.
>
> In the context of this code:
>
> ```typescript
> replace(`${pathname}?${params.toString()}`);
> ```
>
> This statement does the following:
>
> 1. It constructs a new URL by combining the current `pathname` with the updated query parameters.
> 2. It then replaces the current URL in the browser with this new URL, without creating a new history entry.
>
> More specifically:
>
> - `pathname` is the current path of the page (e.g., "/users" or "/products").
> - `params.toString()` converts the URLSearchParams object into a query string (e.g., "page=1&query=searchterm").
> - The resulting URL might look like "/users?page=1&query=searchterm".
>
> This approach allows the component to update the URL based on the search input without triggering a full page reload. It's particularly useful for implementing features like search functionality where you want the URL to reflect the current search state, allowing for shareable links and browser history navigation.

1. #### Keeping the URL and input in sync

   To ensure the input field is in sync with the URL and will be populated when sharing, pass a `defaultValue` to the `input` element by reading from `searchParams`.

   ```tsx
   // app/ui/search.tsx
               // ...
               <input
                   className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
                   placeholder={placeholder}
                   onChange={e => {
                       handleSearch(e.target.value);
                   }}
                   defaultValue={searchParams.get('query')?.toString()}
                   // ...
   ```

   `defaultValue` vs. `value` / Controlled vs. Uncontrolled

   If you're using state to manage the value of an input, you'd use the `value` attribute to make it a controlled component. This means React would manage the input's state.

   However, since you're not using state, you can use `defaultValue`. This means the native input will manage its own state. This is ok since you're saving the search query to the URL instead of state.

1. #### Updating the table

   The invoices page components accept a prop called `searchParams`, so you can pass the current URL params to the `<Table>` component.

   ```tsx
   // app/dashboard/invoices/page.tsx
   import Pagination from '@/app/ui/invoices/pagination';
   import Search from '@/app/ui/search';
   import Table from '@/app/ui/invoices/table';
   import { CreateInvoice } from '@/app/ui/invoices/buttons';
   import { lusitana } from '@/app/ui/fonts';
   import { InvoicesTableSkeleton } from '@/app/ui/skeletons';
   import { Suspense } from 'react';

   export default async function Page({
     searchParams,
   }: {
     searchParams?: {
       query?: string;
       page?: string;
     };
   }) {
     const query = searchParams?.query || '';
     const currentPage = Number(searchParams?.page) || 1;

     return (
       <div className="w-full">
         <div className="flex w-full items-center justify-between">
           <h1 className={`{lusitana.className} text-2xl`}>Invoices</h1>
         </div>

         <div className="mt-4 flex items-center justify-between gap-2 md:mt-8">
           <Search placeholder="Search Invoices..." />
           <CreateInvoice />
         </div>

         <Suspense
           key={query + currentPage}
           fallback={<InvoicesTableSkeleton />}
         >
           <Table query={query} currentPage={currentPage} />
         </Suspense>

         <div className="mt-5 flex w-full justify-center">
           {/* <Pagination totalPages={totalPages} /> */}
         </div>
       </div>
     );
   }
   ```

   **When to use the `useSearchParams()` hook vs. the `searchParams` prop?**

   - `<Search>` is a Client Component, so you used the `useSearchParams()` hook to access the params from the client.

   - `<Table>` is a Server Component that fetches its own data, so you can pass the `searchParams` prop from the page to the component.

## Best practice: Debouncing

```bash
pnpm i use-debounce
```

```tsx
//...
// app/ui/search.tsx
import { useDebouncedCallback } from 'use-debounce';

export default function Search({ placeholder }: { placeholder: string }) {
    const searchParams = useSearchParams();
    const pathname = usePathname();
    const { replace } = useRouter();

    const handleSearch = useDebouncedCallback((term: string) => {
        console.log(`Searching... ${term}`);
        const params = new URLSearchParams(searchParams);

        if (term) {
            params.set('query', term);
        } else {
            params.delete('query');
        }

        replace(`${pathname}?${params.toString()}`);
    }, 300);
    //...
```

## Adding pagination

Navigate to the `<Pagination />` component and you'll notice that it's a Client Component. You don't want to fetch data on the client as this would expose your database secrets (remember, you're not using an API layer). Instead, you can fetch the data on the server, and pass it to the component as a prop.

Next, pass the `totalPages` prop to the `<Pagination />` component.

```tsx
// app/dashboard/invoices/page.tsx
// ...
import { fetchInvoicesPages } from '@/app/lib/data';

export default async function Page({
  searchParams,
}: {
  searchParams?: {
    query?: string;
    page?: string;
  };
}) {
  const query = searchParams?.query || '';
  const currentPage = Number(searchParams?.page) || 1;

  const totalPages = await fetchInvoicesPages(query);

  return (
    <div className="w-full">
      <div className="flex w-full items-center justify-between">
        <h1 className={`{lusitana.className} text-2xl`}>Invoices</h1>
      </div>

      <div className="mt-4 flex items-center justify-between gap-2 md:mt-8">
        <Search placeholder="Search Invoices..." />
        <CreateInvoice />
      </div>

      <Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton />}>
        <Table query={query} currentPage={currentPage} />
      </Suspense>

      <div className="mt-5 flex w-full justify-center">
        <Pagination totalPages={totalPages} />
      </div>
    </div>
  );
}
```

Navigate to the `<Pagination />` component and import the `usePathname` and `useSearchParams` hooks.

Next, create a new function inside the `<Pagination />` Component called `createPageURL`.

According to Claude 3.5 sonnet in Cursor:

> The `useSearchParams()` and `new URLSearchParams()` serve different purposes in the context of Next.js and web development:
>
> 1. `useSearchParams()`:
>
> - This is a Next.js hook used in client-side components.
> - It provides access to the current URL's query parameters.
> - It's reactive, meaning it will update if the URL changes without a full page reload.
> - Used to read the current search params.
>
> 2. `new URLSearchParams()`:
>    - This is a built-in Web API constructor.
>    - It creates a new URLSearchParams object, which is useful for manipulating query parameters.
>    - It's not specific to Next.js and can be used in any JavaScript environment.
>    - Used to create or modify search params.
>
> In the code:
>
> ```typescript
> const searchParams = useSearchParams();
> ```
>
> This line gets the current search parameters from the URL.
>
> ```typescript
> const params = new URLSearchParams(searchParams);
> ```
>
> This line creates a new URLSearchParams object, which can be modified without affecting the current URL.
>
> The combination allows you to read the current params and then create a new set of params based on them, which can be modified and used to generate new URLs without changing the current page's URL directly.

Finally, when the user types a new search query, you want to reset the page number to 1. You can do this by updating the `handleSearch` function in your `<Search />` component:

```tsx
// app/ui/search.tsx
'use client';

import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
import { useSearchParams, useRouter, usePathname } from 'next/navigation';
import { useDebouncedCallback } from 'use-debounce';

export default function Search({ placeholder }: { placeholder: string }) {
    const searchParams = useSearchParams();
    const pathname = usePathname();
    const { replace } = useRouter();

    const handleSearch = useDebouncedCallback((term: string) => {
        const params = new URLSearchParams(searchParams);

        params.set('page', '1');

        if (term) {
            params.set('query', term);
        } else {
            params.delete('query');
        }

        replace(`${pathname}?${params.toString()}`);
    }, 300);
    // ...
```

To make the project working with no errors, we also need to add the `allPages` variable and it's assignment statement to the `<Pagination />` component.

```tsx
// app/ui/invoices/pagination.tsx
// ...
export default function Pagination({ totalPages }: { totalPages: number }) {
    const pathname = usePathname();
    const searchParams = useSearchParams();
    const currentPage = Number(searchParams.get('page') || 1);
    const allPages = generatePagination(currentPage, totalPages);
    // ...
```

## Mutating Data

### What are Server Actions?

React Server Actions allow you to run asynchronous code directly on the server. They eliminate the need to create API endpoints to mutate your data. Instead, you write asynchronous functions that execute on the server and can be invoked from your Client or Server Components.

### Using forms with Server Actions

In React, you can use the `action` attribute in the `<form>` element to invoke actions. The action will automatically receive the native `FormData` object, containing the captured data.

For example:

```tsx
// Server Component
export default function Page() {
  // Action
  async function create(formData: FormData) {
    'use server';

    // Logic to mutate data...
  }

  // Invoke the action using the `action` attribute
  return <form action={create}>...</form>;
}
```

An advantage of invoking a Server Action within a Server Component is progressive enhancement - forms work even if JavaScript is disabled on the client.

### Next.js with Server Actions

Server Actions are also deeply integrated with Next.js caching. When a form is submitted through a Server Action, not only can you use the action to mutate data, but you can also revalidate the associated cache using APIs like `revalidatePath` and `revalidateTag`.

### Creating an invoice

1. #### Create a new route and form

   Inside the `/invoices` folder, add a new route segment called `/create` with a `page.tsx` file.

   ```tsx
   // app/dashboard/invoices/create/page.tsx
   import Form from '@/app/ui/invoices/create-form';
   import Breadcrumbs from '@/app/ui/invoices/breadcrumbs';
   import { fetchCustomers } from '@/app/lib/data';

   export default async function Page() {
     const customers = await fetchCustomers();

     return (
       <main>
         <Breadcrumbs
           breadcrumbs={[
             { label: 'Invoices', href: '/dashboard/invoices' },
             {
               label: 'Create Invoice',
               href: '/dashboard/invoices/create',
               active: true,
             },
           ]}
         />
         <Form customers={customers} />
       </main>
     );
   }
   ```

   Your page is a Server Component that fetches `customers` and passes it to the `<Form />` component.

1. #### Create a Server Action

   Now let's create a Server Action that is going to be called when the form is submitted.

   Navigate to your `lib` directory and create a new file named `actions.ts`. At the top of this file, add the React `'use server'` directive.

   By adding the `'use server'`, you mark all the exported funcitons within the file as Server Actions. These server functions can then be imported and used in Client and Server Components.

   You can also write Server Actions directly inside Server Components by adding `'use server'` inside the action. But for this course, we'll keep them all organized in a separate file.

   ```ts
   // app/lib/actions.ts
   'use server';

   export async function createInvoice(formData: FormData) {}
   ```

   Then, in your `<Form />` component, import the `createInvoice` from your `actions.ts` file. Add an `action` attribute to the `<form>` element, and call the `createInvoice` action.

   ```tsx
   // app/ui/invoice/create-form.tsx
   // ...
   import { createInvoice } from "@/app/lib/actions";

   export default function Form({ customers }: { customers: CustomerField[] }) {
   return (
       <form action={createInvoice}>
       {/* ... */}
   ```

   **Good to know**: In HTML, you'd pass an URL to the `action` attribute. This URL would be the destination where your form data shoud be submitted (usually an API endpoint).

   However, in React, the `action` attribute is considered a special prop - meaning React builds on top of it to allowed actions to be invoked.

   Behind the scenes, Server Actions create a `POST` API endpoint. This is why you don't need to create API endpoints manually when using Server Actions.

1. #### Extract the data from `formData`

   Back in your `actions.ts`. There are a [couple of methods](https://developer.mozilla.org/en-US/docs/Web/API/FormData/append) you can use to extract the values from `formData`. Here, let's use the `.get(name)` method.

   ```ts
   // app/lib/actions.ts
   'use server';

   export async function createInvoice(formData: FormData) {
     const rawFormData = {
       customerId: formData.get('customerId'),
       amount: formData.get('amount'),
       status: formData.get('status'),
     };

     console.log(rawFormData);
   }
   ```

   **Tip**: If you're working with forms that have many fields, you may want to consider using the `entries()` method with JavaScript's `Object.fromEntries()`. For example:

   ```ts
   const rawFormData = Object.fromEntries(formData.entries());
   ```

1. #### Validate and prepare the data

   ##### Type validation and coercion

   To handle type validation, you have a few options. While you can manually validate types, using a type validation library can save your time and effort. For this example, we'll use the [zod](https://zod.dev/), a TypeScript-first validation library that can simplify this task for you.

   In your `actions.ts` file, import Zod and define a schema that matches the shape of your form object. This schema will validate the `formData` before saving it to a database.

   ```ts
   // app/lib/actions.ts
   'use server';

   import { z } from 'zod';

   const FormSchema = z.object({
     id: z.string(),
     customerId: z.string(),
     amount: z.coerce.number(),
     status: z.enum(['pending', 'paid']),
     date: z.string(),
   });

   const CreateInvoice = FormSchema.omit({ id: true, date: true });

   export async function createInvoice(formData: FormData) {
     // ...
   }
   ```

   You can then pass your `rawFormData` to `CreateInvoice` to validate the types:

   ```ts
   // app/lib/actions.ts
   export async function createInvoice(formData: FormData) {
     const { customerId, amount, status } = CreateInvoice.parse({
       customerId: formData.get('customerId'),
       amount: formData.get('amount'),
       status: formData.get('status'),
     });
   }
   ```

   ##### Storing values in cents

   It's usually good practice to store monetary values in cents in your database to eliminate JavaScript floating-point errors and ensure greater accuracy.

   ```ts
   // app/lib/actions.ts
   // ...
   export async function createInvoice(formData: FormData) {
     const { customerId, amount, status } = CreateInvoice.parse({
       customerId: formData.get('customerId'),
       amount: formData.get('amount'),
       status: formData.get('status'),
     });

     const amountInCents = amount * 100;
   }
   // ...
   ```

   ##### Creating new dates

   Finally, let's create a new date with the format "YYYY-MM-DD" for the invoices creation date:

   ```ts
   // app/lib/actions.ts
   // ...
   const amountInCents = amount * 100;
   const date = new Date().toISOString().split('T')[0];
   // ...
   ```

1. #### Inserting the data into your database

   ```ts
   'use server';

   import { z } from 'zod';
   import { sql } from '@vercel/postgres';

   const FormSchema = z.object({
     id: z.string(),
     customerId: z.string(),
     amount: z.coerce.number(),
     status: z.enum(['pending', 'paid']),
     date: z.string(),
   });

   const CreateInvoice = FormSchema.omit({ id: true, date: true });

   export async function createInvoice(formData: FormData) {
     const { customerId, amount, status } = CreateInvoice.parse({
       customerId: formData.get('customerId'),
       amount: formData.get('amount'),
       status: formData.get('status'),
     });

     const amountInCents = amount * 100;
     const date = new Date().toISOString().split('T')[0];

     await sql`
       INSERT INTO invoices
         (customer_id, amount, status, date)
       VALUES
         (${customerId}, ${amountInCents}, ${status}, ${date})
     `;
   }
   ```

1. #### Revalidate and redirect

   Nest.js has a [Client-side Router Cache](https://nextjs.org/docs/app/building-your-application/caching#router-cache) that stores the route segments in the user's browser for a time. Along with [prefetching](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#1-prefetching), this cache ensures that users can quickly navigate between routes while reducing the number of requests made to the server.

   Since you're updating the data displayed in invoices route, you want to clear this cache and trigger a new request to the server. You can do this with the [`revalidatePath`](https://nextjs.org/docs/app/api-reference/functions/revalidatePath) function from Next.js:

   ```ts
   // app/lib/actions.ts
   'use server';

   import { z } from 'zod';
   import { sql } from '@vercel/postgres';
   import { revalidatePath } from 'next/cache';

   const FormSchema = z.object({
     id: z.string(),
     customerId: z.string(),
     amount: z.coerce.number(),
     status: z.enum(['pending', 'paid']),
     date: z.string(),
   });

   const CreateInvoice = FormSchema.omit({ id: true, date: true });

   export async function createInvoice(formData: FormData) {
     const { customerId, amount, status } = CreateInvoice.parse({
       customerId: formData.get('customerId'),
       amount: formData.get('amount'),
       status: formData.get('status'),
     });

     const amountInCents = amount * 100;
     const date = new Date().toISOString().split('T')[0];

     await sql`
       INSERT INTO invoices
         (customer_id, amount, status, date)
       VALUES
         (${customerId}, ${amountInCents}, ${status}, ${date})
     `;

     revalidatePath('/dashboard/invoices');
   }
   ```

   Once the database has been updated, the `/dashboard/invoices` path will be revalidated, and fresh data will be fetched from the server.

   At this point, you also want to redirect the user back to the `/dashboard/invoices` page. You can do this with the [`redirect`](https://nextjs.org/docs/app/api-reference/functions/redirect) function from Next.js:

   ```ts
   // app/lib/actions.ts
   'use server';

   import { z } from 'zod';
   import { sql } from '@vercel/postgres';
   import { revalidatePath } from 'next/cache';
   import { redirect } from 'next/navigation';

   const FormSchema = z.object({
     id: z.string(),
     customerId: z.string(),
     amount: z.coerce.number(),
     status: z.enum(['pending', 'paid']),
     date: z.string(),
   });

   const CreateInvoice = FormSchema.omit({ id: true, date: true });

   export async function createInvoice(formData: FormData) {
     const { customerId, amount, status } = CreateInvoice.parse({
       customerId: formData.get('customerId'),
       amount: formData.get('amount'),
       status: formData.get('status'),
     });

     const amountInCents = amount * 100;
     const date = new Date().toISOString().split('T')[0];

     await sql`
        INSERT INTO invoices
          (customer_id, amount, status, date)
        VALUES
          (${customerId}, ${amountInCents}, ${status}, ${date})
      `;

     revalidatePath('/dashboard/invoices');
     redirect('/dashboard/invoices');
   }
   ```

### Updating an invoice

1. #### Create a Dynamic Route Segment with the invoice `id`

   Next.js allows you to create [Dynamic Route Segments](https://nextjs.org/docs/app/building-your-application/routing/dynamic-routes) when you don't know the exact segment name and want to create routes based on data. You can create dynamic route segments by wrapping a folder's name in square brackets.

   In your `/invoices` folder, create a new dynamic route called `[id]`, then a new route called `edit` with a `page.tsx` file.

   In your `<Table>` component, notice there's a `<UpdateInvoice />` button that receives the invoice's `id` from the table records.

   Navigate to your `<UpdateInvoice />` component, and update the `href` of the `Link` to accept the `id` prop.

   ```tsx
   // app/ui/invoices/buttons.tsx
   // ...
   export function UpdateInvoice({ id }: { id: string }) {
     return (
       <Link
         href={`/dashboard/invoices/${id}/edit`}
         className="rounded-md border p-2 hover:bg-gray-100"
       >
         <PencilIcon className="w-5" />
       </Link>
     );
   }
   // ...
   ```

1. #### Read the invoice `id` from page `params`

   Back on your `<Page>` component.

   In addition to `searchParams`, page components also accept a prop called `params` which you can use to access the `id`.

   ```tsx
   // app/dashboard/invoices/[id]/edit/page.tsx
   // ...
   export default async function Page({ params }: { params: { id: string } }) {
     const { id } = params;
     // ...
   ```

1. #### Fetch the specific invoice

   Then:

   - Import a new function called `fetchInvoiceById` and pass the `id` as an argument.
   - Import `fetchCustomers` to fetch the customer names for the dropdown.

   You can use `Promise.all` to fetch both the invoice and customers in parallel.

   ```tsx
   // app/dashboard/invoices/[id]/edit/page.tsx
   // ...
   export default async function Page({ params }: { params: { id: string } }) {
     const { id } = params;
     const [invoice, customers] = await Promise.all([
       fetchInvoiceById(id),
       fetchCustomers(),
     ]);
    // ...
   ```

1. #### Pass the `id` to the Server Action

   Lastly, you want to pass the `id` to the Server Action so you can update the right record in your database. You **cannot** pass the `id` as an argument like so:

   ```tsx
   // app/ui/invoices/edit-form.tsx
   // Passing an id as argument won't work
   <form action={updateInvoice(id)}>
   ```

   Instead, you can pass `id` to the Server Action using JS `bind`. This will ensure that any value passed to the Server Action are encoded.

   ```tsx
   // app/ui/invoices/edit-form.tsx
   // ...
    import { updateInvoice } from "@/app/lib/actions";

    export default function EditInvoiceForm({
      invoice,
      customers,
    }: {
      invoice: InvoiceForm;
      customers: CustomerField[];
    }) {
      const updateInvoiceWithId = updateInvoice.bind(null, invoice.id);

      return (
        <form action={updateInvoiceWithId}>
        {/* ... */}
   ```

   Then, in your `actions.ts` file, create a new action, `updateInvoice`:

   ```ts
   // app/lib/actions.ts
   // ...
   // Use Zod to update the expected types
   const UpdateInvoice = FormSchema.omit({ id: true, date: true });

   export async function updateInvoice(id: string, formData: FormData) {
     const { customerId, amount, status } = UpdateInvoice.parse({
       customerId: formData.get('customerId'),
       amount: formData.get('amount'),
       status: formData.get('status'),
     });

     const amountInCents = amount * 100;

     await sql`
           UPDATE invoices
           SET
               customer_id = ${customerId},
               amount = ${amountInCents},
               status = ${status}
           WHERE
               id = ${id}
       `;

     revalidatePath('/dashboard/invoices');
     redirect('/dashboard/invoices');
   }
   ```

   Similarly to the `createInvoice` action, here you are:

   1. Extracting the data from `formData`.
   1. Validating the types with Zod.
   1. Converting the amount to cents.
   1. Passing the variables to the SQL query.
   1. Calling `revalidatePath` to clear the client cache and make a new server request.
   1. Calling `redirect` to redirect the user to the invoice's page.

### Deleting an invoice

To delete an invoice using a Server Action, wrap the delete button in a `<form>` element and pass the `id` to the Server Action using `bind`:

```tsx
// app/ui/invoices/buttons.tsx
// ...
import { deleteInvoice } from '@/app/lib/actions';

// ...

export function DeleteInvoice({ id }: { id: string }) {
  const deleteInvoiceWithId = deleteInvoice.bind(null, id);

  return (
    <form action={deleteInvoiceWithId}>
      <button className="rounded-md border p-2 hover:bg-gray-100">
        <span className="sr-only">Delete</span>
        <TrashIcon className="w-5" />
      </button>
    </form>
  );
}
```

Inside your `actions.ts` file, create a new action called `deleteInvoice`:

```ts
// app/lib/actions.ts
// ...
export async function deleteInvoice(id: string) {
  await sql`DELETE FROM invoices WHERE id = ${id}`;
  revalidatePath('/dashboard/invoice');
}
```

Since this action is being called in the `/dashboard/invoices` path, you don't neet to call `redirect`. Calling `revalidatePath` will trigger a new server request and re-render the table.

## Handling Errors

### Adding `try/catch` to Server Actions

```ts
// app/lib/actions.ts
'use server';

import { z } from 'zod';
import { sql } from '@vercel/postgres';
import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';

const FormSchema = z.object({
  id: z.string(),
  customerId: z.string(),
  amount: z.coerce.number(),
  status: z.enum(['pending', 'paid']),
  date: z.string(),
});

const CreateInvoice = FormSchema.omit({ id: true, date: true });

export async function createInvoice(formData: FormData) {
  const { customerId, amount, status } = CreateInvoice.parse({
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  });

  const amountInCents = amount * 100;
  const date = new Date().toISOString().split('T')[0];

  try {
    await sql`
    INSERT INTO invoices
      (customer_id, amount, status, date)
    VALUES
      (${customerId}, ${amountInCents}, ${status}, ${date})
  `;
  } catch (error) {
    return {
      messge: 'Database Error: Failed to Create Invoice.',
    };
  }

  revalidatePath('/dashboard/invoices');
  redirect('/dashboard/invoices');
}

// Use Zod to update the expected types
const UpdateInvoice = FormSchema.omit({ id: true, date: true });

export async function updateInvoice(id: string, formData: FormData) {
  const { customerId, amount, status } = UpdateInvoice.parse({
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  });

  const amountInCents = amount * 100;

  try {
    await sql`
        UPDATE invoices
        SET
            customer_id = ${customerId},
            amount = ${amountInCents},
            status = ${status}
        WHERE
            id = ${id}
    `;
  } catch (error) {
    return {
      messge: 'Database Error: Failed to Update Invoice.',
    };
  }

  revalidatePath('/dashboard/invoices');
  redirect('/dashboard/invoices');
}

export async function deleteInvoice(id: string) {
  try {
    await sql`DELETE FROM invoices WHERE id = ${id}`;
    revalidatePath('/dashboard/invoice');
  } catch (error) {
    return {
      messge: 'Database Error: Failed to Delete Invoice.',
    };
  }
}
```

Note how `redirect` is being called outside of the `try/catch` block. This is because `redirect` works by throwing an error, which would be caught by the `catch` block.

You also want to show errors to the user to avoid an abrupt failure and allow your application to continue running.

This is where Next.js [`error.tsx`](https://nextjs.org/docs/app/api-reference/file-conventions/error) file comes in.

### Handling all errors with `error.tsx`

The `error.tsx` file can be used to define an UI boundary for a route segment. It serves as a **catch-all** for unexpected errors and allows you to display a fallback UI to your users.

Inside your `/dashboard/invoices` folder, create a new file called `error.tsx`, then enter the following code:

```tsx
// app/dashboard/invoices/error.tsx
'use client';

import { useEffect } from 'react';

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    // Optionally log the error to an error reporting service
    console.error(error);
  }, [error]);

  return (
    <main className="flex h-full flex-col items-center justify-center">
      <h2 className="text-center">Something went wrong!</h2>

      <button
        className="mt-4 rounded-md bg-blue-500 px-4 py-2 text-sm text-white transition-colors hover:bg-blue-400"
        onClick={
          // Attempt to recover by trying to re-render the invoices route
          () => reset()
        }
      >
        Try again
      </button>
    </main>
  );
}
```

There are a few things you'll notice about the code above:

- **'use client'** - `error.tsx` needs to be a Client Component.
- It accepts two props:
  - `error`: This object is an instance of JavaScript's native `Error` object.
  - `reset`: This is a function to reset the error boundary. When executed, the function will try to re-render the route segment.

### Handling 404 errors with the `notFound` function

While `error.tsx` is useful for catching **all** errors, `notFound` can be used when you try to fetch a resource that doesn't exist.

Navigate to `/dashboard/invoices/[id]/edit/page.tsx`, and import `{ notFound }` from 'next/navigation'.

```tsx
// app/dashboard/invoices/[id]/edit/page.tsx
// ...
import { notFound } from 'next/navigation';

export default async function Page({ params }: { params: { id: string } }) {
  const { id } = params;
  const [invoice, customers] = await Promise.all([
    fetchInvoiceById(id),
    fetchCustomers(),
  ]);

  if (!invoice) {
    notFound();
  }
  // ...
```

Create a `not-found.tsx` file inside the `edit` folder, and enter the following code:

```tsx
// app/dashboard/invoices/[id]/edit/not-found.tsx
import Link from 'next/link';
import { FaceFrownIcon } from '@heroicons/react/24/outline';

export default function NotFound() {
  return (
    <main className="flex h-full flex-col items-center justify-center gap-2">
      <FaceFrownIcon className="w-10 text-gray-400" />

      <h2 className="text-xl font-semibold">404 Not Found</h2>

      <p>Could not find the requested invoice.</p>

      <Link
        href="/dashboard/invoices"
        className="mt-4 rounded-md bg-blue-500 px-4 py-2 text-sm text-white transition-colors hover:bg-blue-400"
      >
        Go Back
      </Link>
    </main>
  );
}
```

That's something to keep in mind, `notFound` will take precedence over `error.tsx`, so you can reach out for it when you want to handle more specific errors!

## Improving Accessibility

### Using the ESLint accessibility plugin in Next.js

Next.js includes the `eslint-plugin-jsx-a11y` plugin in its ESLint config to help catch accessibility issues early. Optionally, if you would like to try this out, add `next lint` as a script in your `package.json` file:

```json
  "scripts": {
    "build": "next build",
    "dev": "next dev",
    "start": "next start",
    "lint": "next lint"
  },
```

Then run `pnpm lint` in your terminal:

```bash
pnpm lint
```

### Improving form accessibility

- Semantic HTML
- Labelling
- Focus Outline

### Form validation

#### Client-Side validation

There are a couple of ways you can validate forms on the client. The simplest would be to rely on the form validation provided by the browser by adding the `required` attribute to the `<input>` and `<select>` elements in your forms.

#### Server-Side validation

By validating forms on the server, you can:

- Ensure your data is in the expected format before sending it to the database.
- Reduce the risk of malicious users bypassing client-side validation.
- Have one source of truth for what is considered _valid_ data.

In your `create-form.tsx` component, import `useActionState` hook from `react`. Since `useActionState` is a hook, you will need to turn your form into a Client Component using `'use client'` directive:

```tsx
// app/ui/invoices/create-form.tsx
'use client';

import { CustomerField } from '@/app/lib/definitions';
import Link from 'next/link';
import {
  CheckIcon,
  ClockIcon,
  CurrencyDollarIcon,
  UserCircleIcon,
} from '@heroicons/react/24/outline';
import { Button } from '@/app/ui/button';
import { createInvoice, State } from '@/app/lib/actions';
import { useActionState } from 'react';

export default function Form({ customers }: { customers: CustomerField[] }) {
  const initialState: State = {
    message: null,
    errors: {},
  };

  const [state, formAction] = useActionState(createInvoice, initialState);

  return (
    <form action={formAction}>
    // ...
```

In your `action.ts` file, you can use Zod to validate form data. Update your `FormSchema` as follows:

```ts
// app/lib/actions.ts
// ...
const FormSchema = z.object({
  id: z.string(),
  customerId: z.string({
    invalid_type_error: 'Please select a customer.',
  }),
  amount: z.coerce
    .number()
    .gt(0, { message: 'Please enter an amount greater than $0.' }),
  status: z.enum(['pending', 'paid'], {
    invalid_type_error: 'Please select an invoice status.',
  }),
  date: z.string(),
});
// ...
```

Next, update your `createInvoice` action to accept two parameters - `preState` and `formData`:

```ts
// app/lib/actions.ts
// ...
export type State = {
    errors?: {
        customerId?: string[];
        amount?: string[];
        status?: string[];
    };
    message?: string | null;
};

export async function createInvoice(prevState: State, formData: FormData) {
  // ...
```

- `prevState` - contains the state passed from the `useActionState` hook. You won't be using it in the action in this example, but it's a required prop.

Then, change the Zod `parse()` function to `safeParse()`:

```ts
// app/lib/actions.ts
// ...
export async function createInvoice(id: string, formData: FormData) {
  const validatedFields = UpdateInvoice.safeParse({
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  });
// ...
```

`safeParse()` will return an object containing either a `success` or `error` field. This will help handle validation more gracefully without having put this logic inside the `try/catch` block.

Before sending the information to your database, check if the form fields were validated correctly with a conditional:

```ts
// app/lib/actions.ts
// ...
export async function createInvoice(id: string, formData: FormData) {
  // Validate form fields using Zod
  const validatedFields = UpdateInvoice.safeParse({
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  });

  // If form validation fails, return error early. Otherwise, continue.
  if (!validatedFields.success) {
    return {
      errors: {
        errors: validatedFields.error.flatten().fieldErrors,
        message: 'Missing Fields. Failed to Create Invoices.',
      },
    };
  }
  // ...
```

Finally, since you're handling form validation separately, outside your `try/catch` block, you can return a specific message for any database errors, your final code should look like this:

```ts
// app/lib/actions.ts
// ...
export async function createInvoice(prevState: State, formData: FormData) {
  // Validate form using Zod
  const validatedFields = CreateInvoice.safeParse({
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  });

  // If form validation fails, return errors early. Otherwise, continue.
  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
      message: 'Missing Fields. Failed to Create Invoice.',
    };
  }

  // Prepare data for insertion into the database
  const { customerId, amount, status } = validatedFields.data;
  const amountInCents = amount * 100;
  const date = new Date().toISOString().split('T')[0];

  // Insert data into the database
  try {
    await sql`
    INSERT INTO invoices
      (customer_id, amount, status, date)
    VALUES
      (${customerId}, ${amountInCents}, ${status}, ${date})
  `;
  } catch (error) {
    // If a database error occurs, return a more specific error
    return {
      messge: 'Database Error: Failed to Create Invoice.',
    };
  }

  revalidatePath('/dashboard/invoices');
  redirect('/dashboard/invoices');
}
// ...
```

Now, let's display the errors in your component. Back in the `create-form.tsx` component, you can access the errors using the form `state`.

Add a **ternary operator** that checks for each specific error. For example, after the customer's field, you can add:

```tsx
// app/ui/invoices/create-form.tsx
// ...
return (
    <form action={formAction}>
      <div className="rounded-md bg-gray-50 p-4 md:p-6">
        {/* Customer Name */}
        <div className="mb-4">
          <label htmlFor="customer" className="mb-2 block text-sm font-medium">
            Choose customer
          </label>
          <div className="relative">
            <select
              id="customer"
              name="customerId"
              className="peer block w-full cursor-pointer rounded-md border border-gray-200 py-2 pl-10 text-sm outline-2 placeholder:text-gray-500"
              defaultValue=""
              aria-describedby="customer-error"
            >
              <option value="" disabled>
                Select a customer
              </option>
              {customers.map((customer) => (
                <option key={customer.id} value={customer.id}>
                  {customer.name}
                </option>
              ))}
            </select>
            <UserCircleIcon className="pointer-events-none absolute left-3 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-gray-500" />
          </div>

          <div id="customer-error" aria-live="polite" aria-atomic="true">
            {state.errors?.customerId &&
              state.errors.customerId.map((error: string) => (
                <p className="mt-2 text-sm text-red-500" key={error}>
                  {error}
                </p>
              ))}
          </div>
        </div>
        {/* ... */}
```

In the code above, you're also adding the following aria labels:

- `aria-describedby="customer-error"` - This establishes a relationship between the `select` element and the error message container. It indicates that the container with `id="customer-error"` describes the `select` element. Screen readers will read this description when the user interacts with the `select` box to notify them of errors.
- `id="customer-error"` - This `id` attribute uniquely identifies the HTML element that holds the error message for the `select` input. This is necessary for `aria-describedby` to establish the relationship.
- `aria-live="polite"` - The screen reader should politely notify the user when the error inside the `div` is updated. When the content changes (e.g. when an user corrects an error), the screen reader will announce these changes, but only when the user is idle so as not to interrupt them.

### Adding aria labels

Using the example above, add errors to your remaining form fields. You should also show a message at the bottom of the form if any fields are missing. Once you're ready, run `pnpm lint` to check if you're using the aria labels correctly.

Next, add form validation to the `edit-form.tsx` component.

You'll need to:

- Add `useActionState` to the `edit-form.tsx` component.
- Edit the `updateInvoice`action to handle validation errors from Zod.
- Display the errors in your component, and add arial labels to improve accessibility.

## Adding Authentication

### What is authentication?

#### Authentication vs. Authorization

In web development, authentication and authorization serves different roles:

- **Authentication** is about making sure the user is who they say ther are. You're proving your identity with something you have like a username and password.
- **Authorization** is the next step. Once an user's identity is confirmed, authorization decides what parts of the application they are allowed to use.

So, authentication checks who you are, and authorization determines what you can do or access in the application.

### Creating the login route

Start by creating a new route in your application called `/login`:

```tsx
// app/login/page.tsx
import AcmeLogo from '@/app/ui/acme-logo';
import LoginForm from '@/app/ui/login-form';

export default function LoginPage() {
  return (
    <main className="flex items-center justify-center md:h-screen">
      <div className="relative mx-auto flex w-full max-w-[400px] flex-col space-y-2.5 p-4 md:-mt-32">
        <div className="flex h-20 w-full items-end rounded-lg bg-blue-500 p-3 md:h-36">
          <div className="w-32 text-white md:w-36">
            <AcmeLogo />
          </div>
        </div>
        <LoginForm />
      </div>
    </main>
  );
}
```

### NextAuth.js

[NextAuth.js](https://nextjs.authjs.dev/) abstracts away much of the complexity involved in managing sessions, sign-in and sign-out, and other aspects of authentication.

### Setting up NextAuth.js

```bash
pnpm i next-auth@beta
```

Here, you're installing the `beta` version of NextAuth.js, which is compatible with Next.js 14.

Next, generate a secret key for your application. This key is used to encrypt cookies, ensuring the security of user sessions:

```bash
openssl rand -base64 32
```

Then, in your `.env` file, add your generated key to the `AUTH_SECRET` variable:

```bash
AUTH_SECRET=your-secret-key
```

For auth to work in production, you'll need to update your environment variables in your Vercel project too.

### Adding the pages option

Create an `auth.config.ts` file at the root of our project that export an `authConfig` object. This object will contain the configuration options for NextAuth.js. For now, it will only contain the `pages` option:

```ts
// auth.config.ts
import type { NextAuthConfig } from 'next-auth';

export const authConfig = {
  pages: {
    signIn: '/login',
  },
} satisfies NextAuthConfig;
```

You can use the `pages` option to specify the route for custom sign-in, sign-out, and error pages. This is not required, but by adding `signIn: '/login'` into your `pages` option, the user will be redirected to our custom login page, rather than the NextAuth.js default page.

### Protecting your routes with Next.js Middleware

Next, add the logic to protect your routes. This will prevent users from accessing the dashboard pages unless they are logged in.

```ts
// auth.config.ts
import type { NextAuthConfig } from 'next-auth';

export const authConfig = {
  pages: {
    signIn: '/login',
  },
  callbacks: {
    authorized({ auth, request: { nextUrl } }) {
      const isLoggedIn = !!auth?.user;
      const isOnDashboard = nextUrl.pathname.startsWith('/dashboard');

      if (isOnDashboard) {
        if (isLoggedIn) return true;
        return false; // Redirect unauthenticated users to login page
      } else if (isLoggedIn) {
        return Response.redirect(new URL('/dashboard', nextUrl));
      }

      return true;
    },
  },
  providers: [],
} satisfies NextAuthConfig;
```

The authorized callback is used to verify if the request is authorized to access a page via [Next.js Middleware](https://nextjs.org/docs/app/building-your-application/routing/middleware). It is called before a request is completed, and it receives an object with the `auth` and `request` properties. The `auth` property contains the user's session, and the `request` property contains the incoming request.

The `providers` option is an array where you list different login options.

Next, you will need to import the `authConfig` object into a middleware file. In the root of your project, create a file called `middleware.ts` and enter the following code:

```ts
// middleware.ts
import NextAuth from 'next-auth';
import { authConfig } from './auth.config';

export default NextAuth(authConfig).auth;

export const config = {
  // https://nextjs.org/docs/app/building-your-application/routing/middleware#matcher
  matcher: ['/((?!api|_next/static|_next/image|.*\\.png$).*)'], // Match all request paths except for API routes and Next.js static and image routes
};
```

Here you're initializing NextAuth.js with the `authConfig` object and exporting the `auth` property. You're also using the `matcher` option from Middleware to specify that it should run on specific path.

The advantage of employing Middleware for this task is that the protected routes will not even start rendering until the Middleware verifies the authentication, enhancing both the security and performance of your application.

### Password hashing

In your `seed.js` file, you used a package called `bcrypt` to hash the user's password before storing it in the database. You will use it _again_ later in this chapter to compare that the password entered by the user matches the one in the database. However, you will need to create a separate file for the `bcrypt` package. This is because `bcrypt` relies on Node.js APIs not available in Next.js Middleware.

Create a new file called `auth.ts` that spreads your `authConfig` object:

```ts
// auth.ts
import NextAuth from 'next-auth';
import { authConfig } from './auth.config';

export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
});
```

### Adding the Credentials provider

Next, you will need to add the `providers` option for NextAuth.js. `providers` is an array where you list different login options such as Google or Github. For this course, we will focus on using the [Credentials provider](https://authjs.dev/getting-started/providers/credentials-tutorial) only.

The Credentials provider allows users to login with with a username and a password.

```ts
// auth.ts
import NextAuth from 'next-auth';
import { authConfig } from './auth.config';
import Credentials from 'next-auth/providers/credentials';

export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
  providers: [Credentials({})],
});
```

**Good to know**:

Although we're using the Credentials provider, it's generally recommended to use alternative providers such as [OAuth](https://authjs.dev/getting-started/providers/oauth-tutorial) or [email](https://authjs.dev/getting-started/providers/email-tutorial) providers. See the [NextAuth.js doc](https://authjs.dev/getting-started/providers) for a full list of options.

### Adding the sign in functionalify

You can use the `authorize` function to handle the authentication logic. Similarly to Server Actions, you can use `zod` to validate the email and password before checking if the user exists in the database:

```ts
// auth.ts
import NextAuth from 'next-auth';
import { authConfig } from './auth.config';
import Credentials from 'next-auth/providers/credentials';
import { z } from 'zod';

export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
  providers: [
    Credentials({
      async authorize(credentials) {
        const parsedCredentials = z
          .object({
            email: z.string().email(),
            password: z.string().min(6),
          })
          .safeParse(credentials);
      },
    }),
  ],
});
```

After validating the credentials, create a new `getUser` function that queries the user from the database. Then, call `bcrypt.compare` to check if the passwords match:

```ts
// auth.ts
import NextAuth from 'next-auth';
import { authConfig } from './auth.config';
import Credentials from 'next-auth/providers/credentials';
import { z } from 'zod';
import { sql } from '@vercel/postgres';
import type { User } from '@/app/lib/definitions';
import bcrypt from 'bcrypt';

async function getUser(email: string): Promise<User | undefined> {
  try {
    const user = await sql<User>`SELECT * FROM users WHERE email = ${email}`;
    return user.rows[0];
  } catch (error) {
    console.error('Failed to fetch user: ', error);
    throw new Error('Failed to fetch user.');
  }
}

export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
  providers: [
    Credentials({
      async authorize(credentials) {
        const parsedCredentials = z
          .object({
            email: z.string().email(),
            password: z.string().min(6),
          })
          .safeParse(credentials);

        if (parsedCredentials.success) {
          const { email, password } = parsedCredentials.data;
          const user = await getUser(email);

          if (!user) return null;

          const passwordsMatch = await bcrypt.compare(password, user.password);

          if (passwordsMatch) return user;
        }

        console.log('Invalid credentials');
        return null;
      },
    }),
  ],
});
```

### Updating the login form

Now you need to connect the auth logic with your login form. In your `action.ts` file, create a new action called `authenticate`. This action should import the `signIn` function from `auth.ts`:

```ts
// app/lib/actions.ts
// ...
import { signIn } from '@/auth';
import { AuthError } from 'next-auth';

// ...

export async function authenticate(
  prevState: string | undefined,
  formData: FormData
) {
  try {
    await signIn('credentials', formData);
  } catch (error) {
    if (error instanceof AuthError) {
      switch (error.type) {
        case 'CredentialsSignin':
          return 'Invalid credentials.';
        default:
          return 'Something went wrong.';
      }
    }

    throw error;
  }
}
```

If there's a `'CredentialsSignin'` error, you want to show an appropriate message. You can learn about NextAuth.js errors in [the documentation](https://errors.authjs.dev/).

Finally, in your `login-form.tsx` component, you can use React's `useActionState` to call the server action, handle form errors, and display the form's pending state:

```tsx
// app/ui/login-form.tsx
'use client';

import { lusitana } from '@/app/ui/fonts';
import {
  AtSymbolIcon,
  KeyIcon,
  ExclamationCircleIcon,
} from '@heroicons/react/24/outline';
import { ArrowRightIcon } from '@heroicons/react/20/solid';
import { Button } from './button';
import { useActionState } from 'react';
import { authenticate } from '@/app/lib/actions';

export default function LoginForm() {
  const [errorMessage, formAction, isPending] = useActionState(
    authenticate,
    undefined
  );

  return (
    <form action={formAction} className="space-y-3">
    {/* ... */}
        <Button className="mt-4 w-full" aria-disabled={isPending}>
          Log in <ArrowRightIcon className="ml-auto h-5 w-5 text-gray-50" />
        </Button>
        <div
          className="flex h-8 items-end space-x-1"
          aria-live="polite"
          aria-atomic="true"
        >
          {errorMessage && (
            <>
              <ExclamationCircleIcon className="h-5 w-5 text-red-500" />
              <p className="text-sm text-red-500">{errorMessage}</p>
            </>
          )}
        </div>
      </div>
    </form>
  );
}
```

### Adding the logout functionality

To add the logout functionality to `<SideNav />`, call the `signOut` function from `auth.ts` in your `<form>` element:

```tsx
// app/ui/dashboard/side-nav.tsx
import Link from 'next/link';
import NavLinks from '@/app/ui/dashboard/nav-links';
import AcmeLogo from '@/app/ui/acme-logo';
import { PowerIcon } from '@heroicons/react/24/outline';
import { signOut } from '@/auth';

export default function SideNav() {
  return (
    <div className="flex h-full flex-col px-3 py-4 md:px-2">
    {/* ... */}
        <form
          action={async () => {
            'use server';
            await signOut();
          }}
        >
        {/* ... */}
```

## Adding Metadata

### Why is metadata important?

Metadata plays a significant role in enhancing a webpage's SEO, making it more accessible and understandable for search engines and social media platforms. Proper metadata helps search engines effectively index webpages, improving their ranking in search results. Additionally, metadata like Open Graph improves the appearance of shared links on social media, making the content more appealing and informative for users.

### Types of metadata

- **Title Metadata**: Responsible for the title of a webpage that is displayed on the browser tab. It's crucial for SEO as it helps search engines understand what the webpage is about.
  ```html
  <title>Page Title</title>
  ```
- **Description Metadata**: This metadata provides a brief overview of the webpage content and is often displayed in search engine results.
  ```html
  <meta name="description" content="A brief description of the page content." />
  ```
- **Keyword Metadata**: This metadata includes the keywords related to the webpage content, helping search engines index the page.
  ```html
  <meta name="keywords" content="keyword1, keyword2, keyword3" />
  ```
- **Open Graph Metadata**: This metadata enhances the way a webpage is represented when shared on social media platforms, providing information such as the title, description, and preview image.
  ```html
  <meta property="og:title" content="Title Here" />
  <meta property="og:description" content="Description Here" />
  <meta property="og:image" content="image_url_here" />
  ```
- **Favicon Metadata**: This metadata links the favicon to the webpage, displayed in the browser's address bar or tab.
  ```html
  <link rel="icon" href="path/to/favicon.ico" />
  ```

### Adding metadata

Next.js has a Metadata API that can be used to define your application metadata. There are two ways you can add metadata to your application:

- **Config-based**: Export a [static metadata object](https://nextjs.org/docs/app/api-reference/functions/generate-metadata#metadata-object) or a dynamic [generateMetadata function](https://nextjs.org/docs/app/api-reference/functions/generate-metadata#generatemetadata-function) in a `layout.js` or `page.js` file.
- **File-based**: Next.js has a range of special files that are specifically used for metadata purposes:
  - `favicon.ico`, `apple-icon.jpg`, and `icon.jpg`: Utilized for favicons and icons
  - `opengraph-image.jpg` and `twitter-image.jpg`: Employed for social media images
  - `robots.txt`: Provides instructions for search engine crawling
  - `sitemap.xml`: Offers information about the website's structure

You have the flexibility to use these files for static metadata, or you can generate them programmatically within your project.

With both these options, Next.js will automatically generate the relevent `<head>` element for your pages.

#### Favicon and Open Graph image

Put `favicon.ico` and `opengraph-image.png` to the root of your `/app` folder. By doing this, Next.js will automatically identify and use these files as your favicon and OG image. You can also create dynamic OG images using the [ImageResponse](https://nextjs.org/docs/app/api-reference/functions/image-response) constructor.

#### Page title and descriptions

You can also include a [metadata object](https://nextjs.org/docs/app/api-reference/functions/generate-metadata#metadata-fields) from any `layout.ts` or `page.ts` file to add additional page information like title and description. Any metadata in `layout.ts` will be inherited by all pages that use it.

In your root layout, create a new `metadata` object with the following fields:

```tsx
// app/layout.tsx
// ...
import { Metadata } from 'next';

export const metadata: Metadata = {
  title: {
    template: '%s | Acme Dashboard',
    default: 'Acme Dashboard',
  },
  description: 'The official Next.js Course Dashboard, built with App Router.',
  metadataBase: new URL('https://next-learn-dashboard.vercel.sh'),
};
// ...
```

The `%s` in the template will be replaced with specific nested page title.

Thus, if you want to add a custom title for a specific page, you can do this by adding a `metadata` object to the page itself. Metadata in nested pages will override the metadata in the parent, and `%s` will be replaced with the child page's title.

For example, in the `/dashboard/invoices` page, you can update the page title:

```tsx
// app/dashboard/invoices/page.tsx
// ...
import { Metadata } from 'next';

export const metadata: Metadata = {
    title: 'Invoices',
};
// ...
```

You can add multiple fields, including `keywords`, `robots`, `canonical`, and more, refer to the [documentation](https://nextjs.org/docs/app/api-reference/functions/generate-metadata).
