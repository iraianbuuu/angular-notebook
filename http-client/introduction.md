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