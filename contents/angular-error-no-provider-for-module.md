---
title: Angular Error: &quot;No Provider for Module...&quot;
slug: angular-error-no-provider-for-module

publish_timestamp: June 18, 2017
url: https://www.codingforentrepreneurs.com/blog/angular-error-no-provider-for-module/

---


The error "No Provider for Module" is a common mistake when first starting out, especially using your own custom services. There are two ways, pretty easy ones, to solve that. You just have to remember to do it.

First off, if it's a service you intend to use throughout your components (let's say more than 2 components) you should put it in your `app.module.ts` in the `providers` argument. Below is an example:

#### `app.module.ts`
```
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { HttpModule } from '@angular/http';
import { YourService } from './your_service_dir/your_service.service';

@NgModule({
  declarations: [
    AppComponent,
    ...
  ],
  imports: [
    BrowserModule,
    FormsModule,
    HttpModule,
  ],
  providers: [YourService], // the key here
  bootstrap: [AppComponent]
})
export class AppModule {
  constructor(router: Router) {
  } 
}
```

Another way to use a service is inside a component's constructor itself. Below is an example of that:

#### `./item-list/item-list.component.ts`
```
import { Component, OnInit, OnDestroy } from '@angular/core';
import { Router } from "@angular/router";

import { YourService } from './your_service_dir/your_service.service';

@Component({
  selector: 'app-item-list',
  templateUrl: './item-list.component.html',
  styleUrls: ['./item-list.component.css'],
  providers: [YourService] // the key here
})
export class StaffListComponent implements OnInit, OnDestroy {
  constructor(
      private _yourService: YourService,
      ) { }
   ngOnIt(){}
   ngOnDestroy(){}
}
```


Basically, if you're going to use any angular features that are not built-in by default, you have to provide the feature into either your `app.module.ts` or inside the component's providers itself. Otherwise Angular will not know how to treat the module.
