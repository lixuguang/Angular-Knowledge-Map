要处理从一个视图到下一个视图的导航，请使用 `Angular Router`。 `Router` 会通过将浏览器 `URL` 解释为更改视图的操作指令来启用导航。

## 常见路由任务
生成一个支持路由的应用
```shell
ng new routing-app --routing --defaults // 
```

`--routing` 是为支持路由的参数

### 为路由导入组件
```ts
// AppRoutingModule (excerpt) ./app-routing.module
import { FirstComponent } from './first/first.component';
import { SecondComponent } from './second/second.component';
```

### 定义一个基本路由
创建路由有三个基本的构建块。
把 `AppRoutingModule` 导入 `AppModule` 并把它添加到 `imports` 数组中。
```ts
// Default CLI AppModule with routing
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { AppRoutingModule } from './app-routing.module'; // CLI imports AppRoutingModule
import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule // CLI adds AppRoutingModule to the AppModule's imports array
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```
```ts
// CLI application routing module 
// ./app-routing.module

import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router'; // CLI imports router

const routes: Routes = [
  { path: 'first-component', component: FirstComponent },
  { path: 'second-component', component: SecondComponent },
];

// configures NgModule imports and exports
@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

```html
<h1>Angular Router App</h1>
<!-- This nav gives you links to click, which tells the router which route to use (defined in the routes constant in  AppRoutingModule) -->
<nav>
  <ul>
    <li><a routerLink="/first-component" routerLinkActive="active">First Component</a></li> <!-- 路由 -->
    <li><a routerLink="/second-component" routerLinkActive="active">Second Component</a></li><!-- 路由 -->
  </ul>
</nav>
<!-- The routed views render in the <router-outlet>-->
<router-outlet></router-outlet>
```

#### 路由顺序
路由的顺序很重要，因为 `Router` 在匹配路由时使用**“先到先得”**策略，所以应该在**不那么具体的路由前面放置更具体的路由**。
首先列出静态路径的路由，
然后是一个与默认路由匹配的空路径路由。
通配符路由是最后一个，因为它匹配每一个 `URL` ，只有当其它路由都没有匹配时， `Router` 才会选择它。


### 获取路由信息
通常，当用户导航你的应用时，你会希望把信息从一个组件传递到另一个组件。
例如，考虑一个显示杂货商品购物清单的应用。列表中的每一项都有一个唯一的 `id` 。
要想编辑某个项目，用户需要单击“编辑”按钮，打开一个 `EditGroceryItem` 组件。
你希望该组件得到该商品的 `id` ，以便它能向用户显示正确的信息。

可以用一个路由把这种类型的信息传给你的应用组件。要做到这一点，你可以使用 `ActivatedRoute` 接口。

要从路由中获取信息：
```ts
import { Router, ActivatedRoute, ParamMap } from '@angular/router'; // 把 ActivatedRoute 和 ParamMap 导入你的组件。

constructor(
  private route: ActivatedRoute, // 通过把 ActivatedRoute 的一个实例添加到你的应用的构造函数中来注入它
) {}

ngOnInit() {
  this.route.queryParams.subscribe(params => { // 更新 ngOnInit() 方法来访问这个 ActivatedRoute 并跟踪 name 参数
    this.name = params['name'];
  });
}
```

### 设置通配符路由
当用户试图导航到那些不存在的应用部件时，在正常的应用中应该能得到很好的处理。要在应用中添加此功能，需要设置通配符路由。当所请求的 `URL` 与任何路由器路径都不匹配时， `Angular` 路由器就会选择这个路由。
要设置通配符路由，请在 `routes` 定义中添加以下代码。
```ts
{ path: '**', component:  } // 这两个星号 ** 告诉 Angular，这个 routes 定义是通配符路由。
```
对于 `component` 属性，你可以使用应用中的任何组件。常见的选择包括应用专属的 `PageNotFoundComponent` ，你可以定义它来向用户展示 `404` 页面，或者跳转到应用的主组件。
通配符路由是**最后一个路由**，因为它匹配所有的 `URL` 。关于路由顺序的更多详细信息，请参阅路由顺序。

```ts
const routes: Routes = [
  { 
  	path: 'first-component', 
  	component: FirstComponent,
  	children:[{ // 嵌套路由
	    path: 'child-a', // child route path
	    component: ChildAComponent, // child route component that the router renders
    }]
  },
  { path: 'second-component', component: SecondComponent },
  { path: '',   redirectTo: '/first-component', pathMatch: 'full' }, // redirect to `first-component` 设置重定向
  { path: '**', component: PageNotFoundComponent },  // Wildcard route for a 404 page 设置通配符路由, 显示 404 页面
];
```
```html
<h2>First Component</h2>

