+++
title = 'Mastering React Query in Next.js'
date = 2024-05-20T13:04:27-03:00
draft = false
+++

React query is a tool for creating hooks for fetching, caching and updating asynchronous data [repository](https://github.com/TanStack/query).

### Motivation

Fetching data from your backend is one of the most performance killers features of any frontend, and it really helps to keep a copy of the fetched data in the browser, but it's not always possible, or convenient to build a Redux store, so react-query is a good alternative for managing API results without having to do a lot of work for it, and without having to fetch it again every time you need it.

Here are some benefits of using react-query in your react app:

  - Simplified Data Fetching: React Query offers a declarative approach, allowing you to define data dependencies using hooks. This eliminates the boilerplate code typically associated with manual data fetching.
  - Automatic Caching: React Query manages in-memory data caching, reducing unnecessary API calls and improving responsiveness.
  - Optimized Refetching: You can configure refetching behavior based on various scenarios, including window focus or network reconnection, ensuring data freshness.
  - Shared State: React Query facilitates the creation of shared state across components, eliminating the need for complex state management solutions for simple data needs.
  - Improved Developer Experience: React Query's hooks provide a clean and concise syntax, enhancing code readability and maintainability.

### Usage

First, you have to wrap your application component with the `QueryClientProvider`, passing an instance of `QueryClient`. This makes the query client accessible throughout your application.:

``` typescript
// _app.tsx

import { QueryClient, QueryClientProvider } from 'react-query';

export default function App({Component, pageProps}) {
  const queryClientRef = useRef<QueryClient>();

  if (!queryClientRef.current) {
    queryClientRef.current = new QueryClient();
  }

  return (
    <React.Fragment>
      <Head>
        ...
      </Head>
      <QueryClientProvider client={queryClientRef.current}>
        <Component {...pageProps} />
      </QueryClientProvider>
    </React.Fragment>
  );
}
```

If you don't know what a provider is, visit [the React docs](https://reactjs.org/docs/context.html).

Secondly, you have to make a service, for example at `/lib/api/getClients.ts`

``` typescript
// /lib/api/getClients.ts

export async function getClients(): Promise<NetworkResponse<Client>> {
  return axios.get(`/api/clients`).then(response => response.data);
}
```

And finally, a hook for consuming the service and giving react query the retry options:


``` typescript
// /lib/hooks/useClients.ts

export function useClients() {
  const { data, status } = useQuery<NetworkResponse<Client>, AxiosError>(
    `/clients`,
    () => getClients(),
    {
      retry: true, // Optionally enable retries on failures
      refetchOnWindowFocus: false,
      refetchOnReconnect: false
    }
  );

  return { data: data, status };
}

```

Now you're good to go, using the hook in every component necessary, like so:

``` typescript

export default function Clients() {
  const { clients } = useClients()

  return (
    <>
      <div className="flex flex-wrap mt-4">
        <div className="w-full mb-12 px-4 h-full">
          {clients && clients.length > 0 ? (
            <ClientsTable clients={clients} />
          ) : (
            <Spinner />
          )}
        </div>
      </div>
    </>
  );
}

```

Something else to mention if you want to use it, React Query also supports managing state updates through mutations and it's a very helpful and easy to use way to handle states without adding too much boilerplate.

### Conclusion

Using react-query is pretty straight forward and simple, and the benefits on performance are considerable, so it's a very useful tool to have in every React project.