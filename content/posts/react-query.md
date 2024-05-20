+++
title = 'React Query'
date = 2024-05-18T13:04:27-03:00
draft = false
+++
## How to use React Query on Nextjs

React query is a tool for creating hooks for fetching, caching and updating asynchronous data [repository](https://github.com/TanStack/query).

### Motivation

Fetching data from your backend is one of the most performance killers features of any frontend, and it really helps to keep a copy of the fetched data in the browser, but it's not always possible, or convenient to build a Redux store, so react-query is a good alternative for managing API results without having to do a lot of work for it, and without having to fetch it again every time you need it.
With react-query, you have a shared source for all components that use that data, with a cache and mutations for updating the state across your application. 

### Usage

First, you have to configure the provider `QueryClientProvider` like this:

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

Secondly, you have to make a service, for example at `/lib/services/getClientsService.ts`

``` typescript
// /lib/hooks/getClientsService.ts

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
      retry: true,
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

### Conclusion

Using react-query is pretty straight forward and simple, and the benefits on performance are considerable, so it's a very useful tool to have in every React project.