<nav>
  <ul>
    <li><a routerLink="child-a">Child A</a></li>
    <li><a routerLink="../second-component">Child B</a></li><!-- 使用相对路径 -->
  </ul>
</nav>

<router-outlet></router-outlet>
```
除了 `../`(查找上一个级别)，还可以使用 `./` 或者不带前导斜杠来指定当前级别。

#### 指定相对路由
要指定相对路由，请使用 `NavigationExtras` 中的 `relativeTo` 属性。在组件类中，从 `@angular/router` 导入 `NavigationExtras` 。
然后在导航方法中使用 `relativeTo` 参数。在链接参数数组（它包含 `items` ）之后添加一个对象，把该对象的 `relativeTo` 属性设置为当前的 `ActivatedRoute` ，也就是 `this.route` 。

```ts
goToItems() { // goToItems() 方法会把目标 URI 解释为相对于当前路由的，并导航到 items 路由。
  this.router.navigate(['items'], { relativeTo: this.route });
}
```

### 访问查询参数和片段
有时，应用中的某个特性需要访问路由的部件，比如查询参数或片段（`fragment`）。本教程的这个阶段使用了一个“英雄之旅”中的列表视图，你可以在其中点击一个英雄来查看详情。路由器使用 `id` 来显示正确的英雄的详情。
首先，在要导航的组件中导入以下成员。
```ts
// Component 1 (excerpt)
import { ActivatedRoute } from '@angular/router';
import { Observable } from 'rxjs';
import { switchMap } from 'rxjs/operators';

constructor(
	private route: ActivatedRoute
) {}

heroes$: Observable; // 可观察对象
selectedId: number; // 保存英雄的 id 号
heroes = HEROES; // 英雄列表

ngOnInit() {
  this.heroes$ = this.route.paramMap.pipe(
    switchMap(params => {
      this.selectedId = Number(params.get('id'));
      return this.service.getHeroes();
    })
  );
}
```

```ts
// Component 2 (excerpt)
import { Router, ActivatedRoute, ParamMap } from '@angular/router';
import { Observable } from 'rxjs';

constructor(
	private route: ActivatedRoute,
	private router: Router  
) {}

hero$: Observable; // 可观察对象

ngOnInit() {
	const heroId = this.route.snapshot.paramMap.get('id');
	this.hero$ = this.service.getHero(heroId);
}

gotoItems(hero: Hero) {
	const heroId = hero ? hero.id : null;
	// Pass along the hero id if available
	// so that the HeroList component can select that item.
	this.router.navigate(['/heroes', { id: heroId }]); // 导航到Component 1
}
```

### 惰性加载
你可以配置路由定义来实现惰性加载模块，这意味着 `Angular` 只会在需要时才加载这些模块，而不是在应用启动时就加载全部。 另外，你可以在后台预加载一些应用部件来改善用户体验。


#### 惰性加载和预加载
要惰性加载 `Angular` 模块，请在 `AppRoutingModule` `routes` 中使用 `loadChildren` 代替 `component` 进行配置，代码如下。
```ts
// src/app/app-routing.module.ts

const routes: Routes = [
  {
    path: 'customers',
    loadChildren: () => import('./customers/customers.module').then(m => m.CustomersModule)
  },
  {
    path: 'orders',
    loadChildren: () => import('./orders/orders.module').then(m => m.OrdersModule)
  },
  {
    path: '',
    redirectTo: '',
    pathMatch: 'full'
  }
];
```
```ts
// src/app/customers/customers.module.ts

import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { CustomersRoutingModule } from './customers-routing.module';
import { CustomersComponent } from './customers.component';

@NgModule({
  imports: [
    CommonModule,
    CustomersRoutingModule
  ],
  declarations: [CustomersComponent]
})
export class CustomersModule { }
```

```ts
// src/app/customers/customers-routing.module.ts

import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';

import { CustomersComponent } from './customers.component';


