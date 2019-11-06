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
