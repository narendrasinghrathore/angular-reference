# Dependency Providers
A dependency  [provider](https://angular.io/guide/glossary#provider)  configures an injector with a  [DI token](https://angular.io/guide/glossary#di-token), which that injector uses to provide the concrete, runtime version of a dependency value. The injector relies on the provider configuration to create instances of the dependencies that it injects into components, directives, pipes, and other services.

You must configure an injector with a provider, or it won't know how to create the dependency. The most obvious way for an injector to create an instance of a service class is with the class itself. If you specify the service class itself as the provider token, the default behavior is for the injector to instantiate that class with  `new`.

## The  Provider  object literal

The class-provider syntax is a shorthand expression that expands into a provider configuration, defined by the  `Provider` . The following code snippets shows how a class that is given as the  `providers`  value is expanded into a full provider object.
The expanded provider configuration is an object literal with two properties.

-   The  `provide`  property holds the  [token](https://angular.io/guide/dependency-injection#token)  that serves as the key for both locating a dependency value and configuring the injector.
    
-   The second property is a provider definition object, which tells the injector how to create the dependency value. The provider-definition key can be  `useClass`, as in the example. It can also be  `[useExisting](https://angular.io/api/core/ExistingSansProvider#useExisting)`,  `[useValue](https://angular.io/api/core/ValueSansProvider#useValue)`, or  `[useFactory](https://angular.io/api/core/FactorySansProvider#useFactory)`. Each of these keys provides a different type of dependency, as discussed below.
