
# Angular mini reference guide


## Dependency Injection

Dependencies are services or objects that a class needs to perform its function. DI is a coding pattern in which a class asks for dependencies from external sources rather than creating them itself.
Having multiple classes in the same file can be confusing. We generally recommend that you define components and services in separate files.

If you do combine a component and service in the same file, it is important to define the service first, and then the component. If you define the component before the service, you get a run-time null reference error.
 The @Injectable() decorator marks it as a service that can be injected, but Angular can't actually inject it anywhere until you configure an Angular dependency injector with a provider of that service.

Injectors are inherited, which means that if a given injector can't resolve a dependency, it asks the parent injector to resolve it. A component can get services from its own injector, from the injectors of its component ancestors, from the injector of its parent NgModule, or from the root injector.

You can configure injectors with providers at different levels of your app, by setting a metadata value in one of three places:

 - In the @Injectable() decorator for the service itself.
 - In the @NgModule() decorator for an NgModule.
 - In the @Component() decorator for a component.
 
 
 The @Injectable() decorator has the providedIn metadata option, where you can specify the provider of the decorated service class with the root injector, or with the injector for a specific NgModule.

The @NgModule() and @Component() decorators have the providers metadata option, where you can configure providers for NgModule-level or component-level injectors.
 
Components are directives, and the providers option is inherited from @Directive(). You can also configure providers for directives and pipes at the same level as the component.


### Injector hierarchy and service instances
Services are singletons within the scope of an injector. That is, there is at most one instance of a service in a given injector.

There is only one root injector for an app. Providing UserService at the root or AppModule level means it is registered with the root injector. There is just one UserService instance in the entire app and every class that injects UserService gets this service instance unless you configure another provider with a child injector.

Angular DI has a hierarchical injection system, which means that nested injectors can create their own service instances. Angular regularly creates nested injectors. Whenever Angular creates a new instance of a component that has providers specified in @Component(), it also creates a new child injector for that instance. Similarly, when a new NgModule is lazy-loaded at run time, Angular can create an injector for it with its own providers.

Child modules and component injectors are independent of each other, and create their own separate instances of the provided services. When Angular destroys an NgModule or component instance, it also destroys that injector and that injector's service instances.

Thanks to injector inheritance, you can still inject application-wide services into these components. A component's injector is a child of its parent component's injector, and inherits from all ancestor injectors all the way back to the application's root injector. Angular can inject a service provided by any injector in that lineage.

When Angular creates a class whose constructor has parameters, it looks for type and injection metadata about those parameters so that it can inject the correct service. If Angular can't find that parameter information, it throws an error. Angular can only find the parameter information if the class has a decorator of some kind. The @Injectable() decorator is the standard decorator for service classes.
The decorator requirement is imposed by TypeScript. TypeScript normally discards parameter type information when it transpiles the code to JavaScript. TypeScript preserves this information if the class has a decorator and the emitDecoratorMetadata compiler option is set true in TypeScript's tsconfig.json configuration file. The CLI configures tsconfig.json with emitDecoratorMetadata: true.

This means you're responsible for putting @Injectable() on your service classes.


### Dependency injection tokens
When you configure an injector with a provider, you associate that provider with a DI token. The injector maintains an internal token-provider map that it references when asked for a dependency. The token is the key to the map.

In simple examples, the dependency value is an instance, and the class type serves as its own lookup key. Here you get a HeroService directly from the injector by supplying the HeroService type as the token:

### Optional dependencies
When a component or service declares a dependency, the class constructor takes that dependency as a parameter. You can tell Angular that the dependency is optional by annotating the constructor parameter with @Optional().

## Hierarchical injectors

Two injector hierarchies
There are two injector hierarchies in Angular:

 - ModuleInjector hierarchy—configure a ModuleInjector in this hierarchy using an @NgModule() or @Injectable() annotation.
 - ElementInjector hierarchy—created implicitly at each DOM element. An ElementInjector is empty by default unless you configure it in the providers property on @Directive() or @Component().
 
### ModuleInjector
The ModuleInjector can be configured in one of two ways:

