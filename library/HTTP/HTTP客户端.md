## 技术点概要
URI MIME HTTP请求和响应报文 HTTP请求方法和状态码 XHR上传和下载数据 XHR流式传输 XHR定时轮询和长轮询区别与优缺点 HTTPClient CORS

# 使用 HTTP 与后端服务进行通信
大多数前端应用都要通过 `HTTP` 协议与服务器通讯，才能下载或上传数据并访问其它后端服务。 `Angular` 给应用提供了一个 `HTTP` 客户端 `API` ，也就是 `@angular/common/http` 中的 `HttpClient` 服务类。

HTTP 客户端服务提供了以下主要功能。

- 请求类型化响应对象的能力。
- 简化的错误处理。
- 各种特性的可测试性。
- 请求和响应的拦截机制。

## 服务器通讯的准备工作
要想使用 `HttpClient` ，就要先导入 `Angular` 的 `HttpClientModule` 。大多数应用都会在根模块 `AppModule` 中导入它。
```ts 
// app/app.module.ts (excerpt)

import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { HttpClientModule } from '@angular/common/http';

@NgModule({
  imports: [
    BrowserModule,
    HttpClientModule, // import HttpClientModule after BrowserModule.
  ],
  declarations: [
    AppComponent,
  ],
  bootstrap: [ AppComponent ]
})
export class AppModule {}
```
然后，你可以把 `HttpClient` 服务注入成一个应用类的依赖项，如下面的 `ConfigService` 例子所示。
```ts
// app/config/config.service.ts (excerpt)

import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

import { Observable, throwError } from 'rxjs'; // 可观察对象。
import { catchError, retry } from 'rxjs/operators'; 

@Injectable()
export class ConfigService {
  constructor(
  	private http: HttpClient // 注入
  ) { }
}
```

## 从服务器请求数据
使用 `HttpClient.get()` 方法从服务器获取数据。该异步方法会发送一个 `HTTP` 请求，并返回一个 `Observable` ，它会在收到响应时发出所请求到的数据。返回的类型取决于你调用时传入的 `observe` 和 `responseType` 参数。
`get()` 方法有两个参数。要获取的端点 `URL` ，以及一个可以用来配置请求的选项对象。
```ts
options: {
    headers?: HttpHeaders | {[header: string]: string | string[]},
    observe?: 'body' | 'events' | 'response', // *** observe 选项用于指定要返回的响应内容。 默认 'body'
    params?: HttpParams|{[param: string]: string | number | boolean | ReadonlyArray<string | number | boolean>}, // 使用 params 属性可以配置带HTTP URL 参数的请求
    reportProgress?: boolean, // reportProgress 选项可以在传输大量数据时监听进度事件。
    responseType?: 'arraybuffer'|'blob'|'json'|'text', // *** responseType 选项指定返回数据的格式。 默认 'json'
    withCredentials?: boolean,
  }
```

```ts
// assets/config.json

{
  "heroesUrl": "api/heroes",
  "textfile": "assets/textfile.txt",
  "date": "2020-01-29"
}
```
```ts
// app/config/config.service.ts (getConfig v.1)

configUrl = 'assets/config.json';

getConfig() {
  return this.http.get<Config>(this.configUrl);
}
```
```ts
// app/config/config.component.ts (showConfig v.1)

export interface Config {
  heroesUrl: string;
  textfile: string;
  date: any;
}

showConfig() {
  this.configService.getConfig()
    .subscribe((data: Config) => this.config = {
        heroesUrl: data.heroesUrl, // (data as any).heroesUrl,
        textfile:  data.textfile,
        date: data.date,
    });
}
```

***OBSERVE* 和 *RESPONSE* 的类型**

`observe` 和 `response` 选项的类型是字符串的联合类型，而不是普通的字符串。

``` ts
// this works
client.get('/foo', {responseType: 'text'})

// but this does NOT work
const options = {
  responseType: 'text',
};
client.get('/foo', options)

// this works
const options = {
  responseType: 'text' as const,
};
client.get('/foo', options);

```
### 读取完整的响应体
```ts
// xx.service.ts

getConfigResponse(): Observable<HttpResponse<Config>> {
  return this.http.get<Config>(
    this.configUrl, { 
    	observe: 'response'  // 读取完整的响应体
    });
}
```

```ts
// app/config/config.component.ts (showConfigResponse)

showConfigResponse() {
  this.configService.getConfigResponse()
    // resp is of type `HttpResponse<Config>`
    .subscribe(resp => {
      // display its headers
      const keys = resp.headers.keys();
      this.headers = keys.map(key =>
        `${key}: ${resp.headers.get(key)}`);

      // access the body directly, which is typed as `Config`.
      this.config = { ...resp.body! };
    });
}
```

### 发起 `JSONP` 请求
当服务器不支持 `CORS` 协议时，应用程序可以使用 `HttpClient` 跨域发出 `JSONP` 请求。

`Angular` 的 `JSONP` 请求会返回一个 `Observable` 。 遵循订阅可观察对象变量的模式，并在使用 `async` 管道管理结果之前，使用 `RxJS map` 操作符转换响应。

在 `Angular` 中，通过在 `NgModule` 的 `imports` 中包含 `HttpClientJsonpModule` 来使用 `JSONP` 。

在以下范例中， `searchHeroes()` 方法使用 `JSONP` 请求来查询名称包含搜索词的英雄。

```ts
/* GET heroes whose name contains search term */
searchHeroes(term: string): Observable {
  term = term.trim();

  const heroesURL = `${this.heroesURL}?${term}`;
  return this.http.jsonp(heroesUrl, 'callback').pipe( // 使用jsonp发送请求
      catchError(this.handleError('searchHeroes', [])) // then handle the error
    );
}
```
该请求将 `heroesURL` 作为第一个参数，并将回调函数名称作为第二个参数。响应被包装在回调函数中，该函数接受 `JSONP` 方法返回的可观察对象，并将它们通过管道传给错误处理程序。

### 请求非 `JSON` 数据
不是所有的 `API` 都会返回 `JSON` 数据。在下面这个例子中， `DownloaderService` 中的方法会从服务器读取文本文件，并把文件的内容记录下来，然后把这些内容使用 `Observable<string>` 的形式返回给调用者。

```ts
// app/downloader/downloader.service.ts (getTextFile)

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
```
这里的 `HttpClient.get()` 返回字符串而不是默认的 `JSON` 对象，因为它的 `responseType` 选项是 `'text'`。

