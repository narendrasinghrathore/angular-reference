# Angular mini reference guide

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
