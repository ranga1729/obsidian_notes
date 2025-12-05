- Next uses file based routing.
- Folders define the routes. Each folder inside the `app` directory represents a route segment that maps to a URL path segment. 
- `page.tsx` renders when the user visits the route segment where this file resides. 
- Routes can be nested using folders. 
Ex:

| Folder Structure           | URL Path       | Description                              |
| -------------------------- | -------------- | ---------------------------------------- |
| `app/page.tsx`             | `/`            | The application's home page              |
| `app/abot/page.tsx`        | `/about`       | A static route.                          |
| `app/blog/[slug]/page.tsx` | `blog/tshirts` | A dynamic segment. `slug` is a paramter. |
#### Dynamic Segments
Used when you need a single dynamic parameter in the URL. 
Ex: `app/posts/[id]/page.tsx`
The value is accessed in the component via the params prop. 
Ex: `params.id`
#### Router groups
Purpose: Organize file structure into logical groupings without affecting the resulting URL path.  
Ex:

| File/Folder structure            | URL      | Use case                                              |
| -------------------------------- | -------- | ----------------------------------------------------- |
| `app/(marketing)/about/page.tsx` | `/about` | Grouping marketing pages together in the file system. |
| `app/(shop)/cart/page.tsx`       | `/cart`  | Grouping e-com pages                                  |

The most often use cases to use grouping is to maintain different layouts. 
router won't show the folder name within parentheses in the URL.

-----
#### Prisma workflow(code first approach)
1. Do the changes to the `prisma.schema` file.
2. Generate client by running `npx prisma generate`. 
3. Create and apply migration by `npx prisma migrate dev --name your_migration_name`
4. To run the seeding, `npx prisma db seed`

-----
#### Next API Routes(app router)
Put all API routes in the `app/api/` folder. 
Ex: `app/api/roles/route.ts`
```tsx
import { NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";

export async function GET() {
  try {
    const roles = await prisma.role.findMany();
    return NextResponse.json(roles);
  } catch (error) {
    console.error("Failed to fetch roles:", error);
    return NextResponse.json(
      { error: 'Failed to fetch roles' },
      { status: 500 }
    );
  }
}
```

- Use app router for all new project. 
- App router has built-in support for fetch caching and data re-validation. 
- App router uses named export(`export async function GET() { }`)
- The file must be name `route.ts`.
- Folder structure defines the API endpoint,

| Folder structure              | Endpoint                        |
| ----------------------------- | ------------------------------- |
| `app/api/users/route.ts`      | `/api/users`                    |
| `app/api/users/[id]/route.ts` | `api/users/123` (Dynamic route) |
| `app/dashboard/api/route.ts`  | `/dashboard/api`                |

Common HTTP methods for API functions,
- GET': Retrieve data
- POST : Create new data
- PUT/PATCH : update existing data
- DELETE : Remove data

Reading URL Params,
```ts
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const search = searchParams.get('search');
}
```

Reading dynamic path params
```ts
// app/api/users/[id]/route.ts
interface Params {
  id: string;
}
// 'context' is the second argument, which contains the params
export async function GET(request: Request, context: { params: Params }) {
  const id = context.params.id;
}
```

Need multiple GET methods in the same file ?
It's not allowed. Instead strategize your endpoint management.
Ex: 
1. Use query parameters to differentiate logic
```ts
const { searchParams } = new URL(request.url);
const roleName = searchParams.get('name')

if (roleName) { 
	//logic for fetch a specific role
	roles = await prisma.role.findFirst({ 
		where: { name: roleName }
	}); 
	if (!roles) { 
		return NextResponse.json(
			{ message: `Role not found` }, 
			{ status: 404 }
		); 
	} 
} else {
	//logic for fetch all
	roles = await prisma.role.findMany(); 
}

```

2. Use different routes(different file paths)

| endpoint                            | goal                    |
| ----------------------------------- | ----------------------- |
| `app/api/roles/route.ts`            | get all roles           |
| `app/api/roles/[id]/route.ts`       | get specific role by id |
| `app/api/role-permissions/route.ts` | get role permissions    |


----
#### Status Code

| Status code |                       | Usage                                       |
| ----------- | --------------------- | ------------------------------------------- |
| 200         | OK                    | Successful GET, PUT, PATCH                  |
| 201         | Created               | Successful POST                             |
| 204         | No Content            | Successful DELETE                           |
| 400         | Bad request           | Client provided invalid/missing data        |
| 401         | Unauthorized          | Client is not authenticated                 |
| 403         | Forbidden             | Client is authenticated but lack permission |
| 404         | Not found             | Resource does not exist.                    |
| 405         | Method not allowed    | A method handler is not exported.           |
| 500         | Internal server error | A server-side error.                        |

-----
#### JWT Payload
Current payload,
```ts
export interface JwtPayload {
  userId: string,
  email: string,
  iat?: number
  exp?: number
}
```
- `iat` : issued at(time)
- `exp` : expiration(time)

These value settings are handled by the library `jsonwebtoken`,
```ts
return jwt.sign(payload, JWT_SECRET, {
  expiresIn: JWT_EXPIRE_IN 
})
```
- No need to specifically mention them in the payload.
- Don't include sensitive data in the payload.

For an e-com application what else can be added to the payload to minimize api calls,
- Cart Id
- firstName, lastName (good for UX, no need of api calls/database queries)
- role 

But if a user update their fist/last name, you need to force a token refresh.

**Why include `role` in the JWT Payload if you are planning to have a API middleware to check user permission based on the Bearer token ?**
1. Conditional rendering 
Ex: if the `role=admin` the app can immediately render the button to navigate to the admin dashboard without making any API call. Then subsequent request must checked with the middleware for security.
2. API middleware optimization.
If the request header includes the role as the something doesn't have the permission to access the requesting source, block it without even checking the database for further details. 
3. Create a fine line between authentication and authorization
JWT payload including the role : authentication
API middleware to check permissions: authorization