`RxJS` 的 `tap` 操作符（如“窃听”中所述）使代码可以检查通过可观察对象的成功值和错误值，而不会干扰它们。

在 `DownloaderComponent` 中的 `download()` 方法通过订阅这个服务中的方法来发起一次请求。

```ts
// app/downloader/downloader.component.ts (download)

download() {
  this.downloaderService.getTextFile('assets/textfile.txt')
    .subscribe(results => this.contents = results);
}
```

## 处理请求错误
如果请求在服务器上失败了，那么 `HttpClient` 就会返回一个错误对象而不是一个成功的响应对象。

执行服务器请求的同一个服务中也应该执行错误检查、解释和解析。

发生错误时，你可以获取失败的详细信息，以便通知你的用户。在某些情况下，你也可以自动重试该请求。

### 获取错误详情
当数据访问失败时，应用会给用户提供有用的反馈。原始的错误对象作为反馈并不是特别有用。除了检测到错误已经发生之外，还需要获取错误详细信息并使用这些细节来撰写用户友好的响应。

可能会出现两种类型的错误。

- 服务器端可能会拒绝该请求，并返回状态码为 `404` 或 `500` 的 `HTTP` 响应对象。这些是错误响应。

- 客户端也可能出现问题，例如网络错误会让请求无法成功完成，或者 `RxJS` 操作符也会抛出异常。这些错误会产生 `JavaScript` 的 `ErrorEvent` 对象。 这些错误的 `status` 为 `0` ，并且其 `error` 属性包含一个 `ProgressEvent` 对象，此对象的 `type` 属性可以提供更详细的信息。

`HttpClient` 在其 `HttpErrorResponse` 中会捕获两种错误。可以检查这个响应是否存在错误。

下面的例子在之前定义的 `ConfigService` 中定义了一个错误处理程序。

```ts
// app/config/config.service.ts (handleError)

private handleError(error: HttpErrorResponse) {
  if (error.status === 0) {
    // A client-side or network error occurred. Handle it accordingly.
    console.error('An error occurred:', error.error);
  } else {
    // The backend returned an unsuccessful response code.
    // The response body may contain clues as to what went wrong.
    console.error(
      `Backend returned code ${error.status}, body was: `, error.error);
  }
  // Return an observable with a user-facing error message.
  return throwError(
    'Something bad happened; please try again later.');
}
```
该处理程序会返回一个带有用户友好的错误信息的 `RxJS ErrorObservable` 。下列代码修改了 `getConfig()` 方法，它使用一个管道把 `HttpClient.get()` 调用返回的所有 `Observable` 发送给错误处理器。
```ts
// app/config/config.service.ts (getConfig v.3 with error handler)

private handleError(error: HttpErrorResponse) {
  if (error.status === 0) {
    // A client-side or network error occurred. Handle it accordingly.
    console.error('An error occurred:', error.error);
  } else {
    // The backend returned an unsuccessful response code.
    // The response body may contain clues as to what went wrong.
    console.error(
      `Backend returned code ${error.status}, body was: `, error.error);
  }
  // Return an observable with a user-facing error message.
  return throwError(
    'Something bad happened; please try again later.');
}

getConfig() {
  return this.http.get<Config>(this.configUrl)
    .pipe(
      catchError(this.handleError)
    );
}
```

### 重试失败的请求
有时候，错误只是临时性的，只要重试就可能会自动消失。 比如，在移动端场景中可能会遇到网络中断的情况，只要重试一下就能拿到正确的结果。

`RxJS` 库提供了几个**重试操作符**。例如， `retry()` 操作符会自动重新订阅一个失败的 `Observable` 几次。重新订阅 `HttpClient` 方法会导致它重新发出 `HTTP` 请求。

下面的例子演示了如何在把一个失败的请求传给错误处理程序之前，先通过管道传给 `retry()` 操作符。

```ts
// app/config/config.service.ts (getConfig with retry)

getConfig() {
  return this.http.get<Config>(this.configUrl)
    .pipe(
      retry(3), // retry a failed request up to 3 times
      catchError(this.handleError) // then handle the error
    );
}
```

## 把数据发送到服务器
除了从服务器获取数据外， `HttpClient` 还支持其它一些 `HTTP` 方法，比如 `PUT` ， `POST` 和 `DELETE` ，你可以用它们来修改远程数据。

### 发起一个 POST 请求
```ts
/** POST: add a new hero to the database */
addHero(hero: Hero): Observable<Hero> {
  return this.http.post<Hero>(this.heroesUrl, hero, httpOptions)
    .pipe(
      catchError(this.handleError('addHero', hero))
    );
}
```
`HttpClient.post()` 方法像 `get()` 一样也有类型参数，可以用它来指出你期望服务器返回特定类型的数据。该方法需要一个资源 `URL` 和两个额外的参数：

- `body` - 要在请求体中 `POST` 过去的数据。
- `options` - 一个包含方法选项的对象，在这里，它用来指定必要的请求头。

```ts
// app/heroes/heroes.component.ts (addHero)

this.heroesService
  .addHero(newHero)
  .subscribe(hero => this.heroes.push(hero));
```

### 发起 DELETE 请求
```ts
// app/heroes/heroes.service.ts (deleteHero)

/** DELETE: delete the hero from the server */
deleteHero(id: number): Observable<unknown> {
  const url = `${this.heroesUrl}/${id}`; // DELETE api/heroes/42
  return this.http.delete(url, httpOptions)
    .pipe(
      catchError(this.handleError('deleteHero'))
    );
}
```
```ts
// app/heroes/heroes.component.ts (deleteHero)

this.heroesService
  .deleteHero(hero.id)
  .subscribe();
```
该组件不会等待删除操作的结果，所以它的 `subscribe` （订阅）中没有回调函数。不过就算你不关心结果，也仍然要订阅它。调用 `subscribe()` 方法会执行这个可观察对象，这时才会真的发起 `DELETE` 请求。

**你必须调用 subscribe()，否则什么都不会发生。仅仅调用 HeroesService.deleteHero() 是不会发起 DELETE 请求的。**

在调用方法返回的可观察对象的 subscribe() 方法之前，HttpClient 方法不会发起 HTTP 请求。这适用于 HttpClient 的所有方法。

**AsyncPipe 会自动为你订阅（以及取消订阅）。**