Using the @Injectable() providedIn property to refer to @NgModule(), or root.
Using the @NgModule() providers array.
 
#### Tree-shaking and @Injectable()
Using the @Injectable() providedIn property is preferable to the @NgModule() providers array because with @Injectable() providedIn, optimization tools can perform tree-shaking, which removes services that your app isn't using and results in smaller bundle sizes.

Tree-shaking is especially useful for a library because the application which uses the library may not have a need to inject it.
Child ModuleInjectors are created when lazy loading other @NgModules.
The @Injectable() decorator identifies a service class. The providedIn property configures a specific ModuleInjector, here root, which makes the service available in the root ModuleInjector.

#### Platform injector
There are two more injectors above root, an additional ModuleInjector and NullInjector().

Consider how Angular bootstraps the app with the following in main.ts:

  platformBrowserDynamic().bootstrapModule(AppModule).then(ref => {...})
  
The bootstrapModule() method creates a child injector of the platform injector which is configured by the AppModule. This is the root ModuleInjector.

The platformBrowserDynamic() method creates an injector configured by a PlatformModule, which contains platform-specific dependencies. This allows multiple apps to share a platform configuration. For example, a browser has only one URL bar, no matter how many apps you have running. You can configure additional platform-specific providers at the platform level by supplying extraProviders using the platformBrowser() function.

The next parent injector in the hierarchy is the NullInjector(), which is the top of the tree. If you've gone so far up the tree that you are looking for a service in the NullInjector(), you'll get an error unless you've used @Optional() because ultimately, everything ends at the NullInjector() and it returns an error or, in the case of @Optional(), null. https://angular.io/generated/images/guide/dependency-injection/injectors.svg

<img src="https://angular.io/generated/images/guide/dependency-injection/injectors.svg" width="400"/>

While the name root is a special alias, other ModuleInjectors don't have aliases. You have the option to create ModuleInjectors whenever a dynamically loaded component is created, such as with the Router, which will create child ModuleInjectors.

All requests forward up to the root injector, whether you configured it with the bootstrapModule() method, or registered all providers with root in their own services.

If you configure an app-wide provider in the @NgModule() of AppModule, it overrides one configured for root in the @Injectable() metadata. You can do this to configure a non-default provider of a service that is shared with multiple apps.

### ElementInjector
Angular creates ElementInjectors implicitly for each DOM element.
Providing a service in the @Component() decorator using its providers or viewProviders property configures an ElementInjector.

When you provide services in a component, that service is available via the ElementInjector at that component instance. It may also be visible at child component/directives based on visibility rules described in the resolution rules section. When the component instance is destroyed, so is that service instance.

#### @Directive() and @Component()
A component is a special type of directive, which means that just as @Directive() has a providers property, @Component() does too. This means that directives as well as components can configure providers, using the providers property. When you configure a provider for a component or directive using the providers property, that provider belongs to the ElementInjector of that component or directive. Components and directives on the same element share an injector.

#### Resolution rules
When resolving a token for a component/directive, Angular resolves it in two phases:

 - Against the ElementInjector hierarchy (its parents)
 - Against the ModuleInjector hierarchy (its parents)
 
 When a component declares a dependency, Angular tries to satisfy that dependency with its own ElementInjector. If the component's injector lacks the provider, it passes the request up to its parent component's ElementInjector.

The requests keep forwarding up until Angular finds an injector that can handle the request or runs out of ancestor ElementInjectors.

If Angular doesn't find the provider in any ElementInjectors, it goes back to the element where the request originated and looks in the ModuleInjector hierarchy. If Angular still doesn't find the provider, it throws an error.

If you have registered a provider for the same DI token at different levels, the first one Angular encounters is the one it uses to resolve the dependency. If, for example, a provider is registered locally in the component that needs a service, Angular doesn't look for another provider of the same service.

#### Resolution modifiers
Angular's resolution behavior can be modified with @Optional(), @Self(), @SkipSelf() and @Host(). Import each of them from @angular/core and use each in the component class constructor when you inject your service.

