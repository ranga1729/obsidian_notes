1. Building the registration endpoint. 
Tried to send the registration data in the req body and the server shows 500 server error but still add the user data to the table. 

Known as "Write-then-Fail" error which results an inconsistent state(user created but API returns and error).

The user being created successfully in the database because the code block responsible for the database write executes completely. 

The 500 internal server error is being thrown after the database write but before the function can return a successful 201 response. 

The error must be between the database write code `prisma.user.create(...)` and the response return `return response;`

Possible error lines,
1. JWT creation
2. Cookie setting

- Error was at the JWT creation function. 
```ts
const JWT_EXPIRE_IN = process.env.JWT_EXPIRE_IN

export function generateToken(payload: Omit<JwtPayload, "iat"|"exp">): string {
  return jwt.sign(payload, JWT_SECRET, {
    expiresIn: JWT_EXPIRE_IN
  })
}
```
`JWT_EXPIRE_IN` wasn't read by the function from the `.env` file.
Fix : `const JWT_EXPIRE_IN = process.env.JWT_EXPIRE_IN || "7d"`