`HttpClient` 的所有方法返回的可观察对象都设计为冷的。 `HTTP` 请求的执行都是延期执行的，让你可以用 `tap` 和 `catchError` 这样的操作符来在实际执行 `HTTP` 请求之前，先对这个可观察对象进行扩展。

调用 `subscribe(...)` 会触发这个可观察对象的执行，并导致 `HttpClient` 组合并把 `HTTP` 请求发给服务器。

可以把这些可观察对象看做实际 `HTTP` 请求的蓝图。

### 发起 PUT 请求
```ts
/** PUT: update the hero on the server. Returns the updated hero upon success. */
updateHero(hero: Hero): Observable<Hero> {
  return this.http.put<Hero>(this.heroesUrl, hero, httpOptions)
    .pipe(
      catchError(this.handleError('updateHero', hero))
    );
}
```

### 添加和更新请求头
很多服务器都需要额外的头来执行保存操作。 例如，服务器可能需要一个授权令牌，或者需要 `Content-Type` 头来显式声明请求体的 **MIME 类型**。

#### 添加请求头
`HeroesService` 在一个 `httpOptions` 对象中定义了这样的头，它们被传给每个 `HttpClient` 的保存型方法。

```ts
// app/heroes/heroes.service.ts (httpOptions)

import { HttpHeaders } from '@angular/common/http';

const httpOptions = {
  headers: new HttpHeaders({
    'Content-Type':  'application/json',
    Authorization: 'my-auth-token'
  })
};
```

#### 更新请求头
你不能直接修改前面的选项对象中的 `HttpHeaders` 请求头，因为 `HttpHeaders` 类的实例是不可变对象。请改用 `set()` 方法，以返回当前实例应用了新更改之后的副本。

下面的例子演示了当旧令牌过期时，可以在发起下一个请求之前更新授权头。

```ts
httpOptions.headers = httpOptions.headers.set('Authorization', 'my-new-auth-token');
```

## 配置 HTTP URL 参数
使用 `HttpParams` 类和 `params` 选项在你的 `HttpRequest` 中添加 `URL` 查询字符串。

下面的例子中， `searchHeroes()` 方法用于查询名字中包含搜索词的英雄。

首先导入 `HttpParams` 类。
```ts
import {HttpParams} from "@angular/common/http";

/* GET heroes whose name contains search term */
searchHeroes(term: string): Observable<Hero[]> {
  term = term.trim();

  // Add safe, URL encoded search parameter if there is a search term
  const options = term ?
   { params: new HttpParams().set('name', term) } : {};

  return this.http.get<Hero[]>(this.heroesUrl, options)
    .pipe(
      catchError(this.handleError<Hero[]>('searchHeroes', []))
    );
}
```
如果有搜索词，代码会用进行过 `URL` 编码的搜索参数来构造一个 `options` 对象。例如，如果搜索词是 `"cat"`，那么 `GET` 请求的 `URL` 就是 `api/heroes?name=cat`。
你也可以使用 `fromString` 变量从查询字符串中直接创建 `HTTP` 参数：

```ts
const params = new HttpParams({fromString: 'name=foo'});
```

## 拦截请求和响应
借助拦截机制，你可以声明一些拦截器，它们可以检查并转换从应用中发给服务器的 `HTTP` 请求。这些拦截器还可以在返回应用的途中检查和转换来自服务器的响应。多个拦截器构成了请求/响应处理器的双向链表。

拦截器可以用一种常规的、标准的方式对每一次 `HTTP` 的请求/响应任务执行从认证到记日志等很多种**隐式**任务。

如果没有拦截机制，那么开发人员将不得不对每次 `HttpClient` 调用**显式**实现这些任务。

### 编写拦截器
要实现拦截器，就要实现一个实现了 `HttpInterceptor` 接口中的 `intercept()` 方法的类。

这里是一个什么也不做的**空白**拦截器，它只会不做任何修改的传递这个请求。

```ts
// app/http-interceptors/noop-interceptor.ts

import { Injectable } from '@angular/core';
import {
  HttpEvent, HttpInterceptor, HttpHandler, HttpRequest
} from '@angular/common/http';

import { Observable } from 'rxjs';

/** Pass untouched request through to the next request handler. */
@Injectable()
export class NoopInterceptor implements HttpInterceptor {

  intercept(req: HttpRequest<any>, next: HttpHandler):
    Observable<HttpEvent<any>> {
    return next.handle(req);
  }
}
```
`intercept` 方法会把请求转换成一个最终返回 `HTTP` 响应体的 `Observable` 。 在这个场景中，每个拦截器都完全能自己处理这个请求。

大多数拦截器拦截都会在传入时检查请求，然后把（可能被修改过的）请求转发给 `next` 对象的 `handle()` 方法，而 `next` 对象实现了 `HttpHandler` 接口。

```ts
export abstract class HttpHandler {
  abstract handle(req: HttpRequest<any>): Observable<HttpEvent<any>>;
}
```
像 `intercept()` 一样，`handle()` 方法也会把 `HTTP` 请求转换成 `HttpEvents` 组成的 `Observable` ，它最终包含的是来自服务器的响应。 `intercept()` 函数可以检查这个可观察对象，并在把它返回给调用者之前修改它。

这个无操作的拦截器，会使用原始的请求调用 `next.handle()` ，并返回它返回的可观察对象，而不做任何后续处理。

### next 对象
`next` 对象表示拦截器链表中的下一个拦截器。 这个链表中的最后一个 `next` 对象就是 `HttpClient` 的后端处理器（`backend handler`），它会把请求发给服务器，并接收服务器的响应。

大多数的拦截器都会调用 `next.handle()` ，以便这个请求流能走到下一个拦截器，并最终传给后端处理器。 拦截器也可以不调用 `next.handle()` ，使这个链路短路，并返回一个带有人工构造出来的服务器响应的 自己的 `Observable` 。

这是一种常见的中间件模式，在像 `Express.js` 这样的框架中也会找到它。

### 提供这个拦截器
这个 `NoopInterceptor` 就是一个由 `Angular` 依赖注入 (`DI`) 系统管理的服务。像其它服务一样，你也必须先提供这个拦截器类，应用才能使用它。

由于拦截器是 `HttpClient` 服务的（可选）依赖，所以你必须在提供 `HttpClient` 的同一个（或其各级父注入器）注入器中提供这些拦截器。那些在 `DI` 创建完 `HttpClient` **之后**再提供的拦截器将会被忽略。