#### Types of modifiers
Resolution modifiers fall into three categories:

 - What to do if Angular doesn't find what you're looking for, that is @Optional()
 - Where to start looking, that is @SkipSelf()
 - Where to stop looking, @Host() and @Self()

By default, Angular always starts at the current Injector and keeps searching all the way up. Modifiers allow you to change the starting (self) or ending location.

Additionally, you can combine all of the modifiers except @Host() and @Self() and of course @Skipself() and @Self().

#### @Optional()
@Optional() allows Angular to consider a service you inject to be optional. This way, if it can't be resolved at runtime, Angular simply resolves the service as null, rather than throwing an error. 

#### @Self()
Use @Self() so that Angular will only look at the ElementInjector for the current component or directive.

A good use case for @Self() is to inject a service but only if it is available on the current host element. To avoid errors in this situation, combine @Self() with @Optional().

#### @SkipSelf()
@SkipSelf() is the opposite of @Self(). With @SkipSelf(), Angular starts its search for a service in the parent ElementInjector, rather than in the current one. So if the parent ElementInjector were using the value 🌿 (fern) for emoji , but you had 🍁 (maple leaf) in the component's providers array, Angular would ignore 🍁 (maple leaf) and use 🌿 (fern).

#### @SkipSelf() with @Optional()
Use @SkipSelf() with @Optional() to prevent an error if the value is null. In the following example, the Person service is injected in the constructor. @SkipSelf() tells Angular to skip the current injector and @Optional() will prevent an error should the Person service be null.

  class Person {
    constructor(@Optional() @SkipSelf() parent: Person) {}
  }
  
#### @Host()
@Host() lets you designate a component as the last stop in the injector tree when searching for providers. Even if there is a service instance further up the tree, Angular won't continue looking.

  @Component({
    selector: 'app-host',
    templateUrl: './host.component.html',
    styleUrls: ['./host.component.css'],
    //  provide the service
    providers: [{ provide: FlowerService, useValue: { emoji: '🌼' } }]
  })
  export class HostComponent {
    // use @Host() in the constructor when injecting the service
    constructor(@Host() @Optional() public flower: FlowerService) { }

  }
  
Since HostComponent has @Host() in its constructor, no matter what the parent of HostComponent might have as a flower.emoji value, the HostComponent will use 🌼 (yellow flower).

## Logical structure of the template
When you provide services in the component class, services are visible within the  `ElementInjector`  tree relative to where and how you provide those services.

Understanding the underlying logical structure of the Angular template will give you a foundation for configuring services and in turn control their visibility.

Components are used in your templates, as in the following example:

    <app-root>
        <app-child></app-child>
    </app-root>

**Note:** Usually, you declare the components and their templates in separate files. For the purposes of understanding how the injection system works, it is useful to look at them from the point of view of a combined logical tree. The term logical distinguishes it from the render tree (your application DOM tree). To mark the locations of where the component templates are located, this guide uses the `<#VIEW>` pseudo element, which doesn't actually exist in the render tree and is present for mental model purposes only.

The following is an example of how the `<app-root>` and `<app-child>` view trees are combined into a single logical tree:


    <app-root>
      <#VIEW>
        <app-child>
         <#VIEW>
           ...content goes here...
         </#VIEW>
        </app-child>
      <#VIEW>
    </app-root>

## Providing services in  `@[Component](https://angular.io/api/core/Component)()`[](https://angular.io/guide/hierarchical-dependency-injection#providing-services-in-component "Link to this heading")

How you provide services via an  `@[Component](https://angular.io/api/core/Component)()`  (or  `@[Directive](https://angular.io/api/core/Directive)()`) decorator determines their visibility. The following sections demonstrate  `providers`  and  `[viewProviders](https://angular.io/api/core/Component#viewProviders)`  along with ways to modify service visibility with  `@[SkipSelf](https://angular.io/api/core/SkipSelf)()`  and  `@[Host](https://angular.io/api/core/Host)()`.

A component class can provide services in two ways:

1.  with a  `providers`  array

    @Component({
      ...
      providers: [
        {provide: FlowerService, useValue: {emoji: '🌺'}}
      ]
    })

