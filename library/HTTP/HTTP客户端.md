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