由于在 `AppModule` 中导入了 `HttpClientModule` ，导致本应用在其根注入器中提供了 `HttpClient` 。所以你也同样要在 `AppModule` 中提供这些拦截器。

在从 `@angular/common/http` 中导入了 `HTTP_INTERCEPTORS` 注入令牌之后，编写如下的 `NoopInterceptor` 提供者注册语句：

```ts
{ provide: HTTP_INTERCEPTORS, useClass: NoopInterceptor, multi: true },
```

注意 `multi: true` 选项。 这个必须的选项会告诉 `Angular` `HTTP_INTERCEPTORS` 是一个**多重提供者**的令牌，表示它会注入一个多值的数组，而不是单一的值。

你**也可以**直接把这个提供者添加到 `AppModule` 中的提供者数组中，不过那样会非常啰嗦。况且，你将来还会用这种方式创建更多的拦截器并提供它们。 你还要特别注意提供这些拦截器的顺序。

认真考虑创建一个封装桶（ `barrel` ）文件，用于把所有拦截器都收集起来，一起提供给 `httpInterceptorProviders` 数组，可以先从这个 `NoopInterceptor` 开始。

```ts
// app/http-interceptors/index.ts

/* "Barrel" of Http Interceptors */
import { HTTP_INTERCEPTORS } from '@angular/common/http';

import { NoopInterceptor } from './noop-interceptor';

/** Http interceptor providers in outside-in order */
export const httpInterceptorProviders = [
  { provide: HTTP_INTERCEPTORS, useClass: NoopInterceptor, multi: true },
];
```

然后导入它，并把它加到 `AppModule` 的 `providers array` 中，就像这样：

```ts
// app/app.module.ts (interceptor providers)
providers: [
  httpInterceptorProviders
],
```

当你再创建新的拦截器时，就同样把它们添加到 `httpInterceptorProviders` 数组中，而不用再修改 `AppModule` 。

### 拦截器的顺序
`Angular` 会按你提供拦截器的顺序应用它们。比如，考虑一个场景：你想处理 `HTTP` 请求的身份验证并记录它们，然后再将它们发送到服务器。要完成此任务，你可以提供 `AuthInterceptor` 服务，然后提供 `LoggingInterceptor` 服务。发出的请求将从 `AuthInterceptor` 到 `LoggingInterceptor` 。这些请求的响应则沿相反的方向流动，从 `LoggingInterceptor` 回到 `AuthInterceptor` 。

以下是该过程的直观表示：

[]()

### 处理拦截器事件
大多数 `HttpClient` 方法都会返回 `HttpResponse<any>` 型的可观察对象。 `HttpResponse` 类本身就是一个事件，它的类型是 `HttpEventType.Response` 。
但是，单个 `HTTP` 请求可以生成其它类型的多个事件，包括报告上传和下载进度的事件。 `HttpInterceptor.intercept()` 和 `HttpHandler.handle()` 会返回 `HttpEvent<any>` 型的可观察对象。

很多拦截器只关心发出的请求，而对 `next.handle()` 返回的事件流不会做任何修改。但是，有些拦截器需要检查并修改 `next.handle()` 的响应。上述做法就可以在流中看到所有这些事件。

虽然拦截器有能力改变请求和响应，但 `HttpRequest` 和 `HttpResponse` 实例的属性却是只读（ `readonly` ）的， 因此让它们**基本上是不可变的**。

有充足的理由把它们做成不可变对象：应用可能会重试发送很多次请求之后才能成功，这就意味着这个拦截器链表可能会多次重复处理同一个请求。 如果拦截器可以修改原始的请求对象，那么重试阶段的操作就会从修改过的请求开始，而不是原始请求。 而这种不可变性，可以确保这些拦截器在每次重试时看到的都是同样的原始请求。

**你的拦截器应该在没有任何修改的情况下返回每一个事件，除非它有令人信服的理由去做。**

`TypeScript` 会阻止你设置 `HttpRequest` 的只读属性。

```ts
// Typescript disallows the following assignment because req.url is readonly
req.url = req.url.replace('http://', 'https://');
```

如果你必须修改一个请求，先把它克隆一份，修改这个克隆体后再把它传给 `next.handle()` 。你可以在一步中克隆并修改此请求，例子如下。

```ts
// app/http-interceptors/ensure-https-interceptor.ts (excerpt)

// ※clone request and replace 'http://' with 'https://' at the same time
const secureReq = req.clone({
  url: req.url.replace('http://', 'https://')
});

// send the cloned, "secure" request to the next handler.
return next.handle(secureReq);
```

这个 `clone()` 方法的哈希型参数允许你在复制出克隆体的同时改变该请求的某些特定属性。

#### 修改请求体
`readonly` 这种赋值保护，无法防范深修改（修改子对象的属性），也不能防范你修改请求体对象中的属性。

```ts
req.body.name = req.body.name.trim(); // bad idea!
```

如果必须修改请求体，请执行以下步骤。

1. 复制请求体并在副本中进行修改。
2. 使用 `clone()` 方法克隆这个请求对象。
3. 用修改过的副本替换被克隆的请求体。

```ts
// copy the body and trim whitespace from the name property
const newBody = { ...body, name: body.name.trim() };
// clone request and set its body
const newReq = req.clone({ body: newBody });
// send the cloned request to the next handler.
return next.handle(newReq);
```

#### 克隆时清除请求体
有时，你需要清除请求体而不是替换它。为此，请将克隆后的请求体设置为 `null` 。

> 提示：
> 如果你把克隆后的请求体设为 `undefined` ，那么 `Angular` 会认为你想让请求体保持原样。

```ts
newReq = req.clone({ … }); // body not mentioned => preserve original body
newReq = req.clone({ body: undefined }); // preserve original body
newReq = req.clone({ body: null }); // clear the body
```

## HTTP 拦截器用例
### 设置默认请求头
```ts
// app/http-interceptors/auth-interceptor.ts

import { AuthService } from '../auth.service';

@Injectable()
export class AuthInterceptor implements HttpInterceptor {

  constructor(private auth: AuthService) {}

  intercept(req: HttpRequest<any>, next: HttpHandler) {
    // Get the auth token from the service.
    const authToken = this.auth.getAuthorizationToken();

    // Clone the request and replace the original headers with
    // cloned headers, updated with the authorization.
    const authReq = req.clone({
      headers: req.headers.set('Authorization', authToken)
    });

    // 这种在克隆请求的同时设置新请求头的操作太常见了，因此它还有一个快捷方式 `setHeaders`：
    // Clone the request and set the new header in one step.
    const authReq = req.clone({ 
      setHeaders: { Authorization: authToken } 
    });

    // send cloned request with header to the next handler.
    return next.handle(authReq);
  }
}
```

