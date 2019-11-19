## What are structural directives
Structural directives are responsible for HTML layout. They shape or reshape the DOM's  _structure_, typically by adding, removing, or manipulating elements.

As with other directives, you apply a structural directive to a  _host element_. The directive then does whatever it's supposed to do with that host element and its descendants.

Structural directives are easy to recognize. An asterisk (*) precedes the directive attribute name.

    <div *ngIf="hero" class="name">{{hero.name}}</div>

Angular desugars this notation into a marked-up `<ng-template>` that surrounds the host element and its descendents. Each structural directive does something different with that template.

Three of the common, built-in structural directives—[NgIf](https://angular.io/guide/template-syntax#ngIf), [NgFor](https://angular.io/guide/template-syntax#ngFor), and [NgSwitch...](https://angular.io/guide/template-syntax#ngSwitch)—are described in the [_Template Syntax_](https://angular.io/guide/template-syntax) guide and seen in samples throughout the Angular documentation.

Throughout this guide, you'll see a directive spelled in both  _UpperCamelCase_  and  _lowerCamelCase_. Already you've seen  `[NgIf](https://angular.io/api/common/NgIf)`  and  `[ngIf](https://angular.io/api/common/NgIf)`. There's a reason.  `[NgIf](https://angular.io/api/common/NgIf)`  refers to the directive  _class_;  `[ngIf](https://angular.io/api/common/NgIf)`  refers to the directive's  _attribute name_.

A directive  _class_  is spelled in  _UpperCamelCase_  (`[NgIf](https://angular.io/api/common/NgIf)`). A directive's  _attribute name_  is spelled in  _lowerCamelCase_  (`[ngIf](https://angular.io/api/common/NgIf)`). The guide refers to the directive  _class_  when talking about its properties and what the directive does. The guide refers to the  _attribute name_  when describing how you apply the directive to an element in the HTML template.

There are two other kinds of Angular directives, described extensively elsewhere: (1) components and (2) attribute directives.

A  _component_  manages a region of HTML in the manner of a native HTML element. Technically it's a directive with a template.

An  [_attribute_  directive](https://angular.io/guide/attribute-directives)  changes the appearance or behavior of an element, component, or another directive. For example, the built-in  [`NgStyle`](https://angular.io/guide/template-syntax#ngStyle)  directive changes several element styles at the same time.

You can apply many  _attribute_  directives to one host element. **You can  [only apply one](https://angular.io/guide/structural-directives#one-per-element)  _structural_  directive to a host element.**

## NgIf case study
`[NgIf](https://angular.io/api/common/NgIf)` is the simplest structural directive and the easiest to understand. It takes a boolean expression and makes an entire chunk of the DOM appear or disappear.
The `[ngIf](https://angular.io/api/common/NgIf)` directive doesn't hide elements with CSS. It adds and removes them physically from the DOM.
When the condition is false, `[NgIf](https://angular.io/api/common/NgIf)` removes its host element from the DOM, detaches it from DOM events (the attachments that it made), detaches the component from Angular change detection, and destroys it. The component and DOM nodes can be garbage-collected and free up memory.

> The difference between hiding and removing doesn't matter for a simple
> paragraph. It does matter when the host element is attached to a
> resource intensive component. Such a component's behavior continues
> even when hidden. The component stays attached to its DOM element. It
> keeps listening to events. Angular keeps checking for changes that
> could affect data bindings. Whatever the component was doing, it keeps
> doing.
> 
> Although invisible, the component—and all of its descendant
> components—tie up resources. The performance and memory burden can be
> substantial, responsiveness can degrade, and the user sees nothing.
> 
> On the positive side, showing the element again is quick. The
> component's previous state is preserved and ready to display. The
> component doesn't re-initialize—an operation that could be expensive.
> So hiding and showing is sometimes the right thing to do.
> 
> But in the absence of a compelling reason to keep them around, your
> preference should be to remove DOM elements that the user can't see
> and recover the unused resources with a structural directive like 
> `[NgIf](https://angular.io/api/common/NgIf)`  .
> 
> **These same considerations apply to every structural directive, whether built-in or custom.**  Before applying a structural directive,
> you might want to pause for a moment to consider the consequences of
> adding and removing elements and of creating and destroying
> components.


## The asterisk (*) prefix
The asterisk is "syntactic sugar" for something a bit more complicated. Internally, Angular translates the `*[ngIf](https://angular.io/api/common/NgIf)`  _attribute_ into a `<ng-template>`  _element_, wrapped around the host element, like this.

    <div *ngIf="hero" class="name">{{hero.name}}</div>

    <ng-template [ngIf]="hero">
      <div class="name">{{hero.name}}</div>
    </ng-template>

-   The  `*[ngIf](https://angular.io/api/common/NgIf)`  directive moved to the  `<ng-template>`  element where it became a property binding,`[[ngIf](https://angular.io/api/common/NgIf)]`.
-   The rest of the  `<div>`, including its class attribute, moved inside the  `<ng-template>`  element.


## Inside  _*ngFor_

The `NgFor` directive has more features, both required and optional. Everything _outside_ the `[ngFor](https://angular.io/api/common/NgForOf)` string stays with the host element (the `<div>`) as it moves inside the `<ng-template>`.

## Microsyntax
The Angular microsyntax lets you configure a directive in a compact, friendly string. **The microsyntax parser translates that string into attributes on the** `<ng-template>`:
-   The  `let`  keyword declares a  [_template input variable_](https://angular.io/guide/structural-directives#template-input-variable)  that you reference within the template. The input variables in this example are  `hero`,  `i`, and  `[odd](https://angular.io/api/common/NgForOfContext#odd)`. The parser translates  `let hero`,  `let i`, and  `let  [odd](https://angular.io/api/common/NgForOfContext#odd)`  into variables named  `let-hero`,  `let-i`, and  `let-odd`.
    
-   The microsyntax parser title-cases all directives and prefixes them with the directive's attribute name, such as  `[ngFor](https://angular.io/api/common/NgForOf)`. For example, the  `[ngFor](https://angular.io/api/common/NgForOf)`  input properties,  `of`  and  `trackBy`, become  `[ngForOf](https://angular.io/api/common/NgForOf)`  and  `[ngForTrackBy](https://angular.io/api/common/NgForOf#ngForTrackBy)`, respectively. That's how the directive learns that the list is  `heroes`  and the track-by function is  `[trackById](https://angular.io/api/core/IterableChangeRecord#trackById)`.

-  As the  `NgFor`  directive loops through the list, it sets and resets properties of its own  _context_  object. These properties can include, but aren't limited to,  `index`,  `[odd](https://angular.io/api/common/NgForOfContext#odd)`, and a special property named  `$implicit`.
    
-   The  `let-i`  and  `let-odd`  variables were defined as  `let i=index`  and  `let  [odd](https://angular.io/api/common/NgForOfContext#odd)=[odd](https://angular.io/api/common/NgForOfContext#odd)`. Angular sets them to the current value of the context's  `index`  and  `[odd](https://angular.io/api/common/NgForOfContext#odd)`  properties.
    
-   The context property for  `let-hero`  wasn't specified. Its intended source is implicit. Angular sets  `let-hero`  to the value of the context's  `$implicit`  property, which  `NgFor`  has initialized with the hero for the current iteration.
    
-   The  [`NgFor`  API guide](https://angular.io/api/common/NgForOf "API: NgFor")  describes additional  `NgFor`  directive properties and context properties.
    
-   The  `[NgForOf](https://angular.io/api/common/NgForOf)`  directive implements  `NgFor`. Read more about additional  `[NgForOf](https://angular.io/api/common/NgForOf)`  directive properties and context properties in the  [NgForOf API reference](https://angular.io/api/common/NgForOf).

### Grammar

When you write your own structural directives, use the following grammar:

    *:prefix="( :let | :expression ) (';' | ',')? ( :let | :as | :keyExp )*"

The following tables describe each portion of the microsyntax grammar.

`prefix`

HTML attribute key

`key`

HTML attribute key

`local`

local variable name used in the template

`export`

value exported by the directive under a given name

`expression`

standard Angular expression

`keyExp = :key ":"? :expression ("as" :local)? ";"?`

`let = "let" :local "=" :export ";"?`

`as = :export "as" :local ";"?`

### [Translation](https://angular.io/guide/structural-directives#translation)

### [Microsyntax examples](https://angular.io/guide/structural-directives#microsyntax-examples)


## Template input variable

A  _template input variable_  is a variable whose value you can reference  _within_  a single instance of the template. There are several such variables in this example:  `hero`,  `i`, and  `[odd](https://angular.io/api/common/NgForOfContext#odd)`. All are preceded by the keyword  `let`.

A  _template input variable_  is  **_not_**  the same as a  [template  _reference_  variable](https://angular.io/guide/template-syntax#ref-vars), neither  _semantically_  nor  _syntactically_.

You declare a template _input_ variable using the `let` keyword (`let hero`). The variable's scope is limited to a _single instance_ of the repeated template. You can use the same variable name again in the definition of other structural directives.



You declare a template _reference_ variable by prefixing the variable name with `#` (`#var`). A _reference_ variable refers to its attached element, component or directive. It can be accessed _anywhere_ in the _entire template_.
Template _input_ and _reference_ variable names have their own namespaces. The `hero` in `let hero` is never the same variable as the `hero` declared as `#hero`.

## One structural directive per host element

Someday you'll want to repeat a block of HTML but only when a particular condition is true. You'll  _try_  to put both an  `*[ngFor](https://angular.io/api/common/NgForOf)`  and an  `*[ngIf](https://angular.io/api/common/NgIf)`  on the same host element. Angular won't let you. You may apply only one  _structural_  directive to an element.

The reason is simplicity. Structural directives can do complex things with the host element and its descendents. When two directives lay claim to the same host element, which one takes precedence? Which should go first, the  `[NgIf](https://angular.io/api/common/NgIf)`  or the  `NgFor`? Can the  `[NgIf](https://angular.io/api/common/NgIf)`  cancel the effect of the  `NgFor`? If so (and it seems like it should be so), how should Angular generalize the ability to cancel for other structural directives?

There are no easy answers to these questions. Prohibiting multiple structural directives makes them moot. There's an easy solution for this use case: **put the  `*[ngIf](https://angular.io/api/common/NgIf)`  on a container element that wraps the  `*[ngFor](https://angular.io/api/common/NgForOf)`  element. One or both elements can be an  [`ng-container`](https://angular.io/guide/structural-directives#ngcontainer)  so you don't have to introduce extra levels of HTML.**


## Inside  _NgSwitch_  directives
The Angular _NgSwitch_ is actually a set of cooperating directives: `[NgSwitch](https://angular.io/api/common/NgSwitch)`, `[NgSwitchCase](https://angular.io/api/common/NgSwitchCase)`, and `[NgSwitchDefault](https://angular.io/api/common/NgSwitchDefault)`.

Example:

    <div [ngSwitch]="hero?.emotion">
      <app-happy-hero    *ngSwitchCase="'happy'"    [hero]="hero"></app-happy-hero>
      <app-sad-hero      *ngSwitchCase="'sad'"      [hero]="hero"></app-sad-hero>
      <app-confused-hero *ngSwitchCase="'confused'" [hero]="hero"></app-confused-hero>
      <app-unknown-hero  *ngSwitchDefault           [hero]="hero"></app-unknown-hero>
    </div>

`[NgSwitch](https://angular.io/api/common/NgSwitch)`  itself is not a structural directive. It's an  _attribute_  directive that controls the behavior of the other two switch directives. That's why you write  `[[ngSwitch](https://angular.io/api/common/NgSwitch)]`, never  `*[ngSwitch](https://angular.io/api/common/NgSwitch)`.

`[NgSwitchCase](https://angular.io/api/common/NgSwitchCase)`  and  `[NgSwitchDefault](https://angular.io/api/common/NgSwitchDefault)`  _are_  structural directives. You attach them to elements using the asterisk (*) prefix notation. An  `[NgSwitchCase](https://angular.io/api/common/NgSwitchCase)`  displays its host element when its value matches the switch value. The  `[NgSwitchDefault](https://angular.io/api/common/NgSwitchDefault)`  displays its host element when no sibling  `[NgSwitchCase](https://angular.io/api/common/NgSwitchCase)`  matches the switch value.

The element to which you apply a directive is its _host_ element. The `<happy-hero>` is the host element for the happy `*[ngSwitchCase](https://angular.io/api/common/NgSwitchCase)`. The `<unknown-hero>` is the host element for the `*[ngSwitchDefault](https://angular.io/api/common/NgSwitchDefault)`.

As with other structural directives, the `[NgSwitchCase](https://angular.io/api/common/NgSwitchCase)` and `[NgSwitchDefault](https://angular.io/api/common/NgSwitchDefault)` can be desugared into the `<ng-template>` element form.
