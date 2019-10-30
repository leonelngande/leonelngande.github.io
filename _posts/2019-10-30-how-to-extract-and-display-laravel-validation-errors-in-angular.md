---
layout: post
hidden: false
title: How To Extract and Display Laravel Validation Errors in Angular
tags:
  - Angular
  - Laravel
  - Laravel Validation Errors
---
While working on [Xamcademy](https://xamcademy.com), I've had the need to extract and display form validation errors returned by the Xamcademy API which is powered by [Laravel](https://laravel.com). I wrote a utility function a while back which I forgot about, and remembered while trying to write something similar today.

**1 - Extracting the Validation Errors**

For reference, here's a sample error response:

![Laravel API response for invalid form request](/images/uploads/user-edit-error-response.png)

Here's the utility function:

```typescript
 import {HttpErrorResponse} from '@angular/common/http';

export const extractErrorMessagesFromErrorResponse = (errorResponse: HttpErrorResponse) => {
  // 1 - Create empty array to store errors
  const errors = [];

  // 2 - check if the error object is present in the response
  if (errorResponse.error) {

    // 3 - Push the main error message to the array of errors
    errors.push(errorResponse.error.message);

    // 4 - Check for Laravel form validation error messages object
    if (errorResponse.error.errors) {

      // 5 - For each error property (which is a form field)
      for (const property in errorResponse.error.errors) {

        if (errorResponse.error.errors.hasOwnProperty(property)) {

          // 6 - Extract it's array of errors
          const propertyErrors: Array<string> = errorResponse.error.errors[property];

          // 7 - Push all errors in the array to the errors array
          propertyErrors.forEach(error => errors.push(error));
        }

      }

    }

  }

  return errors;
};
```

This function can then be called in the error callback of the HTTPClient request to your API.  Let's use a hypothetical UserFormComponent as an example.

```typescript
import {Component, OnInit} from '@angular/core';
import {FormBuilder, Validators} from '@angular/forms';
import {IUser} from '../../state/user.model';
import {UserService} from '../../state/users.service';
import {take} from 'rxjs/operators';
import {HttpErrorResponse} from '@angular/common/http';
import {
  extractErrorMessagesFromErrorResponse,
} from '../../../@core/helper-functions/extract-error-messages-from-error-response';

@Component({
  selector: 'xc-user-form',
  templateUrl: './user-form.component.html',
  styleUrls: ['./user-form.component.scss'],
})
export class UserFormComponent implements OnInit {

  form: FormGroup;

  constructor(private fb: FormBuilder, private userService: UserService) {}

  ngOnInit() {
    this.initForm();
  }

  private initForm() {

    this.form = this.fb.group({
      id: [],
      email: [{value: '', disabled: true}, Validators.required],
      name: ['', Validators.required],
      // ...other fields
    });
  }

  submit(formData: IUser) {
    this.userService.create(formData)
      .pipe(
        take(1),
      )
      .subscribe(
        (response) => {
          // do something with success response
        },
        (errorResponse: HttpErrorResponse) => {
          // Extract form error messages from API  <------ HERE!!!
          const messages = extractErrorMessagesFromErrorResponse(errorResponse);
        },
      );
  }
}
```

Our interest is in the `submit()` function, specifically the error callback of the HTTPClient response observable.

We call the UserService's `create` method which is in charge of making the request to the API through Angular's HTTPClient and returning the response.

We then proceed to call our `extractErrorMessagesFromErrorResponse` function defined above and store the returned errors in the `messages` variable. You can place that function where it makes sense for you in your Angular project, I often use a `helper-functions` folder.

**2 - Displaying the Validation Errors** 

You can choose to display the errors however you like, given it's an array of strings you can loop over in the component template. Having written such functionality a good number of times across projects already, I decided to write something I could re-use, the `FormStatus` class.

```typescript
interface IShowMessages {
  error?: boolean;
  success?: boolean;
}

export class FormStatus {
  submitted: boolean;
  showMessages: IShowMessages;
  errors: string[];
  messages: string[];

  constructor(params?: {submitted?: boolean, showMessages?: IShowMessages, errors?: string[], messages?: string[]}) {
    this.submitted = (params && params.submitted) || false;
    this.showMessages = (params && params.showMessages) || {};
    this.errors = (params && params.errors) || [];
    this.messages = (params && params.messages) || [];
  }

  /**
   * Determines if error messages can be shown
   */
  canShowErrors() {
    return this.showMessages.error && this.errors && this.errors.length > 0 && !this.submitted;
  }

  /**
   * Determines if success messages can be shown
   */
  canShowSuccess() {
    return this.showMessages.success && this.messages && this.messages.length > 0 && !this.submitted;
  }

  /**
   * Called when the form it's attached to is being submitted
   * Resets the `errors` and `messages` arrays and sets `submitted` to true
   */
  onFormSubmitting() {
    this.errors = this.messages = [];
    this.submitted = true;
  }

  /**
   * Called when the form it's attached to has received a response for it's submit action
   * Sets `submitted` to false, sets and displays error or success messages
   *
   * @param params
   */
  onFormSubmitResponse(params: {success: boolean, messages: string[]}) {
    this.submitted = false;
    if (params.success) {
      this.showMessages.success = true;
      this.messages = params.messages;
    } else {
      this.showMessages.error = true;
      this.errors = params.messages;
    }
  }
}
```

We're going to use this class in the UserFormComponent's ts file and template to automatically handle repetitve aspects of the form submission process like setting the form as submitted, clearing the messages arrays on submit, and determining whether to show the messages as success or error messages. If you've had to deal with these over and over in your forms, then you'll understand where am coming from 🙂.

Let's update the UserFormComponent to make use of our new class.

```typescript
 import {Component, OnInit} from '@angular/core';
import {FormBuilder, Validators} from '@angular/forms';
import {IUser} from '../../state/user.model';
import {UserService} from '../../state/users.service';
import {take} from 'rxjs/operators';
import {HttpErrorResponse} from '@angular/common/http';
import {
  extractErrorMessagesFromErrorResponse,
} from '../../../@core/helper-functions/extract-error-messages-from-error-response';
import {FormStatus} from '../../../@core/form-helpers/form-status';

@Component({
  selector: 'xc-user-form',
  templateUrl: './user-form.component.html',
  styleUrls: ['./user-form.component.scss'],
})
export class UserFormComponent implements OnInit {

  form: FormGroup;

  // 1 - Initialize a form status object for the component
  formStatus = new FormStatus();

  constructor(private fb: FormBuilder, private userService: UserService) {}

  ngOnInit() {
    this.initForm();
  }

  private initForm() {

    this.form = this.fb.group({
      id: [],
      email: [{value: '', disabled: true}, Validators.required],
      name: ['', Validators.required],
      // ...other fields
    });
  }

  submit(formData: IUser) {
    // 2 - Call onFormSubmitting to handle setting the form as submitted and
    //     clearing the error and success messages array
    this.formStatus.onFormSubmitting();

    this.userService.create(formData)
      .pipe(
        take(1),
      )
      .subscribe(
        (response) => {
          // do something with success response
        },
        (errorResponse: HttpErrorResponse) => {
          const messages = extractErrorMessagesFromErrorResponse(errorResponse);
          // 3 - call onFormSubmitResponse with the submission success status (false) and the array of messages
          this.formStatus.onFormSubmitResponse({success: false, messages: messages});
        },
      );
  }
}
```

The additions to the component's ts file are marked as comments 1, 2, and 3.

Let's now see what the template for this component can look like.

```html
<form novalidate [formGroup]="form" (submit)="submit(form.value)">

  <!-- OUR AREA OF INTEREST -->
  <ng-container *ngIf="formStatus">
    
    <div *ngIf="formStatus.canShowErrors()" class="alert alert-danger" role="alert">
      <div><strong>Oops!</strong></div>
      <div *ngFor="let error of formStatus.errors">{{ error }}</div>
    </div>
    
    <div *ngIf="formStatus.canShowSuccess()" class="alert alert-success" role="alert">
      <div><strong>Success!</strong></div>
      <div *ngFor="let message of formStatus.messages">{{ message }}</div>
    </div>
    
  </ng-container>


  <div class="row">
    
    <div class="col-md-12">
      <div class="form-group">
        <!-- Input field for email goes here -->
      </div>

      <div class="form-group">
        <!-- Input field for name goes here -->
      </div>
    </div>

    <div class="col-md-12 text-right">
      <button type="submit"
              [class.btn-pulse]="formStatus.submitted"
              [disabled]="!form?.valid || formStatus.submitted">
        Save
      </button>
    </div>
    
  </div>


</form>
```

FYI, using Bootstrap 4 as CSS framework. Here's a preview for the errors in the error response image at the start of the post.

![Error messages preview](/images/uploads/error-messages-preview.png)

With that we've come to the end of this blog post, and hope this is useful to you in some way 🙌.