这种可以修改头的拦截器可以用于很多不同的操作，比如：

- 认证 / 授权
- 控制缓存行为。比如 `If-Modified-Since`
- `XSRF` 防护

### 记录请求与响应对
因为拦截器可以**同时**处理请求和响应，所以它们也可以对整个 `HTTP` 操作执行计时和记录日志等任务。

考虑下面这个 `LoggingInterceptor` ，它捕获请求的发起时间、响应的接收时间，并使用注入的 `MessageService` 来发送总共花费的时间。
```ts
// app/http-interceptors/logging-interceptor.ts

import { finalize, tap } from 'rxjs/operators';
import { MessageService } from '../message.service';

@Injectable()
export class LoggingInterceptor implements HttpInterceptor {
  constructor(private messenger: MessageService) {}

  intercept(req: HttpRequest<any>, next: HttpHandler) {
    const started = Date.now();
    let ok: string;

    // extend server response observable with logging
    return next.handle(req)
      .pipe(
        tap({
          // Succeeds when there is a response; ignore other events
          next: (event) => (ok = event instanceof HttpResponse ? 'succeeded' : ''),
          // Operation failed; error is an HttpErrorResponse
          error: (error) => (ok = 'failed')
        }),
        // Log when response observable either completes or errors
        finalize(() => {
          const elapsed = Date.now() - started;
          const msg = `${req.method} "${req.urlWithParams}"
             ${ok} in ${elapsed} ms.`;
          this.messenger.add(msg);
        })
      );
  }
}
```

`RxJS` 的 `tap` 操作符会捕获请求成功了还是失败了。 `RxJS` 的 `finalize` 操作符无论在响应成功还是失败时都会调用（这是必须的），然后把结果汇报给 `MessageService` 。

在这个可观察对象的流中，无论是 `tap` 还是 `finalize` 接触过的值，都会照常发送给调用者。

### 自定义 `JSON` 解析
拦截器可用来以自定义实现替换内置的 `JSON` 解析。

以下示例中的 `CustomJsonInterceptor` 演示了如何实现此目的。如果截获的请求期望一个 '`json`' 响应，则将 `responseType` 更改为 '`text`' 以禁用内置的 `JSON` 解析。然后，通过注入的 `JsonParser` 解析响应。

```ts
// app/http-interceptors/custom-json-interceptor.ts

// The JsonParser class acts as a base class for custom parsers and as the DI token.
@Injectable()
export abstract class JsonParser {
  abstract parse(text: string): any;
}

@Injectable()
export class CustomJsonInterceptor implements HttpInterceptor {
  constructor(private jsonParser: JsonParser) {}

  intercept(httpRequest: HttpRequest<any>, next: HttpHandler) {
    if (httpRequest.responseType === 'json') {
      // If the expected response type is JSON then handle it here.
      return this.handleJsonResponse(httpRequest, next);
    } else {
      return next.handle(httpRequest);
    }
  }

  private handleJsonResponse(httpRequest: HttpRequest<any>, next: HttpHandler) {
    // Override the responseType to disable the default JSON parsing.
    httpRequest = httpRequest.clone({responseType: 'text'});
    // Handle the response using the custom parser.
    return next.handle(httpRequest).pipe(map(event => this.parseJsonResponse(event)));
  }

  private parseJsonResponse(event: HttpEvent<any>) {
    if (event instanceof HttpResponse && typeof event.body === 'string') {
      return event.clone({body: this.jsonParser.parse(event.body)});
    } else {
      return event;
    }
  }
}
```

然后，你可以实现自己的自定义 `JsonParser` 。这是一个具有特殊日期接收器的自定义 `JsonParser` 。

```ts
// app/http-interceptors/custom-json-interceptor.ts

@Injectable()
export class CustomJsonParser implements JsonParser {
  parse(text: string): any {
    return JSON.parse(text, dateReviver);
  }
}

function dateReviver(key: string, value: any) {
  /* . . . */
}
```

你提供 `CustomParser` 以及 `CustomJsonInterceptor` 。

```ts
// app/http-interceptors/index.ts

{ provide: HTTP_INTERCEPTORS, useClass: CustomJsonInterceptor, multi: true },
{ provide: JsonParser, useClass: CustomJsonParser },
```

### 用拦截器实现缓存

拦截器还可以自行处理这些请求，而不用转发给 `next.handle()` 。

比如，你可能会想缓存某些请求和响应，以便提升性能。你可以把这种缓存操作委托给某个拦截器，而不破坏你现有的各个数据服务。

下例中的 `CachingInterceptor` 演示了这种方法。

```ts
// app/http-interceptors/caching-interceptor.ts)

@Injectable()
export class CachingInterceptor implements HttpInterceptor {
  constructor(private cache: RequestCache) {}

  intercept(req: HttpRequest<any>, next: HttpHandler) {
    // continue if not cacheable.
    if (!isCacheable(req)) { return next.handle(req); }

    const cachedResponse = this.cache.get(req);
    return cachedResponse ?
      of(cachedResponse) : sendRequest(req, next, this.cache);
  }
}
```

- `isCacheable()` 函数用于决定该请求是否允许缓存。在这个例子中，只有发到 `npm` 包搜索 `API` 的 GET 请求才是可以缓存的。
- 如果该请求是不可缓存的，该拦截器会把该请求转发给链表中的下一个处理器
- 如果可缓存的请求在缓存中找到了，该拦截器就会通过 `of()` 函数返回一个已缓存的响应体的可观察对象，然后绕过 `next` 处理器（以及所有其它下游拦截器）
- 如果可缓存的请求不在缓存中，代码会调用 `sendRequest()` 。这个函数会把请求转发给 `next.handle()` ，它会最终调用服务器并返回来自服务器的响应对象。

```ts
/**
 * Get server response observable by sending request to `next()`.
 * Will add the response to the cache on the way out.
 */
function sendRequest(
  req: HttpRequest<any>,
  next: HttpHandler,
  cache: RequestCache): Observable<HttpEvent<any>> {
  return next.handle(req).pipe(
    tap(event => {
      // There may be other events besides the response.
      if (event instanceof HttpResponse) {
        cache.put(req, event); // Update the cache.
      }
    })
  );
}
```

