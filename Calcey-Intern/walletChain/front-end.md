- to build the react template for typescript using yarn
```bash
yarn create reat-app <app_name> --template typescrpt
```
- All components should be in a folder called `components` in the `src` folder. 
- when creating a component, first import React using,
```tsx
import React from 'react'
```
- To import `NavLink` from React router,
```tsx
import { NavLink } from 'react-router-dom'
```
- `NavLink` is similar to HTML `<a>` anchor tag. But anchor tag load the relevant web page completely. But `NavLink` will only load the relevant components to the page without refreshing the whole page when a user clicks on any of the links.
- When importing an entire library to your components, you don't have to use "{ }" curly braces. Simply mention the component name and the location. Curly braces can be used in **to import specific parts of the modules instead of importing the whole module**.
```tsx
import { useState, useEffect } from 'react';
```