2.  with a  `[viewProviders](https://angular.io/api/core/Component#viewProviders)`  array

    @Component({
      ...
      viewProviders: [
        {provide: AnimalService, useValue: {emoji: '🐶'}}
      ]
    })

**NOTE:**  In the logical tree, you'll see  `@Provide`,  `@[Inject](https://angular.io/api/core/Inject)`, and  `@[NgModule](https://angular.io/api/core/NgModule)`, which are not real HTML attributes but are here to demonstrate what is going on under the hood.

-   `@[Inject](https://angular.io/api/core/Inject)(Token)=>[Value](https://angular.io/)`  demonstrates that if  `Token`  is injected at this location in the logical tree its value would be  `[Value](https://angular.io/)`.
-   `@Provide(Token=[Value](https://angular.io/))`  demonstrates that there is a declaration of  `Token`  provider with value  `[Value](https://angular.io/)`  at this location in the logical tree.
-   `@[NgModule](https://angular.io/api/core/NgModule)(Token)`  demonstrates that a fallback  `[NgModule](https://angular.io/api/core/NgModule)`  injector should be used at this location.


### Example app structure[](https://angular.io/guide/hierarchical-dependency-injection#example-app-structure "Link to this heading")

The example app has a  `FlowerService`  provided in  `root`  with an  `emoji`  value of  `🌺`  (red hibiscus).


    @Injectable({
      providedIn: 'root'
    })
    export class FlowerService {
      emoji = '🌺';
    }

Consider a simple app with only an `AppComponent` and a `ChildComponent`. The most basic rendered view would look like nested HTML elements such as the following:

    <app-root> <!-- AppComponent selector -->
        <app-child> <!-- ChildComponent selector -->
        </app-child>
    </app-root>

However, behind the scenes, Angular uses a logical view representation as follows when resolving injection requests:

    <app-root> <!-- AppComponent selector -->
        <#VIEW>
            <app-child> <!-- ChildComponent selector -->
                <#VIEW>
                </#VIEW>
            </app-child>
        </#VIEW>
    </app-root>

The  `<#VIEW>`  here represents an instance of a template. Notice that each component has its own  `<#VIEW>`.

Knowledge of this structure can inform how you provide and inject your services, and give you complete control of service visibility.

Now, consider that  `<app-root>`  simply injects the  `FlowerService`:

    export class AppComponent  {
      constructor(public flower: FlowerService) {}
    }
Add a binding to the `<app-root>` template to visualize the result:

    <p>Emoji from FlowerService: {{flower.emoji}}</p>

The output in the view would be:

    Emoji from FlowerService: 🌺

In the logical tree, this would be represented as follows:

    <app-root @NgModule(AppModule)
            @Inject(FlowerService) flower=>"🌺">
      <#VIEW>
        <p>Emoji from FlowerService: {{flower.emoji}} (🌺)</p>
        <app-child>
          <#VIEW>
          </#VIEW>
         </app-child>
      </#VIEW>
    </app-root>

When  `<app-root>`  requests the  `FlowerService`, it is the injector's job to resolve the  `FlowerService`  token. The resolution of the token happens in two phases:

1.  The injector determines the starting location in the logical tree and an ending location of the search. The injector begins with the starting location and looks for the token at each level in the logical tree. If the token is found it is returned.
2.  If the token is not found, the injector looks for the closest parent  `@[NgModule](https://angular.io/api/core/NgModule)()`  to delegate the request to.

In the example case, the constraints are:

1.  Start with  `<#VIEW>`  belonging to  `<app-root>`  and end with  `<app-root>`.

-   Normally the starting point for search is at the point of injection. However, in this case  `<app-root>`  `@[Component](https://angular.io/api/core/Component)`s are special in that they also include their own  `[viewProviders](https://angular.io/api/core/Component#viewProviders)`, which is why the search starts at  `<#VIEW>`  belonging to  `<app-root>`. (This would not be the case for a directive matched at the same location).
-   The ending location just happens to be the same as the component itself, because it is the topmost component in this application.

2.  The  `AppModule`  acts as the fallback injector when the injection token can't be found in the  `ElementInjector`s.