> 注意 `sendRequest()` 是如何在返回应用程序的过程中拦截响应的。该方法通过 `tap()` 操作符来管理响应对象，该操作符的回调函数会把该响应对象添加到缓存中。
> 然后，原始的响应会通过这些拦截器链，原封不动的回到服务器的调用者那里。
> 数据服务，比如 `PackageSearchService` ，并不知道它们收到的某些 `HttpClient` 请求实际上是从缓存的请求中返回来的。

### 用拦截器来请求多个值
`HttpClient.get()` 方法通常会返回一个可观察对象，它会发出一个值（数据或错误）。拦截器可以把它改成一个可以发出多个值的可观察对象。

修改后的 `CachingInterceptor` 版本可以返回一个立即发出所缓存响应的可观察对象，然后把请求发送到 `NPM` 的 `Web API` ，然后把修改过的搜索结果重新发出一次。

```ts
// cache-then-refresh
if (req.headers.get('x-refresh')) {
  const results$ = sendRequest(req, next, this.cache);
  return cachedResponse ?
    results$.pipe( startWith(cachedResponse) ) :
    results$;
}
// cache-or-fetch
return cachedResponse ?
  of(cachedResponse) : sendRequest(req, next, this.cache);
```

> `cache-then-refresh` 选项是由一个自定义的 `x-refresh` 请求头触发的。
> `PackageSearchComponent` 中的一个检查框会切换 `withRefresh` 标识，它是 `PackageSearchService.search()` 的参数之一。 `search()` 方法创建了自定义的 `x-refresh` 头，并在调用 `HttpClient.get()`前把它添加到请求里。

修改后的 `CachingInterceptor` 会发起一个服务器请求，而不管有没有缓存的值。 就像 前面 的 `sendRequest()` 方法一样进行订阅。 在订阅 `results$` 可观察对象时，就会发起这个请求。
如果没有缓存值，拦截器直接返回 `results$` 。
如果有缓存的值，这些代码就会把缓存的响应加入到 `result$` 的管道中，使用重组后的可观察对象进行处理，并发出两次。先立即发出一次缓存的响应体，然后发出来自服务器的响应。订阅者将会看到一个包含这两个响应的序列。

## 跟踪和显示请求进度
应用程序有时会传输大量数据，而这些传输可能要花很长时间。文件上传就是典型的例子。你可以通过提供关于此类传输的进度反馈，为用户提供更好的体验。

要想发出一个带有进度事件的请求，你可以创建一个 `HttpRequest` 实例，并把 `reportProgress` 选项设置为 `true` 来启用对进度事件的跟踪。

```ts
// app/uploader/uploader.service.ts (upload request)

const req = new HttpRequest('POST', '/upload/file', file, {
  reportProgress: true
});
```

> 提示：
> 每个进度事件都会触发变更检测，所以只有当需要在 UI 上报告进度时，你才应该开启它们。
> 当 `HttpClient.request()` 和 `HTTP` 方法一起使用时，可以用 `observe: 'events'` 来查看所有事件，包括传输的进度。

接下来，把这个请求对象传给 `HttpClient.request()` 方法，该方法返回一个 `HttpEvents` 的 `Observable` （与 拦截器 部分处理过的事件相同）。

```ts
// app/uploader/uploader.service.ts (upload body)

// The `HttpClient.request` API produces a raw event stream
// which includes start (sent), progress, and response events.
return this.http.request(req).pipe(
  map(event => this.getEventMessage(event, file)),
  tap(message => this.showProgress(message)),
  last(), // return last (completed) message to caller
  catchError(this.handleError(file))
);
```

`getEventMessage` 方法解释了事件流中每种类型的 `HttpEvent` 。

```ts
// app/uploader/uploader.service.ts (getEventMessage)

/** Return distinct message for sent, upload progress, & response events */
private getEventMessage(event: HttpEvent<any>, file: File) {
  switch (event.type) {
    case HttpEventType.Sent:
      return `Uploading file "${file.name}" of size ${file.size}.`;

    case HttpEventType.UploadProgress:
      // Compute and show the % done:
      const percentDone = event.total ? Math.round(100 * event.loaded / event.total) : 0;
      return `File "${file.name}" is ${percentDone}% uploaded.`;

    case HttpEventType.Response:
      return `File "${file.name}" was completely uploaded!`;

    default:
      return `File "${file.name}" surprising upload event: ${event.type}.`;
  }
}
```

> 本指南中的范例应用中没有用来接受上传文件的服务器。 `app/http-interceptors/upload-interceptor.ts` 的 `UploadInterceptor` 通过返回一个模拟这些事件的可观察对象来拦截和短路上传请求。


## 通过防抖来优化与服务器的交互
如果你需要发一个 `HTTP` 请求来响应用户的输入，那么每次按键就发送一个请求的效率显然不高。最好等用户停止输入后再发送请求。这种技术叫做防抖。
考虑下面这个模板，它让用户输入一个搜索词来按名字查找 `npm` 包。当用户在搜索框中输入名字时， `PackageSearchComponent` 就会把这个根据名字搜索包的请求发给 `npm web API` 。

```html
<!-- app/package-search/package-search.component.html (search) -->

<input type="text" (keyup)="search(getValue($event))" id="name" placeholder="Search"/>

<ul>
  <li *ngFor="let package of packages$ | async">
    <b>{{package.name}} v.{{package.version}}</b> -
    <i>{{package.description}}</i>
  </li>
</ul>
```

在这里， `keyup` 事件绑定会将每个按键都发送到组件的 `search()` 方法。
```ts
// $event.target 的类型在模板中只是 EventTarget，而在 getValue() 方法中，目标会转换成 HTMLInputElement 类型，以允许对它的 value 属性进行类型安全的访问。

getValue(event: Event): string {
  return (event.target as HTMLInputElement).value;
}
```
这里， `keyup` 事件绑定会把每次按键都发送给组件的 `search()` 方法。下面的代码片段使用 `RxJS` 的操作符为这个输入实现了防抖。
```ts
// app/package-search/package-search.component.ts (excerpt)

constructor(private searchService: PackageSearchService) { }

withRefresh = false;
packages$!: Observable<NpmPackageInfo[]>;
private searchText$ = new Subject<string>();

search(packageName: string) {
  this.searchText$.next(packageName);
}

ngOnInit() {
  this.packages$ = this.searchText$.pipe(
    debounceTime(500),
    distinctUntilChanged(),
    switchMap(packageName =>
      this.searchService.search(packageName, this.withRefresh))
  );
}


```

