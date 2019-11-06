# Dependency Providers
A dependency  [provider](https://angular.io/guide/glossary#provider)  configures an injector with a  [DI token](https://angular.io/guide/glossary#di-token), which that injector uses to provide the concrete, runtime version of a dependency value. The injector relies on the provider configuration to create instances of the dependencies that it injects into components, directives, pipes, and other services.

You must configure an injector with a provider, or it won't know how to create the dependency. The most obvious way for an injector to create an instance of a service class is with the class itself. If you specify the service class itself as the provider token, the default behavior is for the injector to instantiate that class with  `new`.

## The  Provider  object literal

The class-provider syntax is a shorthand expression that expands into a provider configuration, defined by the  `Provider` . The following code snippets shows how a class that is given as the  `providers`  value is expanded into a full provider object.
The expanded provider configuration is an object literal with two properties.

-   The  `provide`  property holds the  [token](https://angular.io/guide/dependency-injection#token)  that serves as the key for both locating a dependency value and configuring the injector.
    
-   The second property is a provider definition object, which tells the injector how to create the dependency value. The provider-definition key can be  `useClass`, as in the example. It can also be  [useExisting](https://angular.io/api/core/ExistingSansProvider#useExisting), [useValue](https://angular.io/api/core/ValueSansProvider#useValue), or  [useFactory](https://angular.io/api/core/FactorySansProvider#useFactory). Each of these keys provides a different type of dependency, as discussed below.

## Alternative class providers
Different classes can provide the same service. For example, the following code tells the injector to return a `BetterLogger` instance when the component asks for a logger using the `Logger` token.

    [{ provide: Logger, useClass: BetterLogger }]
    
### Class providers with dependencies
Another class, `EvenBetterLogger`, might display the user name in the log message. This logger gets the user from an injected `UserService` instance.

    @Injectable()
    export class EvenBetterLogger extends Logger {
      constructor(private userService: UserService) { super(); }
    
      log(message: string) {
        let name = this.userService.user.name;
        super.log(`Message to ${name}: ${message}`);
      }
    }
The injector needs providers for both this new logging service and its dependent `UserService`. Configure this alternative logger with the `useClass` provider-definition key, like `BetterLogger`. The following array specifies both providers in the `providers` metadata option of the parent module or component.


    [ UserService,
      { provide: Logger, useClass: EvenBetterLogger }]

### Aliased class providers

Suppose an old component depends upon the  `OldLogger`  class.  `OldLogger`  has the same interface as  `NewLogger`, but for some reason you can't update the old component to use it.

When the old component logs a message with  `OldLogger`, you want the singleton instance of  `NewLogger`  to handle it instead. In this case, the dependency injector should inject that singleton instance when a component asks for either the new or the old logger.  `OldLogger`  should be an  _alias_  for  `NewLogger`.

If you try to alias  `OldLogger`  to  `NewLogger`  with  `useClass`, you end up with two different  `NewLogger`  instances in your app.

    [ NewLogger,
      // Not aliased! Creates two instances of `NewLogger`
      { provide: OldLogger, useClass: NewLogger}]

To make sure there is only one instance of `NewLogger`, alias `OldLogger` with the `[useExisting](https://angular.io/api/core/ExistingSansProvider#useExisting)` option.

    [ NewLogger,
      // Alias OldLogger w/ reference to NewLogger
      { provide: OldLogger, useExisting: NewLogger}]


## Value providers

Sometimes it's easier to provide a ready-made object rather than ask the injector to create it from a class. To inject an object you have already created, configure the injector with the  `[useValue](https://angular.io/api/core/ValueSansProvider#useValue)`  option

The following code defines a variable that creates such an object to play the logger role.

    // An object in the shape of the logger service
    export function SilentLoggerFn() {}
    
    const silentLogger = {
      logs: ['Silent logger says "Shhhhh!". Provided via "useValue"'],
      log: SilentLoggerFn
    };

The following provider object uses the `[useValue](https://angular.io/api/core/ValueSansProvider#useValue)` key to associate the variable with the `Logger` token.

    [{ provide: Logger, useValue: silentLogger }]

### Non-class dependencies
Not all dependencies are classes. Sometimes you want to inject a string, function, or object.

Apps often define configuration objects with lots of small facts, like the title of the application or the address of a web API endpoint. These configuration objects aren't always instances of a class. They can be object literals, as shown in the following example.

    export const HERO_DI_CONFIG: AppConfig = {
      apiEndpoint: 'api.heroes.com',
      title: 'Dependency Injection'
    };
**TypeScript interfaces are not valid tokens**

The  `HERO_DI_CONFIG`  constant conforms to the  `AppConfig`  interface. Unfortunately, you cannot use a TypeScript interface as a token. In TypeScript, an interface is a design-time artifact, and doesn't have a runtime representation (token) that the DI framework can use.

    // FAIL! Can't use interface as provider token
    [{ provide: AppConfig, useValue: HERO_DI_CONFIG })]

This might seem strange if you're used to dependency injection in strongly typed languages where an interface is the preferred dependency lookup key. However, JavaScript, doesn't have interfaces, so when TypeScript is transpiled to JavaScript, the interface disappears. There is no interface type information left for Angular to find at runtime.

One alternative is to provide and inject the configuration object in an NgModule like `AppModule`.

    providers: [
      UserService,
      { provide: APP_CONFIG, useValue: HERO_DI_CONFIG }
    ],
Another solution to choosing a provider token for non-class dependencies is to define and use an `[InjectionToken](https://angular.io/api/core/InjectionToken)` object. The following example shows how to define such a token.

    import { InjectionToken } from '@angular/core';
    
    export const APP_CONFIG = new InjectionToken<AppConfig>('app.config');

The type parameter, while optional, conveys the dependency's type to developers and tooling. The token description is another developer aid.

Register the dependency provider using the  `[InjectionToken](https://angular.io/api/core/InjectionToken)`  object:

    providers: [{ provide: APP_CONFIG, useValue: HERO_DI_CONFIG }]

Now you can inject the configuration object into any constructor that needs it, with the help of an `@[Inject](https://angular.io/api/core/Inject)()` parameter decorator.

    constructor(@Inject(APP_CONFIG) config: AppConfig) {
      this.title = config.title;
    }

## Factory providers
