# [HttpClient](https://angular.io/guide/http)

Most front-end applications communicate with backend services over the HTTP protocol. Modern browsers support two different APIs for making HTTP requests: the  `XMLHttpRequest`  interface and the  `fetch()`  API.

The  `[HttpClient](https://angular.io/api/common/http/HttpClient)`  in  `@angular/common/[http](https://angular.io/api/common/http)`  offers a simplified client HTTP API for Angular applications that rests on the  **`XMLHttpRequest`**  interface exposed by browsers. Additional benefits of  `[HttpClient](https://angular.io/api/common/http/HttpClient)`  include testability features, typed request and response objects, request and response interception,  `Observable`  apis, and streamlined error handling.

### Requesting a typed response
You can structure your  `[HttpClient](https://angular.io/api/common/http/HttpClient)`  request to declare the type of the response object, to make consuming the output easier and more obvious. Specifying the response type acts as a type assertion during the compile time.

To specify the response object type, first define an interface with the required properties. (Use an interface rather than a class; a response cannot be automatically converted to an instance of a class.)

    export interface Config {
      heroesUrl: string;
      textfile: string;
    }

Next, specify that interface as the `[HttpClient.get()](https://angular.io/api/common/http/HttpClient#get)` call's type parameter in the service.

    getConfig() {
      // now returns an Observable of Config
      return this.http.get<Config>(this.configUrl);
    }