`searchText$` 是来自用户的搜索框值的序列。它被定义为 `RxJS` `Subject` 类型，这意味着它是一个多播 `Observable` ，它还可以通过调用 `next(value)` 来自行发出值，就像在 `search()` 方法中一样。

除了把每个 `searchText` 的值都直接转发给 `PackageSearchService` 之外， `ngOnInit()`中的代码还通过下列三个操作符对这些搜索值进行**管道**处理，以便只有当它是一个新值并且用户已经停止输入时，要搜索的值才会抵达该服务。

|RXJS|操作符详情|
|-|-|
|debounceTime(500)|等待用户停止输入（本例中为 1/2 秒）。|
|distinctUntilChanged()|等待搜索文本发生变化。|
|switchMap()|将搜索请求发送到服务。|

这些代码把 `packages$` 设置成了使用搜索结果组合出的 `Observable` 对象。模板中使用 `AsyncPipe` 订阅了 `packages$` ，一旦搜索结果的值发回来了，就显示这些搜索结果。

### 使用 `switchMap()` 操作符
`switchMap()` 操作符接受一个返回 `Observable` 的函数型参数。在这个例子中， `PackageSearchService.search` 像其它数据服务方法那样返回一个 `Observable` 。如果先前的搜索请求仍在**进行中**（如网络连接不良），它将取消该请求并发送新的请求。

> 注意：
> `switchMap()` 会按照原始的请求顺序返回这些服务的响应，而不用关心服务器实际上是以乱序返回的它们。
> 如果你觉得将来会复用这些防抖逻辑，可以把它移到单独的工具函数中，或者移到 `PackageSearchService` 中。

## 安全： `XSRF` 防护
跨站请求伪造 ( `XSRF` 或 `CSRF` )是一个攻击技术，它能让攻击者假冒一个已认证的用户在你的网站上执行未知的操作。 

`HttpClient` 支持一种通用的机制来防范 `XSRF` 攻击。当执行 `HTTP` 请求时，一个拦截器会从 `cookie` 中读取 `XSRF` 令牌（默认名字为 `XSRF-TOKEN` ），并且把它设置为一个 `HTTP` 头 `X-XSRF-TOKEN` ，由于只有运行在你自己的域名下的代码才能读取这个 `cookie` ，因此后端可以确认这个 `HTTP` 请求真的来自你的客户端应用，而不是攻击者。

默认情况下，拦截器会在所有的修改型请求中（比如 `POST` 等）把这个请求头发送给使用相对 `URL` 的请求。但不会在 `GET/HEAD` 请求中发送，也不会发送给使用绝对 `URL` 的请求。

要获得这种优点，你的服务器需要在页面加载或首个 `GET` 请求中把一个名叫 `XSRF-TOKEN` 的令牌写入可被 `JavaScript` 读到的会话 `cookie` 中。而在后续的请求中，服务器可以验证这个 `cookie` 是否与 `HTTP` 头 `X-XSRF-TOKEN` 的值一致，以确保只有运行在你自己域名下的代码才能发起这个请求。这个令牌必须对每个用户都是唯一的，并且必须能被服务器验证，因此不能由客户端自己生成令牌。把这个令牌设置为你的站点认证信息并且加了盐（salt）的摘要，以提升安全性。

为了防止多个 `Angular` 应用共享同一个域名或子域时出现冲突，要给每个应用分配一个唯一的 `cookie` 名称。

> `HttpClient` 支持的只是 `XSRF` 防护方案的客户端这一半。 你的后端服务必须配置为给页面设置 `cookie` ，并且要验证请求头，以确保全都是合法的请求。如果不这么做，就会导致 `Angular` 的默认防护措施失效。
 
### 配置自定义 `cookie/header` 名称

如果你的后端服务中对 `XSRF` 令牌的 `cookie` 或 头使用了不一样的名字，就要使用 `HttpClientXsrfModule.withConfig()` 来覆盖掉默认值。
```ts
imports: [
  HttpClientModule,
  HttpClientXsrfModule.withOptions({
    cookieName: 'My-Xsrf-Cookie',
    headerName: 'My-Xsrf-Header',
  }),
],
```

## 测试 `HTTP` 请求
如同所有的外部依赖一样，你必须把 `HTTP` 后端也 `Mock` 掉，以便你的测试可以模拟这种与后端的互动。 `@angular/common/http/testing` 库能让这种 `Mock` 工作变得直截了当。

`Angular` 的 `HTTP` 测试库是专为其中的测试模式而设计的。在这种模式下，会首先在应用中执行代码并发起请求。然后，这个测试会期待发起或未发起过某个请求，并针对这些请求进行断言，最终对每个所预期的请求进行刷新（ `flush` ）来对这些请求提供响应。

最终，测试可能会验证这个应用不曾发起过非预期的请求。

### 搭建测试环境
要开始测试那些通过 HttpClient 发起的请求，就要导入 `HttpClientTestingModule` 模块，并把它加到你的 `TestBed` 设置里去，代码如下。

```ts
// app/testing/http-client.spec.ts (imports)

// Http testing module and mocking controller
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';

// Other imports
import { TestBed } from '@angular/core/testing';
import { HttpClient, HttpErrorResponse } from '@angular/common/http';

// 然后把 `HTTPClientTestingModule` 添加到 `TestBed` 中，并继续设置**被测服务**。

describe('HttpClient testing', () => {
  let httpClient: HttpClient;
  let httpTestingController: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [ HttpClientTestingModule ]
    });

    // Inject the http service and test controller for each test
    httpClient = TestBed.inject(HttpClient);
    httpTestingController = TestBed.inject(HttpTestingController);
  });
  /// Tests begin ///
}); 
```

现在，在测试中发起的这些请求会发给这些测试用的后端（`testing backend`），而不是标准的后端。

这种设置还会调用 `TestBed.inject()` ，来获取注入的 `HttpClient` 服务和模拟对象的控制器 `HttpTestingController` ，以便在测试期间引用它们。

### 期待并回复请求
现在，你就可以编写测试，等待 `GET` 请求并给出模拟响应。

