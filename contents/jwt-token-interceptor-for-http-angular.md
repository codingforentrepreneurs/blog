---
title: Creating an JWT Token Interceptor for HTTP Requests in Angular
slug: jwt-token-interceptor-for-http-angular

publish_timestamp: Jan. 31, 2018
url: https://www.codingforentrepreneurs.com/blog/jwt-token-interceptor-for-http-angular/

---


Below is a snippet that's useful for using authentication tokens (such as a CSRF Token or a JWT token) within Angular (not AngularJS) for any/all requests. 

We show you how to implement it exact in the [Angular Integration](https://www.codingforentrepreneurs.com/courses/django-angular-ionic/angular-integration/) section of [Django + Angular + Ionic](https://www.codingforentrepreneurs.com/courses/django-angular-ionic/)


## Requirements:
- Angular 4+
- [ngx-cookie-serivce](https://www.npmjs.com/package/ngx-cookie-service) package installed (including in `app.module.ts`)

```
// app/auth/token.interceptor.ts

import { Injectable } from '@angular/core';
import {
  HttpRequest,
  HttpHandler,
  HttpEvent,
  HttpInterceptor,
  HttpResponse,
  HttpErrorResponse
} from '@angular/common/http';

import 'rxjs/add/operator/do';
import { Observable } from 'rxjs/Observable';
import { CookieService } from 'ngx-cookie-service'; //


@Injectable()
export class TokenInterceptor implements HttpInterceptor {
  constructor(
    private cookieService: CookieService
    ) {}
  intercept(request: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    let csrftoken = this.cookieService.get('csrftoken')
    let jwttoken = this.cookieService.get('jwttoken')
    request = request.clone({
      setHeaders: {
        // This is where you can use your various tokens
        // Authorization: `JWT ${jwttoken}`,
        // 'X-CSRFToken': `${csrftoken}`
      }
    });
    return next.handle(request).do((event: HttpEvent<any>) => {
      if (event instanceof HttpResponse) {
        // do stuff with response if you want
      }
    }, (err: any) => {
      if (err instanceof HttpErrorResponse) {
        if (err.status === 401 || err.status === 403) {
          console.log("handle error here")
          
        }
      }
    });
  }
}
```

#### Update `app.module.ts`
```
// app/auth/token.interceptor.ts

import { CookieService } from 'ngx-cookie-service';
import { HTTP_INTERCEPTORS } from '@angular/common/http';
import { TokenInterceptor } from './auth/token.interceptor';

@NgModule({
  declarations: [
    ...
  ],
  imports: [
    ...

  ],
  providers: [ 
     ...
      CookieService,
      {
          provide: HTTP_INTERCEPTORS,
          useClass: TokenInterceptor,
          multi: true
      }
    ],
  bootstrap: [AppComponent]
})
export class AppModule { }

```
