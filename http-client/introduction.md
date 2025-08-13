# HTTP Client

## Table of Contents

- [Introduction](#introduction)
- [Setting up](#setting-up)
- [Configuration of HttpClient](#configuration-of-httpclient)
- [Example](#example)
- [withRequestsMadeViaParent](#withrequestsmadeviaparent)
- [Making HTTP Requests](#making-http-requests)
  - [Fetching JSON Data](#fetching-json-data)
  - [Fetching other types of data](#fetching-other-types-of-data)
- [Mutating server state](#mutating-server-state)
- [Setting URL Parameters](#setting-url-parameters)
- [Setting request headers](#setting-request-headers)
- [Interacting with the server response events](#interacting-with-the-server-response-events)
- [Handling Request failures](#handling-request-failures)
  - [Timeouts](#timeouts)
- [Advanced fetch options](#advanced-fetch-options)
  - [Keep-alive connections](#keep-alive-connections)
  - [HTTP Caching control](#http-caching-control)
  - [Request priority for Core Web Vitals](#request-priority-for-core-web-vitals)
  - [Request Mode](#request-mode)
  - [Redirect handling](#redirect-handling)
  - [Credentials handling](#credentials-handling)
- [HTTP Observables](#http-observables)

## Introduction

Angular provides a client HTTP API for Angular applications, the `HTTPClient` service class in `@angular/common/http`.

## Setting up

`HttpClient` is provided using the `provideHttpClient` helper function.

`app.config.ts`

```ts
export const appConfig: ApplicationConfig = {
  providers: [provideHttpClient()],
};
```

## Configuration of HttpClient

`provideHttpClient` accepts a list of optional feature configurations. The `HttpClient` uses the `XMLHttpRequest` API to make requests.

1. `withFetch` - Uses `fetch` API instead of `XMLHttpRequest`.
2. `withInterceptors` - Configures a set of interceptor functions which will process requests made through `HttpClient`.
3. `withXsrfConfiguration` - Allows customization of XSRF security functionality.
4. `withNoXsrfProtection` - Disables built-in XSRF security functionality.
5. `withRequestsMadeViaParent() - This overrides any configuration for `HttpClient` which may present in the parent injector.

## Example

`app.component.ts`

```ts
  starwars: any;
  private apiService = inject(ApiService);

  fetchData() {
    this.apiService.getData().subscribe((data) => {
      this.starwars = data;
    });
  }
```

`app.component.html`

```html
<button type="button" (click)="fetchData()">Show Data</button>

<pre>{{ starwars | json }}</pre>
```

`api.service.ts`

```ts
import { HttpClient } from '@angular/common/http';
import { inject, Injectable } from '@angular/core';

@Injectable({...})
export class ApiService {
  private http = inject(HttpClient);
  private URL = 'https://swapi.info/api/people/1';
  getData() {
    return this.http.get(this.URL);
  }
}
```

## `withRequestsMadeViaParent`

By default, when we configure `HttpClient` using `provideHttpClient` within a given injector, this configuration overrides any configuration for `HttpClient` which may be present in the parent injector.

This is useful if you want to add interceptors in a child injector, while still sending the request through the parent injector's interceptors as well.

## Making HTTP Requests

`HttpClient` has methods similiar to HTTP verbs to make requests. Each method returns **RxJS `Observable`**.

### Fetching JSON Data

To fetch the data from a backend we can use `HttpClient.get()` method. By default, `HttpClient` assumes that servers will return JSON data.

`api.service.ts`

```ts
export class ApiService {
  private http = inject(HttpClient);
  private URL = "https://jsonplaceholder.typicode.com/todos/1";
  getData() {
    return this.http.get(this.URL);
  }
}
```

### Fetching other types of data

We can tell the `HttpClient` what response type to expect and return when making request. This is done with the `responseType` option.

| Response Type value | Returned response type |
| ------------------- | ---------------------- |
| `json` (default)    | JSON data              |
| `text`              | String data            |
| `arrayBuffer`       | Arraybuffer            |
| `blob`              | Blob                   |

`api.service.ts`

```ts
  getFile() {
    return this.http.get('/assets/files/demo.txt', {
      responseType: 'text',
    });
  }

  getImage() {
    return this.http.get('/assets/images/zoho-cliq.png', {
      responseType: 'arraybuffer',
    });
  }
```

## Mutating server state

The `HttpClient.post()` method behaves similarly to `get()` and accepts an additional `body`.

```ts
  createPost() {
    const newPost = {
      title: 'foo',
      body: 'bar',
      userId: '1',
    };
    return this.http.post(this.POSTS_URL, newPost , {
      headers : {
        'Content-type' : 'application/json'
      }
    }).subscribe((response) => console.log(response));
  }
```

| Type                                 | Serialized as                                       |
| ------------------------------------ | --------------------------------------------------- |
| string                               | plain text                                          |
| number,boolean,array or plain object | JSON                                                |
| `ArrayBuffer`                        | raw data from the buffer                            |
| `Blob`                               | raw data with the `Blob`'s content type             |
| `FormData`                           | `multipart/form-data` encoded data                  |
| `HttpParams or `URLSearchParams`     | `application/x-www-form-urlencoded formatted string |

## Setting URL Parameters

To specify request parameters in the request URL, we can use `params` option. Alternatively, we can pass an instance of `HttpParams` if we need control over the parameters.

> `HttpParams` are immutable and cannot be directly changed.

```ts
  getTodosByFilter(id: number) {
    const params = new HttpParams();
    const todoId = params.set('id', id);
    const userId = todoId.set('userId', id);
    return this.http.get(this.URL, {
      params: userId,
    });
  }
```

## Setting request headers

To specify the request headers in the request URL, we can use `headers` option. Alternatively, we can pass an instance of `HttpHeaders` if we need control over the headers.

> `HttpHeaders` are immutable and cannot be changed directly.

```ts
  getTodosByFilter(id: number) {
    const customHeader = headers.set("X-Custom-Header", "@#12");
    return this.http.get(this.URL, {
      headers: customHeader,
    });
  }
```

## Interacting with the server response events

`HttpClient` by default returns an `Observable` of the data returned by the server (response body). To access the entire response, set the `observe` option to `response`.

```ts
createPost() {
    const newPost = {
      title: 'foo',
      body: 'bar',
      userId: '1',
    };
    return this.http.post(this.POSTS_URL, newPost , {
      observe : 'response',
    }).subscribe((response) => {
      console.log(response.status)
    });
  }
```

## Receiving raw progress events

`HttpClient` also return a stream of raw events corresponding to specific moments in the request lifecycle. Progress events are disable by default. It can be enabled with `reportProgress` option. To observe the event stream, set the `observer` option to `events`.

| Type                             | Event                                                                              |
| -------------------------------- | ---------------------------------------------------------------------------------- |
| `HttpEventType.sent`             | The request has been dispatched to the server                                      |
| `HttpEventType.UploadProgress`   | An `HttpUploadProgressEvent` reporting progress on uploading the request body      |
| `HttpEventType.ResponseHeader`   | The head of the response has been received, including status and headers           |
| `HttpEventType.DownloadProgress` | An `HttpDownloadProgressEvent` reporting progress on downloading the response body |
| `HttpEventType.Response`         | The entire response has been received, including the response body                 |
| `HttpResponse.User`              | A custom event from an Http interceptor.                                           |

```typescript
    this.http.post('/api/v1/upload',formData , {
      observe : 'events',
      reportProgress : true,
    }).subscribe({
      next: (event) => {
        switch (event.type) {
          case HttpEventType.UploadProgress:
            if (event.total) {
              console.log((event.loaded / event.total) * 100);
            }
            break;
          case HttpEventType.Response:
            console.log("Upload completed successfully:", event.body);
            break;
          default:
            break;
        }
      },
      error: (error) => {
        console.error("Upload failed:", error);
      },
    });
  }
}
```

## Handling Request failures

There are three ways an HTTP request can fail:

- A network or connection error.
- A request didn't respond in time when the timeout option was set.
- The backend can receive the request but fail to process it, and return an error response.

`HttpClient` captures all of the above kinds of errors in an `HttpErrorResponse`. Network and timeout errors have a status code of `0`. Backend errors have the failing `status` code returned by the backend and error response the `error`.

We can use the `catchError` operator to transform an error response into a value. `retry()` operator will automatically attempt to re-subscribe a specified number of times.

```typescript
this.http
  .get("/api/v1/todos")
  .pipe(
    retry({
      count: 3,
      delay: 1000,
    }),
    catchError((error) => {
      return throwError(() => error);
    })
  )
  .subscribe({
    next: (data) => {
      console.log(data);
    },
    error: (error) => {
      console.error(error);
    },
  });
```

### Timeouts

To set a timeout for a request, we can use the `timeout` option.If the backend doesn't respond within the timeout period, the request will fail with a timeout error.

> The timeout will only apply to the backend HTTP request not for the entire request handling chain.

## Advanced fetch options

### Keep-alive connections

The `keepalive` allows a request to outlive the page or tab that made it.

```typescript
http
  .post("/api/analytics", data, {
    keepalive: true,
  })
  .subscribe();
```

### HTTP Caching control

The `cache` option controls how the request interacts with the browser's HTTP cache.

```typescript
//  Use cached response regardless of freshness
http
  .get("/api/config", {
    cache: "force-cache",
  })
  .subscribe((config) => {
    // ...
  });
// Always fetch from network, bypass cache
http
  .get("/api/live-data", {
    cache: "no-cache",
  })
  .subscribe((data) => {
    // ...
  });
// Use cached response only, fail if not in cache
http
  .get("/api/static-data", {
    cache: "only-if-cached",
  })
  .subscribe((data) => {
    // ...
  });
```

### Request priority for Core Web Vitals

The `priority` option controls the request's priority.

```typescript
// High priority for critical resources
http
  .get("/api/user-profile", {
    priority: "high",
  })
  .subscribe((profile) => {
    // ...
  });
// Low priority for non-critical resources
http
  .get("/api/recommendations", {
    priority: "low",
  })
  .subscribe((recommendations) => {
    // ...
  });
// Auto priority (default) lets the browser decide
http
  .get("/api/settings", {
    priority: "auto",
  })
  .subscribe((settings) => {
    // ...
  });
```

### Request Mode

The `mode` option controls the request handles cross-origin requests.

```typescript
// Same-origin requests only
http
  .get("/api/local-data", {
    mode: "same-origin",
  })
  .subscribe((data) => {
    // ...
  });
// CORS-enabled cross-origin requests
http
  .get("https://api.external.com/data", {
    mode: "cors",
  })
  .subscribe((data) => {
    // ...
  });
// No-CORS mode for simple cross-origin requests
http
  .get("https://external-api.com/public-data", {
    mode: "no-cors",
  })
  .subscribe((data) => {
    // ...
  });
```

### Redirect handling

The `redirect` option controls how the request handles redirects.

```typescript
// Follow redirects automatically (default behavior)
http
  .get("/api/resource", {
    redirect: "follow",
  })
  .subscribe((data) => {
    // ...
  });
// Prevent automatic redirects
http
  .get("/api/resource", {
    redirect: "manual",
  })
  .subscribe((response) => {
    // Handle redirect manually
  });
// Treat redirects as errors
http
  .get("/api/resource", {
    redirect: "error",
  })
  .subscribe({
    next: (data) => {
      // Success response
    },
    error: (err) => {
      // Redirect responses will trigger this error handler
    },
  });
```

### Credentials handling

The `credentials` option controls how the request handles credentials.

```typescript
// Include credentials for cross-origin requests
http
  .get("https://api.example.com/protected-data", {
    credentials: "include",
  })
  .subscribe((data) => {
    // ...
  });
// Never send credentials (default for cross-origin)
http
  .get("https://api.example.com/public-data", {
    credentials: "omit",
  })
  .subscribe((data) => {
    // ...
  });
// Send credentials only for same-origin requests
http
  .get("/api/user-data", {
    credentials: "same-origin",
  })
  .subscribe((data) => {
    // ...
  });
// withCredentials overrides credentials setting
http
  .get("https://api.example.com/data", {
    credentials: "omit", // This will be ignored
    withCredentials: true, // This forces credentials: 'include'
  })
  .subscribe((data) => {
    // Request will include credentials despite credentials: 'omit'
  });
```
## HTTP Observables

Each request method on `HttpClient` returns an `Observable` of the response body. `HttpClient` produces **cold** observables. The actual request is not made until the `subscribe()` method is called. Subscribing to same `Observable` multiple times will result in multiple requests being made. Each subscription is independent.