```ts
// app/testing/http-client.spec.ts (HttpClient.get)

it('can test HttpClient.get', () => {
  const testData: Data = {name: 'Test Data'};

  // Make an HTTP GET request
  httpClient.get<Data>(testUrl)
    .subscribe(data =>
      // When observable resolves, result should match test data
      expect(data).toEqual(testData)
    );

  // The following `expectOne()` will match the request's URL.
  // If no requests or multiple requests matched that URL
  // `expectOne()` would throw.
  const req = httpTestingController.expectOne('/data');

  // Assert that the request is a GET.
  expect(req.request.method).toEqual('GET');

  // Respond with mock data, causing Observable to resolve.
  // Subscribe callback asserts that correct data was returned.
  req.flush(testData);

  // Finally, assert that there are no outstanding requests.
  httpTestingController.verify();
});
```

最后一步，验证没有发起过预期之外的请求，足够通用，因此你可以把它移到 `afterEach()` 中：

```ts
afterEach(() => {
  // After every test, assert that there are no more pending requests.
  httpTestingController.verify();
});
```

#### 自定义对请求的预期
如果仅根据 `URL` 匹配还不够，你还可以自行实现匹配函数。比如，你可以验证外发的请求是否带有某个认证头：

```ts
// Expect one request with an authorization header
const req = httpTestingController.expectOne(
  request => request.headers.has('Authorization')
);
```

像前面的 `expectOne()` 测试一样，如果零或两个以上的请求满足了这个断言，它就会抛出异常。

#### 处理一个以上的请求
如果你需要在测试中对重复的请求进行响应，可以使用 `match() API` 来代替 `expectOne()` ，它的参数不变，但会返回一个与这些请求相匹配的数组。一旦返回，这些请求就会从将来要匹配的列表中移除，你要自己验证和刷新（ `flush` ）它。

```ts
// get all pending requests that match the given URL
const requests = httpTestingController.match(testUrl);
expect(requests.length).toEqual(3);

// Respond to each request with different results
requests[0].flush([]);
requests[1].flush([testData[0]]);
requests[2].flush(testData);
```

#### 测试对错误的预期
你还要测试应用对于 `HTTP` 请求失败时的防护。
调用 `request.flush()` 并传入一个错误信息，如下所示。

```ts
it('can test for 404 error', () => {
  const emsg = 'deliberate 404 error';

  httpClient.get<Data[]>(testUrl).subscribe({
    next: () => fail('should have failed with the 404 error'),
    error: (error: HttpErrorResponse) => {
      expect(error.status).withContext('status').toEqual(404);
      expect(error.error).withContext('message').toEqual(emsg);
    },
  });

  const req = httpTestingController.expectOne(testUrl);

  // Respond with mock error
  req.flush(emsg, { status: 404, statusText: 'Not Found' });
});
```
另外，还可以用 `ProgressEvent` 来调用 `request.error()`

```ts
it('can test for network error', done => {
  // Create mock ProgressEvent with type `error`, raised when something goes wrong
  // at network level. e.g. Connection timeout, DNS error, offline, etc.
  const mockError = new ProgressEvent('error');

  httpClient.get<Data[]>(testUrl).subscribe({
    next: () => fail('should have failed with the network error'),
    error: (error: HttpErrorResponse) => {
      expect(error.error).toBe(mockError);
      done();
    },
  });

  const req = httpTestingController.expectOne(testUrl);

  // Respond with mock error
  req.error(mockError);
});
```

## 将元数据传递给拦截器
许多拦截器都需要进行配置或从配置中受益。考虑一个重试失败请求的拦截器。默认情况下，拦截器可能会重试请求三次，但是对于特别容易出错或敏感的请求，你可能要改写这个重试次数。
`HttpClient` 请求包含一个**上下文**，该上下文可以携带有关请求的元数据。该上下文可供拦截器读取或修改，尽管发送请求时它并不会传输到后端服务器。这允许应用程序或其他拦截器使用配置参数来标记这些请求，比如重试请求的次数。

### 创建上下文令牌
`HttpContextToken` 用于在上下文中存储和检索值。你可以用 `new` 运算符创建上下文令牌，如以下例所示：
```ts
// creating a context token

export const RETRY_COUNT = new HttpContextToken(() => 3);
```
`HttpContextToken` 创建期间传递的 `lambda` 函数 `() => 3` 有两个用途：

- 它允许 `TypeScript` 推断此令牌的类型： `HttpContextToken<number>` 。这个请求上下文是类型安全的 —— 从请求上下文中读取令牌将返回适当类型的值。
- 它会设置令牌的默认值。如果尚未为此令牌设置其他值，那么这就是请求上下文返回的值。使用默认值可以避免检查是否已设置了特定值。

### 在发起请求时设置上下文值
发出请求时，你可以提供一个 `HttpContext` 实例，在该实例中你已经设置了一些上下文值。

```ts
// setting context values

this.httpClient
    .get('/data/feed', {
      context: new HttpContext().set(RETRY_COUNT, 5),
    })
    .subscribe(results => {/* ... */});
```

### 在拦截器中读取上下文值
`HttpContext.get()` 在给定请求的上下文中读取令牌的值。如果尚未显式设置令牌的值，则 `Angular` 将返回令牌中指定的默认值。
```ts
// reading context values in an interceptor

import {retry} from 'rxjs';

export class RetryInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const retryCount = req.context.get(RETRY_COUNT);

    return next.handle(req).pipe(
        // Retry the request a configurable number of times.
        retry(retryCount),
    );
  }
}
```

### 上下文是可变的（ `Mutable` ）
与 `HttpRequest` 实例的大多数其他方面不同，请求上下文是可变的，并且在请求的其他不可变转换过程中仍然存在。这允许拦截器通过此上下文协调来操作。比如， `RetryInterceptor` 示例可以使用第二个上下文令牌来跟踪在执行给定请求期间发生过多少错误：

```ts
// coordinating operations through the context

import {retry, tap} from 'rxjs/operators';
export const RETRY_COUNT = new HttpContextToken(() => 3);
export const ERROR_COUNT = new HttpContextToken(() => 0);

export class RetryInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const retryCount = req.context.get(RETRY_COUNT);

    return next.handle(req).pipe(
        tap({
              // An error has occurred, so increment this request's ERROR_COUNT.
             error: () => req.context.set(ERROR_COUNT, req.context.get(ERROR_COUNT) + 1)
            }),
        // Retry the request a configurable number of times.
        retry(retryCount),
    );
  }
}
```