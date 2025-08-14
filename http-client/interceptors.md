# Interceptors

## Introduction

`HttpClient` supports a form of middleware known as **interceptors**. Interceptors are middleware that allows common patterns around retrying,caching,logging and authentication.

`HttpClient` supports two kinds of interceptors:

- Functional
- DI-based.

It can be used in common patterns such as,

- Authentication headers.
- Retrying failed requests.
- Caching responses
- Regularly polling the server

## Defining an Interceptor

The function which receives the outgoing `HttpRequest` and `next` function representing the next step in the interceptor chain.

`custom-header.interceptor.ts`

### Functional-based

```ts
import { HttpHandlerFn, HttpRequest } from "@angular/common/http";

export const customHeaderInterceptor: HttpInterceptorFn = (req, next) => {
  const customHeader = req.clone({
    setHeaders: {
      "X-Own-Header": "Custom-Header-XoX",
    },
  });
  return next(customHeader);
};
```

```ts
@Injectable()
export class CustomHeaderInterceptor implements HttpInterceptor {
  intercept(
    req: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    const customHeader = req.clone({
      setHeaders: {
        "X-Own-Header": "Custom-Header-XoX",
      },
    });
    return next.handle(customHeader);
  }
}
```

## Configuring interceptors

We should the set of interceptors to use when configuring `HttpClient` through DI, by using with `withInterceptors`

`app.config.ts`

### Functional-based

```ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([loggingInterceptor, customHeaderInterceptor])
    ),
  ],
};
```

### Dependency Injection based

```ts
provideHttpClient(
      withInterceptorsFromDi()
    ),
    {
      provide: HTTP_INTERCEPTORS,
      useClass: CustomHeaderInterceptor,
      multi: true,
    },
```

The interceptors we configure are chained together in the order.

## Request and Response Metadata

If we want to include information in request but not to the backend, but specifically meant for interceptors.`HttpRequest` have `context` object which stores metadata as an instance of `HttpContext`. `HttpContext` is mutable.

`api.service.ts`

```typescript
 getData() {
    return this.http.get(this.URL, {
      context: new HttpContext().set(CACHING_ENABLED, true),
    });
  }
```

`constants.ts`

```typescript
export const CACHING_ENABLED = new HttpContextToken(() => true);
```

`caching.interceptor.ts`
l

```typescript
export const cachingInterceptor: HttpInterceptorFn = (req, next) => {
  if (req.context.get(CACHING_ENABLED)) {
    // Add caching logic
  }
  return next(req);
};
```

## Synthetic responses

Most interceptors will simply invoke the `next` handler but this is not strictly a requirement.

### Working with redirect information

When using `useFetch` provider, responses include a `redirected` property.

```typescript
export const authRedirecInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    tap((event) => {
      if (event.type === HttpEventType.Response) {
        if (event.url?.includes("/login")) {
          // Handle Auth Redirection
        }
      }
    })
  );
};
```