### Using the  `providers`  array[](https://angular.io/guide/hierarchical-dependency-injection#using-the-providers-array "Link to this heading")

Now, in the  `ChildComponent`  class, add a provider for  `FlowerService`  to demonstrate more complex resolution rules in the upcoming sections:

    @Component({
      selector: 'app-child',
      templateUrl: './child.component.html',
      styleUrls: ['./child.component.css'],
      // use the providers array to provide a service
      providers: [{ provide: FlowerService, useValue: { emoji: '🌻' } }]
    })
    
    export class ChildComponent {
      // inject the service
      constructor( public flower: FlowerService) { }
    }

Now that the  `FlowerService`  is provided in the  `@[Component](https://angular.io/api/core/Component)()`  decorator, when the  `<app-child>`  requests the service, the injector has only to look as far as the  `<app-child>`'s own  `ElementInjector`. It won't have to continue the search any further through the injector tree.

The next step is to add a binding to the  `ChildComponent`  template.

    <p>Emoji from FlowerService: {{flower.emoji}}</p>
To render the new values, add `<app-child>` to the bottom of the `AppComponent` template so the view also displays the sunflower:

    Child Component
    Emoji from FlowerService: 🌻

In the logical tree, this would be represented as follows:

    <app-root @NgModule(AppModule)
            @Inject(FlowerService) flower=>"🌺">
      <#VIEW>
        <p>Emoji from FlowerService: {{flower.emoji}} (🌺)</p>
        <app-child @Provide(FlowerService="🌻")
                   @Inject(FlowerService)=>"🌻"> <!-- search ends here -->
          <#VIEW> <!-- search starts here -->
            <h2>Parent Component</h2>
            <p>Emoji from FlowerService: {{flower.emoji}} (🌻)</p>
          </#VIEW>
         </app-child>
      </#VIEW>
    </app-root>
When `<app-child>` requests the `FlowerService`, the injector begins its search at the `<#VIEW>` belonging to `<app-child>` (`<#VIEW>` is included because it is injected from `@[Component](https://angular.io/api/core/Component)()`) and ends with `<app-child>`. In this case, the `FlowerService` is resolved in the `<app-child>`'s `providers` array with sunflower 🌻. The injector doesn't have to look any further in the injector tree. It stops as soon as it finds the `FlowerService` and never sees the 🌺 (red hibiscus).

### Using the  `[viewProviders](https://angular.io/api/core/Component#viewProviders)`  array[](https://angular.io/guide/hierarchical-dependency-injection#using-the-viewproviders-array "Link to this heading")

Use the  `[viewProviders](https://angular.io/api/core/Component#viewProviders)`  array as another way to provide services in the  `@[Component](https://angular.io/api/core/Component)()`  decorator. Using  `[viewProviders](https://angular.io/api/core/Component#viewProviders)`  makes services visibile in the  `<#VIEW>`.

The steps are the same as using the  `providers`  array, with the exception of using the  `[viewProviders](https://angular.io/api/core/Component#viewProviders)`  array instead.

