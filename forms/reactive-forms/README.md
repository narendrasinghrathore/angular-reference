# Reactive Forms
Reactive forms use an explicit and immutable approach to managing the state of a form at a given point in time. Each change to the form state returns a new state, which maintains the integrity of the model between changes. Reactive forms are built around observable streams, where form inputs and values are provided as streams of input values, which can be accessed synchronously.

Reactive forms also provide a straightforward path to testing because you are assured that your data is consistent and predictable when requested. Any consumers of the streams have access to manipulate that data safely.

Reactive forms differ from template-driven forms in distinct ways. Reactive forms provide more predictability with synchronous access to the data model, immutability with observable operators, and change tracking through observable streams. If you prefer direct access to modify data in your template, template-driven forms are less explicit because they rely on directives embedded in the template, along with mutable data to track changes asynchronously. See the  [Forms Overview](https://angular.io/guide/forms-overview)  for detailed comparisons between the two paradigms.



The `[FormControl](https://angular.io/api/forms/FormControl)` class is the basic building block when using reactive forms. To register a single form control, import the `[FormControl](https://angular.io/api/forms/FormControl)` class into your component and create a new instance of the form control to save as a class property.

    import { Component } from '@angular/core';
    import { FormControl } from '@angular/forms';
    
    @Component({
      selector: 'app-name-editor',
      templateUrl: './name-editor.component.html',
      styleUrls: ['./name-editor.component.css']
    })
    export class NameEditorComponent {
      name = new FormControl('');
    }

Use the constructor of `[FormControl](https://angular.io/api/forms/FormControl)` to set its initial value, which in this case is an empty string. By creating these controls in your component class, you get immediate access to listen for, update, and validate the state of the form input.


### Registering the control in the template

After you create the control in the component class, you must associate it with a form control element in the template. Update the template with the form control using the  `formControl`  binding provided by  `[FormControlDirective](https://angular.io/api/forms/FormControlDirective)`  included in  `[ReactiveFormsModule](https://angular.io/api/forms/ReactiveFormsModule)`.

    <label>
      Name:
      <input type="text" [formControl]="name">
    </label>
Using the template binding syntax, the form control is now registered to the `name` input element in the template. The form control and DOM element communicate with each other: the view reflects changes in the model, and the model reflects changes in the view.


### Replacing a form control value

Reactive forms have methods to change a control's value programmatically, which gives you the flexibility to update the value without user interaction. A form control instance provides a  `setValue()`  method that updates the value of the form control and validates the structure of the value provided against the control's structure. For example, when retrieving form data from a backend API or service, use the  `setValue()`  method to update the control to its new value, replacing the old value entirely.

The following example adds a method to the component class to update the value of the control to  _Nancy_  using the  `setValue()`  method.

    updateName() {
      this.name.setValue('Nancy');
    }

**Note:** In this example, you're using a single control. When using the `setValue()` method with a form group or form array instance, the value needs to match the structure of the group or array.

## Grouping form controls
Just as a form control instance gives you control over a single input field, a form group instance tracks the form state of a group of form control instances (for example, a form). Each control in a form group instance is tracked by name when creating the form group. The following example shows how to manage multiple form control instances in a single group.

To initialize the form group, provide the constructor with an object of named keys mapped to their control.

    import { Component } from '@angular/core';
    import { FormGroup, FormControl } from '@angular/forms';
    
    @Component({
      selector: 'app-profile-editor',
      templateUrl: './profile-editor.component.html',
      styleUrls: ['./profile-editor.component.css']
    })
    export class ProfileEditorComponent {
      profileForm = new FormGroup({
        firstName: new FormControl(''),
        lastName: new FormControl(''),
      });
    }

The individual form controls are now collected within a group. A `[FormGroup](https://angular.io/api/forms/FormGroup)` instance provides its model value as an object reduced from the values of each control in the group. A form group instance has the same properties (such as `value` and `untouched`) and methods (such as `setValue()`) as a form control instance.

### Step 2: Associating the FormGroup model and view

A form group tracks the status and changes for each of its controls, so if one of the controls changes, the parent control also emits a new status or value change. The model for the group is maintained from its members. After you define the model, you must update the template to reflect the model in the view.

    <form [formGroup]="profileForm">
      
      <label>
        First Name:
        <input type="text" formControlName="firstName">
      </label>
    
      <label>
        Last Name:
        <input type="text" formControlName="lastName">
      </label>
    
    </form>

Note that just as a form group contains a group of controls, the _profile form_  `[FormGroup](https://angular.io/api/forms/FormGroup)` is bound to the `form` element with the `[FormGroup](https://angular.io/api/forms/FormGroup)` directive, creating a communication layer between the model and the form containing the inputs. The `[formControlName](https://angular.io/api/forms/FormControlName)` input provided by the `[FormControlName](https://angular.io/api/forms/FormControlName)` directive binds each individual input to the form control defined in `[FormGroup](https://angular.io/api/forms/FormGroup)`. The form controls communicate with their respective elements. They also communicate changes to the form group instance, which provides the source of truth for the model value.


### Saving form data

The  `ProfileEditor`  component accepts input from the user, but in a real scenario you want to capture the form value and make available for further processing outside the component. The  `[FormGroup](https://angular.io/api/forms/FormGroup)`  directive listens for the  `submit`  event emitted by the  `form`  element and emits an  `ngSubmit`  event that you can bind to a callback function.

Add an  `ngSubmit`  event listener to the  `form`  tag with the  `onSubmit()`  callback method.

src/app/profile-editor/profile-editor.component.html (submit event)

content_copy`<form [formGroup]="profileForm" (ngSubmit)="onSubmit()">`

The  `onSubmit()`  method in the  `ProfileEditor`  component captures the current value of  `profileForm`. Use  `[EventEmitter](https://angular.io/api/core/EventEmitter)`  to keep the form encapsulated and to provide the form value outside the component. The following example uses  `console.warn`  to log a message to the browser console.

src/app/profile-editor/profile-editor.component.ts (submit method)

`onSubmit()  {  // TODO: Use [EventEmitter](https://angular.io/api/core/EventEmitter) with form value console.warn(this.profileForm.value);  }`

The  `submit`  event is emitted by the  `form`  tag using the native DOM event. You trigger the event by clicking a button with  `submit`  type. This allows the user to press the  **Enter**  key to submit the completed form.

Use a  `button`  element to add a button to the bottom of the form to trigger the form submission.

src/app/profile-editor/profile-editor.component.html (submit button)

`<button  type="submit" [disabled]="!profileForm.valid">Submit</button>`

**Note:**  The button in the snippet above also has a  `disabled`  binding attached to it to disable the button when  `profileForm`  is invalid. You aren't performing any validation yet, so the button is always enabled. Simple form validation is covered in the  [Simple form validation](https://angular.io/guide/reactive-forms#simple-form-validation)  section.
