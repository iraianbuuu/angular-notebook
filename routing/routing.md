# Routing

## Table of Contents

- [Router Outlet](#router-outlet)
  - [Nesting Routes with Child Routes](#nesting-routes-with-child-routes)
  - [Secondary Routes](#secondary-routes)
- [Lifecycle](#lifecycle)
- [Navigation](#navigation)

## Router Outlet

The `RouterOutlet` directive is a placeholder that marks the location where the router should render the component for the current URL.

```html
<h1>Home Page</h1>
<router-outlet />
```

### Nesting routes with child routes

```html
<nav>
  <ul>
    <li><a routerLink="profile">Profile</a></li>
  </ul>
</nav>
<p>settings works!</p>

<router-outlet />
```

```ts
{
    path: 'settings',
    component: Settings,
    children: [
      {
        path: 'profile',
        component: Profile,
      },
    ],
},
```

### Secondary routes

Pages may have multiple outlets— you can assign a name to each outlet to specify which content belongs to which outlet. Each outlet must have a unique name. The name cannot be set or changed dynamically. By default, the name is `primary`.

```html
<app-header />
<router-outlet />
<router-outlet name="read-more" />
<router-outlet name="additional-actions" />
<app-footer />
```

```ts
{
  path: 'user/:id',
  component: UserDetails,
  outlet: 'additional-actions'
}
```

### Lifecycle

| Event        | Description                                                              |
| ------------ | ------------------------------------------------------------------------ |
| `activate`   | A new component is instantiated                                          |
| `deactivate` | When a component is destroyed                                            |
| `attach`     | When the `RouteReuseStrategy` instructs the outlet to attach the subtree |
| `detach`     | When the `RouteReuseStrategy` instructs the outlet to detach the subtree |

```ts
<router-outlet
  (activate)='onActivate($event)'
  (deactivate)='onDeactivate($event)'
  (attach)='onAttach($event)'
  (detach)='onDetach($event)'
/>
```

## Navigation

### Using `routerLink`

The `RouterLink` allows you to use standard anchor elements `<a>` with Angular's routing system.

### Using absolute or relative links

Relative URLs in Angular routing allow you to define navigation paths relative to the current route's location. This is in contrast to absolute URLs which contain the full path with the protocol (e.g., http://) and the root domain (e.g., google.com).

```html
<!-- Absolute URL -->
<a href="https://www.angular.dev/essentials">Angular Essentials Guide</a>
<!-- Relative URL -->
<a href="/essentials">Angular Essentials Guide</a>
```

```html
<nav>
  <ul>
    <li>
      <a href="/">Home Page</a>
    </li>
    <li>
      <a href="/settings/profile">Profile - 1 </a>
    </li>
    <li><a routerLink="profile">Profile - 2</a></li>
  </ul>
</nav>
<p>settings works!</p>
<router-outlet />
```

#### How relative URLs work

Angular routing has two syntaxes for defining relative URLs: strings and arrays.

```ts
<li>
  <a routerLink="/settings/profile">Profile</a>
  <a routerLink="['settings','profile']">Profile</a>
</li>
```
