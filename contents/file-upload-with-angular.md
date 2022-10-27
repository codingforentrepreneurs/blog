---
title: File Upload with Angular
slug: file-upload-with-angular

publish_timestamp: Feb. 1, 2018
url: https://www.codingforentrepreneurs.com/blog/file-upload-with-angular/

---

File uploads in Angular is simple and we’ll show you how below.

Below assumes you have authentication handled. In our case, we used a [JWT Token HttpInterceptor](https://www.codingforentrepreneurs.com/blog/jwt-token-interceptor-for-http-angular/) to ensure our requests had the needed auth header baked in.

We also assume you have a [REST API](https://www.codingforentrepreneurs.com/courses/rest-api/) that you're working with. 


### 1. Create your HttpClient Service (in `file-client/file-client.service.ts`)
```
import { Injectable } from '@angular/core';
import { HttpClient, HttpHeaders, HttpEventType, HttpRequest, HttpErrorResponse, HttpEvent } from '@angular/common/http';


import { Observable} from 'rxjs/Observable';
import { ErrorObservable } from 'rxjs/observable/ErrorObservable';
import { of } from  'rxjs/observable/of';
import { catchError, map, tap } from  'rxjs/operators';

/* Naming NOTE
  The API's file field is `fileItem` thus, we name it the same below
  it's like saying <input type='file' name='fileItem' /> 
  on a standard file field
*/


@Injectable()
export class FileUploadClientService {
    apiBaseURL = 'http://127.0.0.1:8000/api/'
    constructor(private http: HttpClient){ }

    fileUpload(fileItem:File, extraData?:object):any{
      let apiCreateEndpoint = `${this.apiBaseURL}files/create/`
      const formData: FormData = new FormData();
     
      formData.append('fileItem', fileItem, fileItem.name);
      if (extraData) {
        for(let key in extraData){
            // iterate and set other form data
          formData.append(key, extraData[key])
        }
      }
      
      const req = new HttpRequest('POST', apiCreateEndpoint, formData, {
        reportProgress: true // for progress data
      });
      return this.http.request(req)
    }
   
  optionalFileUpload(fileItem?:File, extraData?:object):any{
      let apiCreateEndpoint = `${this.baseUrl}files/create/`
      const formData: FormData = new FormData(); //?
       let fileName;
      if (extraData) {
        for(let key in extraData){
            // iterate and set other form data
            if (key == 'fileName'){
              fileName = extraData[key]
            }
          formData.append(key, extraData[key])
        }
      }

      if (fileItem){
        if (!fileName){
           fileName = fileItem.name
        }
        formData.append('image', fileItem, fileName);
      }
      const req = new HttpRequest('POST', apiCreateEndpoint, formData, {
        reportProgress: true // for progress data
      });
      return this.http.request(req)
  }
    list(): Observable<any>{
      const listEndpoint = `${this.apiBaseURL}files/`
      return this.http.get(listEndpoint)
    }

}
```


### 2. Update `app.module.ts`

```
// app/auth/token.interceptor.ts

import { CookieService } from 'ngx-cookie-service';
import { HTTP_INTERCEPTORS } from '@angular/common/http';
import { TokenInterceptor } from './auth/token.interceptor';

import { FileUploadClientService } from './file-client/file-client.service.ts'
@NgModule({
  declarations: [
    ...
  ],
  imports: [
    ...

  ],
  providers: [ 
     ...
     FileUploadClientService,
     ...
    ],
  bootstrap: [AppComponent]
})
export class AppModule { }

```


### 3. Use in Component, like `status-create.component.ts`

```
import { Component, OnInit, OnDestroy, ViewChild } from '@angular/core';
import { HttpClient, HttpHeaders, HttpEventType, HttpRequest, HttpErrorResponse, HttpEvent } from '@angular/common/http';

import { FormControl, FormGroup, Validators, NgForm } from '@angular/forms';
import { FileUploadClientService } from './file-client/file-client.service.ts'


@Component({
  selector: 'app-status-create',
  templateUrl: './status-create.component.html',
  styleUrls: ['./status-create.component.css']
})
export class StatusCreateComponent implements OnInit, OnDestroy {
    statusCreateForm: FormGroup;
    fileDescription: FormControl;
    fileToUpload: File  = null;
    uploadProgress:number = 0;
    uploadComplete:boolean = false;
    uploadingProgressing:boolean = false;
    fileUploadSub: any;
    serverResponse: any;

    @ViewChild('myInput')
    myFileInput: any;


    constructor(
        private fileUploadService: FileUploadClientService
    ) {}

    ngOnInit() {
        /* initilize the form and/or extra form fields
            Do not initialize the file field
        */
      this.fileDescription  = new FormControl("", [
              Validators.required,
              Validators.minLength(4),
              Validators.maxLength(280)
         ])
      this.statusCreateForm = new FormGroup({
          'description': this.fileDescription,
      })
    }

    ngOnDestroy {
        if (this.fileUploadSub){
            this.fileUploadSub.unsubscribe()
        }
    }

    handleProgress(event){
    if (event.type === HttpEventType.DownloadProgress) {
        this.uploadingProgressing =true
        this.uploadProgress = Math.round(100 * event.loaded / event.total)
      }

      if (event.type === HttpEventType.UploadProgress) {
        this.uploadingProgressing =true
        this.uploadProgress = Math.round(100 * event.loaded / event.total)
      }

      if (event.type === HttpEventType.Response) {
        // console.log(event.body);
        this.uploadComplete = true
        this.serverResponse = event.body
      }
    }
    handleSubmit(event:any, statusNgForm:NgForm, statusFormGroup:FormGroup){
      event.preventDefault()
      if (statusNgForm.submitted){
          
          let submittedData = statusFormGroup.value

          this.fileUploadSub = this.fileUploadService.fileUpload(
                this.fileToUpload, 
                submittedData).subscribe(
                    event=>this.handleProgress(event), 
                    error=>{
                        console.log("Server error")
                    });

          statusNgForm.resetForm({})
      }
  }


    handleFileInput(files: FileList) {
        let fileItem = files.item(0);
        console.log("file input has changed. The file is", fileItem)
        this.fileToUpload = fileItem
    }

    resetFileInput() {
        console.log(this.myFileInput.nativeElement.files);
        this.myFileInput.nativeElement.value = "";
        console.log(this.myFileInput.nativeElement.files);
    }

}


```


### 4. Update Template `status-create.component.html`

```
<form [formGroup]='statusFormGroup' #statusNgForm='ngForm' (submit)='handleSubmit($event, statusNgForm, statusFormGroup)'>
    
    <div class="form-group">
        <label for="file">Choose File</label>
            <input #myInput type="file"
                   id="file"
                   (change)="handleFileInput($event.target.files)">
    </div>

    <textarea id='fileDescription' name='fileDescription' formControlName='fileDescription'></textarea>
        
    <div *ngIf='fileDescription.invalid && (fileDescription.dirty || fileDescription.touched)'>
        <div *ngIf='fileDescription.errors.required'>
            Content is required.
        </div>
        <div *ngIf='fileDescription.errors.maxlength'>
            Max length is 280
        </div>
        <div *ngIf='fileDescription.errors.minlength'>
            Min length is 4
        </div>
    </div>

    
    <button type='submit' class='btn btn-default' [disabled]='statusFormGroup.invalid'>Submit</button>
    <button type='button' class='btn btn-default' (click)='statusNgForm.resetForm({})'>Reset</button>
</form>


Progress: 
<div style='width:%;background-color:#007cae;'><span *ngIf='uploadProgress > 25'>%</span></div>
```