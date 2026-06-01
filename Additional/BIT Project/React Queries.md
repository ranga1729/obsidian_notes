React query(TanStack Query) is a data-fetching and state management library for React apps which simplifies the fetching, caching, synchronizing and updating server state. 
It's specifically designed to manage asynchronous operations involving remote data which is fundamentally different from managing local UI state. 
#### Traditional Method of handling asynchronous data fetching with useState and useEffect
As a beginner, my traditional approach to execute data fetching is, 
- Implement the API endpoint/server action(in NextJS)
- Create a method in the frontend(client page.tsx) which asynchronously calls the endpoint/server action.
- Hold the state using useState. 
- UseEffect to re-fetch data if a dependency changed.
Ex: 
```tsx
  const [colors, setColors] = useState<Color[]>([]);
  const [isLoading, setIsLoading] = useState(false);
    const [paginator, setPaginator] = useState<Paginator>({
    pageSize: 10,
    pageIndex: 0,
    totalRecords: 0,
  })
  
const fetchData = async () => {
    try {
      setIsLoading(true);
      const response = await getColors(paginator, filter);

      if(!response.success && response.error) {
        toast.error(response.error);
      }
      if(response.success) {
        setColors(response.data.colors)
        setTotalRecords(response.data.totalRecords)
      }
    } catch(error:any) {
      toast.error(error.message)
    } finally {
      setIsLoading(false)
    }
  }
  
  useEffect(() => {
    fetchData();
  }, [paginator.pageIndex, paginator.pageSize]);
```
In this approach, we need to manually,
- Start loading.
- Call the API
- Handle errors
- Set state
- Stop loading.

Problems with this approach,
- No caching : every page visit re-fetches even if the data hasn't changed. 
- No deduplication: If `fetchData` is called twice simultaneously, two requests get executed.
- No automatic retry : if the request got failed, it dies.
- Race condition : If the paginator changes faster, older response can overwrite newer ones. 
- Boilerplate repetition : every page has the same structure of data fetching function and useState/useEffect combo. 

Core mindset shift here is the understanding about the  "Server State" vs. "Client State"
- Client state: Modals being open, form inputs, dark/light mode.
- Server State: Data you borrow from the database. 

React Query handles the Server State so you don't need to manually manage them. 

Ex: 
The above example code including it's useState and useEffect can be replaced by this query.
```tsx
const { data, isLoading, isError } = useQuery({
	queryKey: ['colors', paginator.pageIndex, paginator.pageSize, filter],
	queryFn: () => getColors(paginator, filter), 
	staleTime: 30_000, 
	placeholderData: (prev) => prev,
})
```

#### Anatomy of a react query,
Writing a good react query is about creating a predictable, self-healing, clean data layer.
Take this example, 
```tsx
import { useQuery } from '@tanstack/react-query';
import { getColors } from './action';
// To fetch color data
const { data, isLoading, isError } = useQuery({ 
	queryKey: ['colors', paginator.pageIndex, paginator.pageSize, filter],
	queryFn: () => getColors(paginator, filter), 
	staleTime: 30_000,
	placeholderData: (prev) => prev,
}); 
```
`getColors` function is defined in the backend and it is used to fetch color data. 
Key takeaways,
1. `queryKey: ['colors', paginator.pageIndex, paginator.pageSize, filter]`
	This is similar to writing a useEffect in react. Whenever any value in this array change. this query automatically triggers a re-fetch.
	The first string 'color' is used as a namespace/label.
2. `queryFn: () => getColors(paginator, filter)`
	This is the actual promise based function that talks to your API. React query waits for this function to resolve(success) of reject(error).
3. `staleTime: 30_000`
	Controls how long data is considered fresh before React query marks it stale and eligible for a background re-fetch.
	If the user leaves the tab and comes back after 10 seconds, react query doesn't initiate another `queryFn`, instead it uses the cached data, reducing unnecessary server hits. 
4. `placeholderData: (prev) => prev`
	Until your updated data retrieves from the backend, this will show previous data, creating a seamless transition for data updates. 


```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { deleteColor } from './action';
import toast from 'react-hot-toast';

export const colorKeys = {
  all: ['colors'] as const,
  list: (paginator: Paginator, filter: ColorFilter) => ['colors', 'list', { paginator, filter }] as const,
}

const queryClient = useQueryClient();
// To delete a color by it's id. 
const deleteMutation = useMutation({ 
	mutationFn: (id: string) => deleteColor(id), 
	onSuccess: (response) => {
      if(response.success) {
        queryClient.invalidateQueries({ queryKey: colorKeys.all})
        toast.success(response.message || en.color_deleted_successfully)
      } else {
        toast.error(response.error || en.failed_to_delete_color)
      }
    },
    onError: (error: Error) => {
      toast.error(error.message || en.failed_to_delete_color)
    },
});
```
`useQuery` is used for reading data. `useMutation` is for writing data(Create, Update, Delete).
Key takeaways,
1. `mutationFn: (id) => deleteColor(id)`
	Similar to the `queryFn`, talks to the backend. 
2. `onSuccess` & `onError`
	These are used to handle lifecycle callbacks. 
	In the previous approach, this is similar to having `.then()`, `.catch()` blocks. 
	`onSuccess` : runs immediately after the query/mutation operation successful completes. (resolved promise)
	`onError` : runs when a query/mutation throws and error. (rejected promise)
3. `queryClientInvalidateQueries`
	This is used to mark cached queries(referred by the `queryKey` label) as stale(expired). 
	So it will fetch the data again and update the cache. This ensures the front-end is in sync with the server after a data change(mutation). 