When you pass an interface as a type parameter to the `[HttpClient.get()](https://angular.io/api/common/http/HttpClient#get)` method, use the RxJS `map` operator to transform the response data as needed by the UI. You can then pass the transformed data to the [async pipe](https://angular.io/api/common/AsyncPipe).

> Specifying the response type is a declaration to TypeScript that it
> should expect your response to be of the given type. This is a
> build-time check and doesn't guarantee that the server will actually
> respond with an object of this type. It is up to the server to ensure
> that the type specified by the server API is returned.


### Reading the full response

The response body doesn't return all the data you may need. Sometimes servers return special headers or status codes to indicate certain conditions that are important to the application workflow.

Tell  `[HttpClient](https://angular.io/api/common/http/HttpClient)`  that you want the full response with the  `observe`  option:

    getConfigResponse(): Observable<HttpResponse<Config>> {
      return this.http.get<Config>(
        this.configUrl, { observe: 'response' });
    }

Now  `[HttpClient.get()](https://angular.io/api/common/http/HttpClient#get)`  returns an  `Observable`  of type  `[HttpResponse](https://angular.io/api/common/http/HttpResponse)`  rather than just the JSON data.

The component's  `showConfigResponse()`  method displays the response headers as well as the configuration:

    showConfigResponse() {
      this.configService.getConfigResponse()
        // resp is of type `HttpResponse<Config>`
        .subscribe(resp => {
          // display its headers
          const keys = resp.headers.keys();
          this.headers = keys.map(key =>
            `${key}: ${resp.headers.get(key)}`);
    
          // access the body directly, which is typed as `Config`.
          this.config = { ... resp.body };
        });
    }


### Making a JSONP request

Apps can use the the  `[HttpClient](https://angular.io/api/common/http/HttpClient)`  to make  [JSONP](https://en.wikipedia.org/wiki/JSONP)  requests across domains when the server doesn't support  [CORS protocol](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).

Angular JSONP requests return an  `Observable`. Follow the pattern for subscribing to observables and use the RxJS  `map`  operator to transform the response before using the  [async pipe](https://angular.io/api/common/AsyncPipe)  to manage the results.

In Angular, use JSONP by including  `[HttpClientJsonpModule](https://angular.io/api/common/http/HttpClientJsonpModule)`  in the  `[NgModule](https://angular.io/api/core/NgModule)`  imports. In the following example, the  `searchHeroes()`  method uses a JSONP request to query for heroes whose names contain the search term.


    /* GET heroes whose name contains search term */
    searchHeroes(term: string): Observable {
      term = term.trim();
    
      let heroesURL = `${this.heroesURL}?${term}`;
      return this.http.jsonp(heroesUrl, 'callback').pipe(
          catchError(this.handleError('searchHeroes', []) // then handle the error
        );
    };


This request passes the `heroesURL` as the first parameter and the callback function name as the second parameter. The response is wrapped in the callback function, which takes the observables returned by the JSONP method and pipes them through to the error handler.

### Requesting non-JSON data
Not all APIs return JSON data. In this next example, a `DownloaderService` method reads a text file from the server and logs the file contents, before returning those contents to the caller as an `Observable<string>`.

    getTextFile(filename: string) {
      // The Observable returned by get() is of type Observable<string>
      // because a text response was specified.
      // There's no need to pass a <string> type parameter to get().
      return this.http.get(filename, {responseType: 'text'})
        .pipe(
          tap( // Log the result or error
            data => this.log(filename, data),
            error => this.logError(filename, error)
          )
        );
    }

`[HttpClient.get()](https://angular.io/api/common/http/HttpClient#get)`  returns a string rather than the default JSON because of the  `[responseType](https://angular.io/api/common/http/HttpRequest#responseType)`  option.

The RxJS  `tap`  operator (as in "wiretap") lets the code inspect both success and error values passing through the observable without disturbing them.


## Error handling
What happens if the request fails on the server, or if a poor network connection prevents it from even reaching the server?  `[HttpClient](https://angular.io/api/common/http/HttpClient)`  will return an  _error_  object instead of a successful response.

You  _could_  handle in the component by adding a second callback to the  `.subscribe()`:

    showConfig() {
      this.configService.getConfig()
        .subscribe(
          (data: Config) => this.config = { ...data }, // success path
          error => this.error = error // error path
        );
    }


It's certainly a good idea to give the user some kind of feedback when data access fails. But displaying the raw error object returned by `[HttpClient](https://angular.io/api/common/http/HttpClient)` is far from the best way to do it.

### [Getting error details](https://angular.io/guide/http#getting-error-details)

Detecting that an error occurred is one thing. Interpreting that error and composing a user-friendly response is a bit more involved.

Two types of errors can occur. The server backend might reject the request, returning an HTTP response with a status code such as 404 or 500. These are error  _responses_.

Or something could go wrong on the client-side such as a network error that prevents the request from completing successfully or an exception thrown in an RxJS operator. These errors produce JavaScript  `ErrorEvent`  objects.

The  `[HttpClient](https://angular.io/api/common/http/HttpClient)`  captures both kinds of errors in its  `[HttpErrorResponse](https://angular.io/api/common/http/HttpErrorResponse)`  and you can inspect that response to figure out what really happened.

Error inspection, interpretation, and resolution is something you want to do in the  _service_, not in the  _component_.

You might first devise an error handler like this one:

    private handleError(error: HttpErrorResponse) {
      if (error.error instanceof ErrorEvent) {
        // A client-side or network error occurred. Handle it accordingly.
        console.error('An error occurred:', error.error.message);
      } else {
        // The backend returned an unsuccessful response code.
        // The response body may contain clues as to what went wrong,
        console.error(
          `Backend returned code ${error.status}, ` +
          `body was: ${error.error}`);
      }
      // return an observable with a user-facing error message
      return throwError(
        'Something bad happened; please try again later.');
    };


Notice that this handler returns an RxJS  [`ErrorObservable`](https://angular.io/guide/http#rxjs)  with a user-friendly error message. Consumers of the service expect service methods to return an  `Observable`  of some kind, even a "bad" one.

Now you take the  `Observables`  returned by the  `[HttpClient](https://angular.io/api/common/http/HttpClient)`  methods and  _pipe them through_  to the error handler.


    getConfig() {
      return this.http.get<Config>(this.configUrl)
        .pipe(
          catchError(this.handleError)
        );
    }

### Retrying
Sometimes the error is transient and will go away automatically if you try again. For example, network interruptions are common in mobile scenarios, and trying again may produce a successful result.

The  [RxJS library](https://angular.io/guide/http#rxjs)  offers several  _retry_  operators that are worth exploring. The simplest is called  `retry()`  and it automatically re-subscribes to a failed  `Observable`  a specified number of times.  _Re-subscribing_  to the result of an  `[HttpClient](https://angular.io/api/common/http/HttpClient)`  method call has the effect of reissuing the HTTP request.

_Pipe_  it onto the  `[HttpClient](https://angular.io/api/common/http/HttpClient)`  method result just before the error handler.

    getConfig() {
      return this.http.get<Config>(this.configUrl)
        .pipe(
          retry(3), // retry a failed request up to 3 times
          catchError(this.handleError) // then handle the error
        );
    }


## [HTTP headers](https://angular.io/guide/http#http-headers)

Many servers require extra headers for save operations. For example, they may require a "Content-Type" header to explicitly declare the MIME type of the request body; or the server may require an authorization token.

### Adding headers
The `HeroesService` defines such headers in an `httpOptions` object that will be passed to every `[HttpClient](https://angular.io/api/common/http/HttpClient)` save method.

    import { HttpHeaders } from '@angular/common/http';
    
    const httpOptions = {
      headers: new HttpHeaders({
        'Content-Type':  'application/json',
        'Authorization': 'my-auth-token'
      })
    };

### Updating headers
You can't directly modify the existing headers within the previous options object because instances of the  `[HttpHeaders](https://angular.io/api/common/http/HttpHeaders)`  class are immutable.

Use the  `set()`  method instead, to return a clone of the current instance with the new changes applied.

Here's how you might update the authorization header (after the old token expired) before making the next request.


    httpOptions.headers =
      httpOptions.headers.set('Authorization', 'my-new-auth-token');


## Sending data to the server
In addition to fetching data from the server, `[HttpClient](https://angular.io/api/common/http/HttpClient)` supports mutating requests, that is, sending data to the server with other HTTP methods such as PUT, POST, and DELETE.

### POST Request

    /** POST: add a new hero to the database */
    addHero (hero: Hero): Observable<Hero> {
      return this.http.post<Hero>(this.heroesUrl, hero, httpOptions)
        .pipe(
          catchError(this.handleError('addHero', hero))
        );
    }
It takes two more parameters:

1.  `hero`  - the data to POST in the body of the request.
2.  `httpOptions`  - the method options which, in this case,  [specify required headers](https://angular.io/guide/http#adding-headers).

### Making a DELETE request

This application deletes a hero with the `HttpClient.delete` method by passing the hero's id in the request URL.

    /** DELETE: delete the hero from the server */
    deleteHero (id: number): Observable<{}> {
      const url = `${this.heroesUrl}/${id}`; // DELETE api/heroes/42
      return this.http.delete(url, httpOptions)
        .pipe(
          catchError(this.handleError('deleteHero'))
        );
    }

**Always  _subscribe_!**

An  `[HttpClient](https://angular.io/api/common/http/HttpClient)`  method does not begin its HTTP request until you call  `subscribe()`  on the observable returned by that method. This is true for  _all_  `[HttpClient](https://angular.io/api/common/http/HttpClient)`  _methods_.


The [`AsyncPipe`](https://angular.io/api/common/AsyncPipe) subscribes (and unsubscribes) for you automatically.

All observables returned from  `[HttpClient](https://angular.io/api/common/http/HttpClient)`  methods are  _cold_  by design. Execution of the HTTP request is  _deferred_, allowing you to extend the observable with additional operations such as  `tap`  and  `catchError`  before anything actually happens.

Calling  `subscribe(...)`  triggers execution of the observable and causes  `[HttpClient](https://angular.io/api/common/http/HttpClient)`  to compose and send the HTTP request to the server.

You can think of these observables as  _blueprints_  for actual HTTP requests.

In fact, each `subscribe()` initiates a separate, independent execution of the observable. Subscribing twice results in two HTTP requests.

    const req = http.get<Heroes>('/api/heroes');
    // 0 requests made - .subscribe() not called.
    req.subscribe();
    // 1 request made.
    req.subscribe();
    // 2 requests made.



### Making a PUT request

An app will send a PUT request to completely replace a resource with updated data. The following  `HeroesService`  example is just like the POST example.

    /** PUT: update the hero on the server. Returns the updated hero upon success. */
    updateHero (hero: Hero): Observable<Hero> {
      return this.http.put<Hero>(this.heroesUrl, hero, httpOptions)
        .pipe(
          catchError(this.handleError('updateHero', hero))
        );
    }


### HTTP interceptors

_HTTP Interception_  is a major feature of  `@angular/common/[http](https://angular.io/api/common/http)`. With interception, you declare  _interceptors_  that inspect and transform HTTP requests from your application to the server. The same interceptors may also inspect and transform the server's responses on their way back to the application. Multiple interceptors form a  _forward-and-backward_  chain of request/response handlers.

Interceptors can perform a variety of  _implicit_  tasks, from authentication to logging, in a routine, standard way, for every HTTP request/response.

Without interception, developers would have to implement these tasks  _explicitly_  for each  `[HttpClient](https://angular.io/api/common/http/HttpClient)`  method call.
