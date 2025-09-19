# Routing

## Table of Contents

- [Introduction](#introduction)
- [Routing](#routing)

## Introduction

Routing helps you change what the user sees in a single-page app. Angular Routing `@angular/core` if the official library for managing navigation in Angular Applications.

1. **Routes** define which component displays when a user visits a specific URL.
2. **Outlets** are placeholders in your templates that dynamically load and render components based on the active route.
3. **Links** provide a way for users to navigate between different routes in your application without triggering a full page reload.

## Routes

Routes serve as the fundamental building blocks for navigation within an Angular app. It's an object that defines which component should render for a specific URL path.

```typescript
export const routes: Routes = [
  {
    path: "/admin",
    component: AdminPage,
  },
];
```

When a user visits the `/admin` path, the app will display the `AdminPage` component.

## Route URL Paths

### Static URL Paths

It refers to routes with predefined paths that don't change based on dynamic parameters.

Examples :

- `/admin`
- `/settings`

### URL Paths with Route Parameters

Parameterized URLs allow you to define dynamic paths that allow multiple URLs to the same component while dynamically displaying data based on parameters in the URL.

```typescript
export const routes: Routes = [
  { path: "user/:id", component: UserProfile },
  { path: "user/:id/:social-media", component: SocialMediaFeed },
];
```

In the above example, URLs such as `/user/angular` and `user/react` render the `UserProfile` component.

Valid route parameter names can only contain :

- Letters (a-z,A-Z)
- Numbers (0-9)
- Underscore (_)
- Hyphen (-)

### Wildcards

It will catch all the routes to a specific path. It is defined with the `**`.

import { Home } from './home/home.component';
import { UserProfile } from './user-profile/user-profile.component';
import { NotFound } from './not-found/not-found.component';

const routes: Routes = [
  { path: 'home', component: Home },
  { path: 'user/:id', component: UserProfile },
  { path: '**', component: NotFound }
];

In the above example, if the user visits any other path it will redirect to thw `NotFound` component.

## Angular URLs Matching

The order of routes is important because Angular uses a **first-match** wins strategy. This means that once Angular matches a URL with a route path, it stops checking any further routes.

```typescript
const routes: Routes = [
  { path: '', component: HomeComponent },              // Empty path
  { path: 'users/new', component: NewUserComponent },  // Static, most specific
  { path: 'users/:id', component: UserDetailComponent }, // Dynamic
  { path: 'users', component: UsersComponent },        // Static, less specific
  { path: '**', component: NotFoundComponent }         // Wildcard - always last
];
```

If a user visits /users/new, Angular router would go through the following steps:

1. Checks '' - doesn't match
2. Checks users/new - matches! Stops here
3. Never reaches users/:id even though it could match
4. Never reaches users
5. Never reaches **

## Redirects

We can define a route that redirects to another route instead of rendering a component.

```typescript
import { BlogComponent } from './home/blog.component';
const routes: Routes = [
  {
    path: 'articles',
    redirectTo: '/blog',
  },
  {
    path: 'blog',
    component: BlogComponent
  },
];
```

## Loading Route Component Strategies

Angular offers two primary strategies to control component loading behavior:

**Eagerly loaded**: Components that are loaded immediately
**Lazily loaded**: Components loaded only when needed

### Eagerly Loaded

When you define a route with the component property, the referenced component is eagerly loaded as part of the same JavaScript bundle as the route configuration.

### Lazily Loaded

The `loadComponent` property is used to lazily load the Javascript for a route only when the route is active.

```typescript
import { Routes } from "@angular/router";
export const routes: Routes = [
  {
    path: 'login',
    loadComponent: () => import('./components/auth/login-page').then(m => m.LoginPage)
  },
  {
    path: '',
    loadComponent: () => import('./components/home/home-page').then(m => m.HomePage)
  }
]
```

## Page Titles

```typescript
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: 'about',
    loadComponent: () =>
      import('./components/about-page/about-page').then((m) => m.AboutPage),
    title : 'About Page'
  },
];
```

The page title can be set dynamically using `ResolveFn`.

```typescript
import { ResolveFn, Routes } from '@angular/router';

const titleResolver: ResolveFn<string> = (route) =>
  route.paramMap.get('id') || '';

export const routes: Routes = [
  {
    path: 'about/:id',
    loadComponent: () =>
      import('./components/about-page/about-page').then((m) => m.AboutPage),
    title: titleResolver,
  },
];
```

## Dependency Injection

## Passing data with routes

### Static Data

```typescript
export const routes: Routes = [
  {
    path: 'admin',
    loadComponent: () =>
      import('./components/admin-page/admin-page').then((m) => m.AdminPage),
    data: {
      admin: 'admin-z-123',
    },
  },
];
```

```typescript
export class AdminPage {
  private route = inject(ActivatedRoute);

  constructor() {
    this.route.data.subscribe((data) => {
      console.log(data); // admin-z-123
    });
  }
}
```

## Nested Routes

Nested routes, also known as child routes, are a common technique for managing more complex navigation routes

```typescript
{
    path: 'product/:id',
    component: Product,
    title: 'Product Page',
    children: [
      {
        path: 'info',
        component: ProductInfo,
      },
    ],
  },
```

```html
<p>product works!</p>
<router-outlet />
```
