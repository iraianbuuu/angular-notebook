# HTTP Client

## Table of Contents

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