const routes: Routes = [
  {
    path: '',
    component: CustomersComponent
  }
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class CustomersRoutingModule { }
```
还要确保从 `AppModule` 中移除了 `CustomersModule` 。 

##### forRoot() 与 forChild()
你可能已经注意到了，CLI 会把 `RouterModule.forRoot(routes)` 添加到 `AppRoutingModule` 的 `imports` 数组中。 
这会让 `Angular` 知道 `AppRoutingModule` 是一个路由模块，而 `forRoot() `表示这是一个根路由模块。 它会配置你传入的所有路由、让你能访问路由器指令并注册 `Router` 。 
`forRoot()` 在应用中**只应该使用一次**，也就是这个 `AppRoutingModule` 中。

CLI 还会把 `RouterModule.forChild(routes) `添加到各个特性模块中。这种方式下 `Angular` 就会知道这个路由列表只负责提供额外的路由并且其设计意图是作为特性模块使用。
你可以在**多个模块**中使用 `forChild()`。

`forRoot()` 方法为路由器管理全局性的注入器配置。 
`forChild()` 方法中没有注入器配置，只有像 `RouterOutlet` 和 `RouterLink` 这样的指令。 

##### 预加载
预加载通过在后台加载部分应用来改进用户体验。你可以预加载模块或组件数据。
要启用所有惰性加载模块的预加载，请从 `Angular` 的 `router` 导入 `PreloadAllModules` 令牌。
```ts
// AppRoutingModule (excerpt)
import { PreloadAllModules } from '@angular/router';

RouterModule.forRoot(
  appRoutes,
  {
    preloadingStrategy: PreloadAllModules
  }
)
```

###### 预加载组件数据
```ts
// Resolver service (excerpt)

import { Resolve } from '@angular/router';

/* An interface that represents your data model */
export interface Crisis {
  id: number;
  name: string;
}

export class CrisisDetailResolverService implements Resolve {
  resolve(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable {
    // 这里做一些先行处理的逻辑
  }
}
```
把这个解析器导入此模块的路由模块。
```ts
// Feature module's routing module (excerpt)

import { CrisisDetailResolverService } from './crisis-detail-resolver.service';

{
  path: '/your-path',
  component: YourComponent,
  resolve: { // 在组件的 route 配置中添加一个 resolve 对象, 用来传递先行处理的数据。
    crisis: CrisisDetailResolverService
  }
}
```
在此组件的构造函数中，注入一个 `ActivatedRoute` 实例，它可以表示当前路由。
```ts
// YourComponent

import { ActivatedRoute } from '@angular/router';

@Component({ ... })
class YourComponent {
  constructor(private route: ActivatedRoute) {}

  ngOnInit() {
    this.route.data
      .subscribe(data => {
        const crisis: Crisis = data.crisis; // 使用注入进来的 ActivatedRoute 类实例来访问与指定路由关联的 data 值。
        // ...
      });
  }
}
```

### 防止未经授权的访问
使用路由守卫来防止用户未经授权就导航到应用的某些部分。 `Angular` 中提供了以下路由守卫：
- `CanActivate` --- 类可以实现的接口，用于确定是否可以激活路由。如果所有守卫都返回了 true，那么导航将继续。如果任何守卫返回 false，则导航将被取消。 --- 检查路由访问
- `CanActivateChild` --- 类可以实现的接口，用于确定是否可以激活**子**路由。如果所有守卫都返回了 true，那么导航将继续。如果任何守卫返回 false，则导航将被取消。 --- 检查子路由访问
- `CanDeactivate` --- 类可以实现的接口，用于确定是否可以离开某个路由。如果所有守卫都返回了 true，那么导航将继续。如果任何守卫返回 false，则导航将被取消。 --- 在放弃未保存的更改之前请求许可
- `Resolve` --- 可以实现为数据提供者的类的接口。数据提供者类可与路由器一起使用，以在导航期间解析数据。 --- 预先获取路由数据
- `CanLoad` --- 类可以实现的接口，用于确定是否可以加载子路由。如果所有守卫都返回了 true，那么导航将继续。如果任何守卫返回 false，则导航将被取消。 --- 在加载功能模块的文件之前检查

- 用 `CanActivate` 来处理导航**到**某路由的情况。
- 用 `CanActivateChild` 来处理导航**到**某子路由的情况。
- 用 `CanDeactivate` 来处理从当前路由**离开**的情况.
- 用 `Resolve` 在路由激活**之前**获取路由数据。
- 用 `CanLoad` 来处理**异步**导航到某特性模块的情况。

```ts
// Component (excerpt)
export class YourGuard implements CanActivate {
  canActivate(
    next: ActivatedRouteSnapshot,
    state: RouterStateSnapshot): boolean {
      // your  logic goes here
  }
}
```

```ts
// Routing module (excerpt)
{
  path: '/your-path',
  component: YourComponent,
  canActivate: [YourGuard],
}
```

#### 无组件路由
要想使用路由守卫，可以考虑使用无组件路由，因为这对于保护子路由很方便。
你可以在同级组件之间共享参数。例如，假设两个同级组件应该彼此相邻，并且它们两个都需要 ID 参数。你可以使用不在顶层指定组件的路由来完成此操作。
```ts
[{
   path: 'parent/:id',
   // component: YourComponent, // 这里没有component属性，在path上给子路由共用了id参数
   children: [
     { path: 'a', component: MainChild },
     { path: 'b', component: AuxChild, outlet: 'aux' }
   ]
}]
```

### 链接参数数组
链接参数数组保存路由导航时所需的成分：
- 指向目标组件的那个路由的路径（`path`）
- 必备路由参数和可选路由参数，它们将进入该路由的 `URL`

可以把 RouterLink 指令绑定到一个数组，就像这样：

```html
// src/app/app.component.ts (h-anchor)

template: `
  <h1 class="title">Angular Router</h1>
  <nav>
    <a [routerLink]="['/crisis-center']">Crisis Center</a> <!-- /crisis-center -->
    <a [routerLink]="['/crisis-center/1', { foo: 'foo' }]">Dragon Crisis</a> <!-- /crisis-center/1;foo='foo' -->
    <a [routerLink]="['/crisis-center/2']">Shark Crisis</a> <!-- /crisis-center/2 -->
  </nav>
  <router-outlet></router-outlet>
`
```
#### 相对链接路径
第一段名称可以用 `/`、`./` 或 `../` 开头。

- 如果第一个片段用 `/` 开头，则路由器会从应用的**根路由**开始查找。
- 如果第一个片段用 `./` 开头或者没有用斜杠开头，路由器就会从**当前激活路由**开始查找。
- 如果第一段以 `../` 开头，则路由器将去往路由树中的**上一层**。

#### 设置和处理查询参数和片段
```html
<a 
	[routerLink]="['/user/bob']" 
	[queryParams]="{debug: true}" 
	fragment="education" 
>
	<!-- => /user/bob?debug=true#education -->
  link to user component
</a>

<a 
	[routerLink]="['/user/bob']" 
	[queryParams]="{otherKey: 2}" 
	queryParamsHandling="merge" 
	queryParamsHandling="preserve" 
>
	<!-- => “merge” 选项会将新的查询参数附加到当前 URL 的参数中： /user/bob?debug=true#education&otherKey=2 -->
	<!-- => “preserve” 选项将放弃所有新的查询参数： /user/bob?debug=true#education -->
  link to user component
</a>

```

#### 保留导航历史
你可以提供要持久到浏览器的 `History.state` 属性中的 `state` 值。例如：
```html
<a [routerLink]="['/user/bob']" [state]="{tracingId: 123}">
  link to user component
</a>
```

使用 `Router#getCurrentNavigation` 来检索保存的导航状态值。例如，要在 `NavigationStart` 事件中捕获 `tracingId`

```ts
// Get NavigationStart events
router.events.pipe(filter(e => e instanceof NavigationStart)).subscribe(e => {
  const navigation = router.getCurrentNavigation();
  tracingService.trace({id: navigation.extras.state.tracingId});
});
```

### `LocationStrategy` 和 浏览器的网址样式

> 浏览器的网址样式
> 现代 `HTML5` 浏览器支持 `history.pushState` API， 这是一项可以改变浏览器的当前地址和历史，却又不会触发服务端页面请求的技术。 路由器可以合成出一个“自然的” `URL` ，它看起来和那些需要进行页面加载的 `URL` 没什么区别。

- `LocationStrategy`
	- `PathLocationStrategy` - 默认的策略，支持“ `HTML5 pushState` ”风格。
	- `HashLocationStrategy` - 支持“ `hash URL` ”风格。

```ts
@NgModule({
  imports: [RouterModule.forRoot(ROUTES),{
  	useHash: true //  修改位置策略（LocationStrategy），用 URL 片段（#）代替 history API。
  }]
})
class MyNgModule {}
```

#### 选择路由策略
你必须在开发项目的早期就选择一种路由策略，因为一旦该应用进入了生产阶段，你网站的访问者就会使用并依赖应用的这些 URL 引用。
几乎所有的 Angular 项目都会使用默认的 HTML5 风格。它生成的 URL 更易于被用户理解，它也为将来做服务端渲染预留了空间。
在服务器端渲染指定的页面，是一项可以在该应用首次加载时大幅提升响应速度的技术。那些原本需要十秒甚至更长时间加载的应用，可以预先在服务端渲染好，并在少于一秒的时间内完整渲染在用户的设备上。
只有当应用的 URL 看起来像是标准的 Web URL，中间没有 hash（#）时，这个选项才能生效。

#### `<base href>`
你必须在应用的 `index.html` 中添加一个 `<base href>` 元素才能让 `pushState` 路由正常工作。 浏览器要用 `<base href>` 的值为引用 `CSS` 、`脚本` 和 `图片文件` 时使用的相对 `URL` 添加前缀。
```html
// src/index.html (base-href)

<base href="/">
```

#### RouterLinkActive
通过添加 routerLinkActive 指令，可以通知你的应用把一个特定的 CSS 类应用到当前的活动路由中。
跟踪元素上的链接路由当前是否处于活动状态，并允许你指定一个或多个 CSS 类，以便在链接路由处于活动状态时添加到该元素。

#### RouterOutlet
一个占位符，Angular 会根据当前的路由器状态动态填充它。

#### UrlMatcher
一个用于匹配路由和 `URL` 的函数。 当 `path` 和 `pathMatch` 的组合不足以表达时，可以为 `Route.matcher` 实现一个自定义的 `URL` 匹配器。
```ts
export function htmlFiles(url: UrlSegment[]) {
  return url.length === 1 && url[0].path.endsWith('.html') ? ({consumed: url}) : null;
}

export const routes = [{ matcher: htmlFiles, component: AnyComponent }];
```

#### 路由器术语
- `Router` 为活动 URL 显示应用中的组件。 管理从一个组件到另一个的导航。
- `RouterModule` 一个单独的 NgModule，它提供了一些必要的服务提供者和一些用于在应用视图间导航的指令。
- `Routes` 定义一个路由数组，每一个条目都会把一个 URL 路径映射到组件。
- `Route` 定义路由器如何基于一个 URL 模式导航到某个组件。 大部分路由都由一个路径和一个组件类组成。
- `RouterOutlet` 该指令 (`<router-outlet>`) 用于指出路由器应该把视图显示在哪里。
- `RouterLink` 用于将可点击的 `HTML` 元素绑定到某个路由的指令。单击带有 `routerLink` 指令且绑定到字符串或链接参数数组的元素，将触发导航。
- `RouterLinkActive` 该指令会在元素上或元素内包含的相关 `routerLink` 处于活动/非活动状态时，从 `HTML` 元素上添加/移除类。
- `ActivatedRoute` 一个提供给每个路由组件的服务，其中包含当前路由专属的信息，例如路由参数、静态数据、解析数据、全局查询参数和全局片段。
- `RouterState` 路由器的当前状态，包括一棵当前激活路由的树以及遍历这棵路由树的便捷方法。
- `链接参数数组` 一个由路由器将其解释为路由指南的数组。你可以将该数组绑定到 `RouterLink` 或将该数组作为参数传给 `Router.navigate` 方法。
- `路由组件` 一个带有 `RouterOutlet` 的 `Angular` 组件，可基于路由器的导航来显示视图。

#### 路由器事件
- `RoutesRecognized` 当路由器解析了 URL，而且路由已经识别完毕时触发的事件。
- `NavigationStart` 导航开始时触发的事件。
- `NavigationEnd` 当导航成功结束时触发的事件。
- `NavigationCancel` 当导航被取消时触发的事件。 这可能在导航期间某个路由守卫返回了 false 或返回了 UrlTree 以进行重定向时发生。
- `NavigationError` 当导航由于非预期的错误而失败时触发的事件。
- `RouteConfigLoadStart` 在 Router 惰性加载路由配置之前触发的事件。
- `RouteConfigLoadEnd` 在某个路由已经惰性加载完毕时触发的事件。
- `GuardsCheckStart` 当路由器开始进入路由守卫阶段时触发的事件。
- `GuardsCheckEnd` 当路由器成功结束了路由守卫阶段时触发的事件。
- `ChildActivationStart` 当路由器开始激活某路由的子路由时触发的事件。
- `ChildActivationEnd` 当路由器成功激活某路由的子路由时触发的事件。
- `ActivationStart` 当路由器开始激活某个路由时触发的事件。
- `ActivationEnd` 当路由器成功停止激活某个路由时触发的事件。
- `ResolveStart` 当路由器开始路由解析阶段时触发的事件。
- `ResolveEnd` 当路由器的路由解析阶段成功完成时触发的事件。
- `Scroll` 用来表示滚动的事件。