For step-by-step instructions, continue with this section. If you can set it up on your own, skip ahead to  [Modifying service availability](https://angular.io/guide/hierarchical-dependency-injection#modify-visibility).

The example app features a second service, the  `AnimalService`  to demonstrate  `[viewProviders](https://angular.io/api/core/Component#viewProviders)`.

First, create an  `AnimalService`  with an  `emoji`  property of 🐳 (whale):

    import { Injectable } from '@angular/core';
    
    @Injectable({
      providedIn: 'root'
    })
    export class AnimalService {
      emoji = '🐳';
    }
Following the same pattern as with the `FlowerService`, inject the `AnimalService` in the `AppComponent` class:

    export class AppComponent  {
      constructor(public flower: FlowerService, public animal: AnimalService) {}
    }
**Note:** You can leave all the `FlowerService` related code in place as it will allow a comparison with the `AnimalService`.

Add a `[viewProviders](https://angular.io/api/core/Component#viewProviders)` array and inject the `AnimalService` in the `<app-child>` class, too, but give `emoji` a different value. Here, it has a value of 🐶 (puppy).

    @Component({
      selector: 'app-child',
      templateUrl: './child.component.html',
      styleUrls: ['./child.component.css'],
      // provide services
      providers: [{ provide: FlowerService, useValue: { emoji: '🌻' } }],
      viewProviders: [{ provide: AnimalService, useValue: { emoji: '🐶' } }]
    })
    
    export class ChildComponent {
      // inject service
      constructor( public flower: FlowerService, public animal: AnimalService) { }
    }
Add bindings to the `ChildComponent` and the `AppComponent` templates. In the `ChildComponent` template, add the following binding:

    <p>Emoji from AnimalService: {{animal.emoji}}</p>

 Additionally, add the same to the `AppComponent` template:

    <p>Emoji from AnimalService: {{animal.emoji}}</p>
Now you should see both values in the browser:

    AppComponent
    Emoji from AnimalService: 🐳
    
    Child Component
    Emoji from AnimalService: 🐶
The logic tree for this example of `[viewProviders](https://angular.io/api/core/Component#viewProviders)` is as follows:

    <app-root @NgModule(AppModule)
            @Inject(AnimalService) animal=>"🐳">
      <#VIEW>
        <app-child>
          <#VIEW
           @Provide(AnimalService="🐶")
           @Inject(AnimalService=>"🐶")>
           <!-- ^^using viewProviders means AnimalService is available in <#VIEW>-->
           <p>Emoji from AnimalService: {{animal.emoji}} (🐶)</p>
          </#VIEW>
         </app-child>
      </#VIEW>
    </app-root>
Just as with the  `FlowerService`  example, the  `AnimalService`  is provided in the  `<app-child>`  `@[Component](https://angular.io/api/core/Component)()`  decorator. This means that since the injector first looks in the  `ElementInjector`  of the component, it finds the  `AnimalService`  value of 🐶 (puppy). It doesn't need to continue searching the  `ElementInjector`  tree, nor does it need to search the  `ModuleInjector`.

### `providers`  vs.  `[viewProviders](https://angular.io/api/core/Component#viewProviders)`[](https://angular.io/guide/hierarchical-dependency-injection#providers-vs-viewproviders "Link to this heading")

To see the difference between using  `providers`  and  `[viewProviders](https://angular.io/api/core/Component#viewProviders)`, add another component to the example and call it  `InspectorComponent`.  `InspectorComponent`  will be a child of the  `ChildComponent`. In  `inspector.component.ts`, inject the  `FlowerService`  and  `AnimalService`  in the constructor:

    export class InspectorComponent {
      constructor(public flower: FlowerService, public animal: AnimalService) { }
    }
You do not need a `providers` or `[viewProviders](https://angular.io/api/core/Component#viewProviders)` array. Next, in `inspector.component.html`, add the same markup from previous components:

    <p>Emoji from FlowerService: {{flower.emoji}}</p>
    <p>Emoji from AnimalService: {{animal.emoji}}</p>
Remember to add the `InspectorComponent` to the `AppModule`  `declarations` array.

    @NgModule({
      imports:      [ BrowserModule, FormsModule ],
      declarations: [ AppComponent, ChildComponent, InspectorComponent ],
      bootstrap:    [ AppComponent ],
      providers: []
    })
    export class AppModule { }
Next, make sure your `child.component.html` contains the following:

    <p>Emoji from FlowerService: {{flower.emoji}}</p>
    <p>Emoji from AnimalService: {{animal.emoji}}</p>
    
    <div class="container">
      <h3>Content projection</h3>
    	<ng-content></ng-content>
    </div>
    
    <h3>Inside the view</h3>
    <app-inspector></app-inspector>
The first two lines, with the bindings, are there from previous steps. The new parts are  `<ng-content>`  and  `<app-inspector>`.  `<ng-content>`  allows you to project content, and  `<app-inspector>`  inside the  `ChildComponent`  template makes the  `InspectorComponent`  a child component of  `ChildComponent`.

Next, add the following to  `app.component.html`  to take advantage of content projection.

    <app-child><app-inspector></app-inspector></app-child>
    
The browser now renders the following, omitting the previous examples for brevity:

    //...Omitting previous examples. The following applies to this section.
    
    Content projection: This is coming from content. Doesn't get to see
    puppy because the puppy is declared inside the view only.
    
    Emoji from FlowerService: 🌻
    Emoji from AnimalService: 🐳
    
    Emoji from FlowerService: 🌻
    Emoji from AnimalService: 🐶

These four bindings demonstrate the difference between  `providers`  and  `[viewProviders](https://angular.io/api/core/Component#viewProviders)`. Since the 🐶 (puppy) is declared inside the <#VIEW>, it isn't visible to the projected content. Instead, the projected content sees the 🐳 (whale).

The next section though, where  `InspectorComponent`  is a child component of  `ChildComponent`,  `InspectorComponent`  is inside the  `<#VIEW>`, so when it asks for the  `AnimalService`, it sees the 🐶 (puppy).

The  `AnimalService`  in the logical tree would look like this:

    <app-root @NgModule(AppModule)
            @Inject(AnimalService) animal=>"🐳">
      <#VIEW>
        <app-child>
          <#VIEW
           @Provide(AnimalService="🐶")
           @Inject(AnimalService=>"🐶")>
           <!-- ^^using viewProviders means AnimalService is available in <#VIEW>-->
           <p>Emoji from AnimalService: {{animal.emoji}} (🐶)</p>
           <app-inspector>
            <p>Emoji from AnimalService: {{animal.emoji}} (🐶)</p>
           </app-inspector>
          </#VIEW>
          <app-inspector>
            <#VIEW>
              <p>Emoji from AnimalService: {{animal.emoji}} (🐳)</p>
            </#VIEW>
          </app-inspector>
         </app-child>
      </#VIEW>
    </app-root>

The projected content of `<app-inspector>` sees the 🐳 (whale), not the 🐶 (puppy), because the 🐶 (puppy) is inside the `<app-child>`  `<#VIEW>`. The `<app-inspector>` can only see the 🐶 (puppy) if it is also within the `<#VIEW>`.






## NgModules
An NgModule is a class marked by the @NgModule decorator. @NgModule takes a metadata object that describes how to compile a component's template and how to create an injector at runtime. It identifies the module's own components, directives, and pipes, making some of them public, through the exports property, so that external components can use them. @NgModule can also add service providers to the application dependency injectors.

Angular libraries are NgModules, such as FormsModule, HttpClientModule, and RouterModule.

NgModules consolidate components, directives, and pipes into cohesive blocks of functionality, each focused on a feature area, application business domain, workflow, or common collection of utilities.
Modules can also add services to the application. Such services might be internally developed, like something you'd develop yourself or come from outside sources, such as the Angular router and HTTP client.
Modules can be loaded eagerly when the application starts or lazy loaded asynchronously by the router.




## HttpClient

The HttpClient in @angular/common/http offers a simplified client HTTP API for Angular applications that rests on the XMLHttpRequest interface exposed by browsers.
Before you can use the HttpClient, you need to import the Angular HttpClientModule

### Requesting a json file in angular project from assets folder
    configUrl = 'assets/config.json';

    getConfig() {
      return this.http.get(this.configUrl);
    }

#### WHY WRITE A SERVICE?
This example is so simple that it is tempting to write the Http.get() inside the component itself and skip the service. In practice, however, data access rarely stays this simple. You typically need to post-process the data, add error handling, and maybe some retry logic to cope with intermittent connectivity.

The component quickly becomes cluttered with data access minutia. The component becomes harder to understand, harder to test, and the data access logic can't be re-used or standardized.

That's why it's a best practice to separate presentation of data from data access by encapsulating data access in a separate service and delegating to that service in the component, even in simple cases like this one.

    Specifying the response type is a declaration to TypeScript that it should expect your response to be of the given type. 
    This is a build-time check and doesn't guarantee that the server will actually respond with an object of this type. 
    It is up to the server to ensure that the type specified by the server API is returned
    
    
    export interface Config {
      heroesUrl: string;
      textfile: string;
    }
    
    getConfig() {
      // now returns an Observable of Config
      return this.http.get<Config>(this.configUrl);
    }


