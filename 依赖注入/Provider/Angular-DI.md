## Angular 中的依赖注入
依赖项是指某个类执行其功能所需的服务或对象。依赖项注入（`DI`）是一种设计模式，在这种设计模式中，类会从外部源请求依赖项而不是创建它们。
`Angular` 的 `DI` 框架会在实例化某个类时为其提供依赖。可以使用 `Angular DI` 来提高应用程序的灵活性和模块化程度。

## 创建可注入服务
要想在 `src/app/heroes` 目录下生成一个新的 `HeroService` 类，请使用下列 `Angular CLI` 命令。
```shell
ng generate service heroes/hero
```

下列命令会创建默认的 `HeroService` 。
```ts
// src/app/heroes/hero.service.ts (CLI-generated)

import { Injectable } from '@angular/core';

@Injectable({ // 标记性元数据，表示一个类可以由 Injector 进行创建。
  providedIn: 'root', // 'root'：在大多数应用程序中是指应用程序级注入器。
})
export class HeroService {
  constructor() { }
}
```

`@Injectable()` 装饰器会指定 `Angular` 可以在 `DI` 体系中使用此类。元数据 `providedIn: 'root'` 表示 `HeroService` 在整个应用程序中都是可见的。

接下来，要获取英雄的模拟数据，请添加一个 `getHeroes()` 方法，该方法会从 `mock.heroes.ts` 中返回英雄。

```ts
// src/app/heroes/hero.service.ts

import { Injectable } from '@angular/core';
import { HEROES } from './mock-heroes';

@Injectable({ // 标记性元数据，表示一个类可以由 Injector 进行创建。
  providedIn: 'root', // 'root'：在大多数应用程序中是指应用程序级注入器。
})
export class HeroService {
  constructor() { }
  getHeroes() { 
  	return HEROES; 
  }
}
```

## 注入服务
注入某些服务会使它们对组件可见。
要将依赖项注入组件的 `constructor()` 中，请提供具有此依赖项类型的构造函数参数。下面的示例在 `HeroListComponent` 的构造函数中指定了 `HeroService` 。 `heroService` 的类型是 `HeroService` 。

```html
// src/app/heroes/hero-list.component (constructor signature)

constructor(heroService: HeroService)
```

## 在其他服务中使用这些服务
```ts
// src/app/logger.service.ts

import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class Logger {
  logs: string[] = []; // capture logs for testing

  log(message: string) {
    this.logs.push(message);
    console.log(message);
  }
}
```

```ts
// src/app/heroes/hero.service.ts

import { Injectable } from '@angular/core';
import { HEROES } from './mock-heroes';
import { Logger } from '../logger.service'; // 引入依赖的logger服务

@Injectable({
  providedIn: 'root',
})
export class HeroService {

  constructor(
  	private logger: Logger // 注入依赖
  ) {  }

  getHeroes() {
    this.logger.log('Getting heroes ...'); // 调用logger服务
    return HEROES;
  }
}
```

```html
// src/app/heroes/hero-list.component (constructor signature)

constructor(heroService: HeroService)
```

## 依赖提供者
通过配置提供者，你可以把服务提供给那些需要它们的应用部件。
依赖提供者会使用 `DI` 令牌来配置注入器，注入器会用它来提供这个依赖值的具体的、运行时版本。

### 指定提供者令牌
如果你把服务类指定为提供者令牌，那么注入器的默认行为是用 `new` 来实例化那个类。
在下面这个例子中， `Logger` 类提供了 `Logger` 的实例。
```ts
providers: [Logger]
```
不过，你也可以用一个替代提供者来配置注入器，这样就可以指定另一些同样能提供日志功能的对象。
可以使用服务类来配置注入器，也可以提供一个替代类、一个对象或一个工厂函数。

### 依赖注入令牌
当使用提供者配置注入器时，会将该提供者与依赖项注入令牌（或叫 `DI` 令牌）关联起来。注入器允许 `Angular` 创建任何内部依赖项的映射。 `DI` 令牌会充当该映射的键名。
依赖项值是一个实例，而这个类的类型用作查找键。
在这里，注入器使用 `HeroService` 类型作为令牌来查找 `heroService` 。
```ts
// src/app/injector.component.ts
heroService: HeroService;
```

当你使用 `HeroService` 类的类型来定义构造函数参数时， `Angular` 会注入与这个 `HeroService` 类令牌相关联的服务：
```ts
// src/app/heroes/hero-list.component.ts
constructor(heroService: HeroService)
```
尽管许多依赖项的值是通过类提供的，但扩展的 `provide` 对象使你可以将不同种类的提供者与 `DI` 令牌相关联。

### 定义提供者
类提供者的语法实际上是一种简写形式，它会扩展成一个由 `Provider` 接口定义的提供者配置对象。 
下面的代码片段展示了 `providers` 中给出的类会如何扩展成完整的提供者配置对象。
```ts
providers: [Logger]
```
Angular 把这个 providers 值扩展为一个完整的提供者对象，如下所示。
```ts
[{ 
  provide: Logger, // provide 属性存有令牌，它作为一个 key，在定位依赖值和配置注入器时使用。
  useClass: Logger // 第二个属性是一个提供者定义对象，它告诉注入器要如何创建依赖值。 
}]
```
扩展的提供者配置是一个具有两个属性的对象字面量：
1. provide 属性存有令牌，它作为一个 key，在定位依赖值和配置注入器时使用。
2. 第二个属性是一个提供者定义对象，它告诉注入器要如何创建依赖值。 提供者定义对象中的 `key` 可以是 `useClass` —— 就像这个例子中一样。 也可以是 `useExisting` 、 `useValue` 或 `useFactory` 。 每一个 `key` 都用于提供一种不同类型的依赖，我们稍后会讨论。

### 指定替代性的类提供者 useClass
不同的类可以提供相同的服务。例如，以下代码告诉注入器，当组件使用 `Logger` 令牌请求一个 `logger` 时，给它返回一个 `BetterLogger` 。
```ts
[{ provide: Logger, useClass: BetterLogger }]
```

#### 配置带依赖的类提供者 useClass
如果替代类提供者有自己的依赖，那就在父模块或组件的元数据属性 `providers` 中指定那些依赖。
```ts
[ UserService,
  { provide: Logger, useClass: EvenBetterLogger }]
```
在这个例子中， `EvenBetterLogger` 会在日志信息里显示用户名。 这个 `logger` 要从注入的 `UserService` 实例中来获取该用户。

```ts
@Injectable()
export class EvenBetterLogger extends Logger {
  constructor(
    private userService: UserService
  ){ 
    super(); 
  }

  override log(message: string) {
    const name = this.userService.user.name;
    super.log(`Message to ${name}: ${message}`);
  }
}
```
注入器需要提供这个新的日志服务以及该服务所依赖的 `UserService` 对象。

#### 别名类提供者 useExisting
要为类提供者设置别名，请在 `providers` 数组中使用 `useExisting` 属性指定别名和类提供者。
在下面的例子中，当组件请求新的或旧的记录器时，注入器都会注入一个 `NewLogger` 的实例。 通过这种方式， `OldLogger` 就成了 `NewLogger` 的别名。
```ts
[ NewLogger,
  // Alias OldLogger w/ reference to NewLogger
  { provide: OldLogger, useExisting: NewLogger}]
```
请确保你没有使用 `useClass` 来把 `OldLogger` 设为 `NewLogger` 的别名，因为如果这样做它就会创建两个不同的 `NewLogger` 实例。

### 为类接口指定别名 useExisting
通常，编写同一个父组件别名提供者的变体时会使用 `forwardRef` ，如下所示。
```ts
// dependency-injection-in-action/src/app/parent-finder.component.ts

providers: [{ provide: Parent, useExisting: forwardRef(() => AlexComponent) }],
```
为简化你的代码，可以使用辅助函数 `provideParent()` 来把这个逻辑提取到一个辅助函数中。
```ts
// dependency-injection-in-action/src/app/parent-finder.component.ts

// Helper method to provide the current component instance in the name of a `parentType`.
export function provideParent(component: any) {
  return { provide: Parent, useExisting: forwardRef(() => component) };
}

providers:  [ provideParent(AlexComponent) ]
```

#### 为多个类接口指定别名 useExisting
要为多个父类型指定别名（每个类型都有自己的类接口令牌），请配置 `provideParent() `以接受更多的参数。
这是一个修订版本，默认值为 `parent` 但同时也接受另一个父类接口作为可选的第二参数。
```ts
// dependency-injection-in-action/src/app/parent-finder.component.ts

// Helper method to provide the current component instance in the name of a `parentType`.
// The `parentType` defaults to `Parent` when omitting the second parameter.
export function provideParent(component: any, parentType?: any) {
  return { provide: parentType || Parent, useExisting: forwardRef(() => component) };
}

providers:  [ provideParent(BethComponent, DifferentParent) ]
```

### 注入一个对象 useValue
要注入一个对象，可以用 `useValue` 选项来配置注入器。 下面的提供者定义对象使用 `useValue` 作为 `key` 来把该变量与 `Logger` 令牌关联起来。
```ts
[{ provide: Logger, useValue: SilentLogger }]
```
在这个例子中， `SilentLogger` 是一个充当记录器角色的对象。
```ts
// An object in the shape of the logger service
function silentLoggerFn() {}

export const SilentLogger = {
  logs: ['Silent logger says "Shhhhh!". Provided via "useValue"'],
  log: silentLoggerFn
};
```

#### 注入一个配置对象
常用的对象字面量是配置对象。下列配置对象包括应用的标题和 `Web API` 的端点地址。
```ts
// src/app/app.config.ts (excerpt)

export const HERO_DI_CONFIG: AppConfig = {
  apiEndpoint: 'api.heroes.com',
  title: 'Dependency Injection'
};
```

要提供并注入配置对象，请在 `@NgModule()` 的 `providers` 数组中指定该对象。
```ts
// src/app/app.module.ts (providers)

providers: [
  UserService,
  { provide: APP_CONFIG, useValue: HERO_DI_CONFIG }
],
```
#### 使用 `InjectionToken` 对象
可以定义和使用一个 `InjectionToken` 对象来为非类的依赖选择一个提供者令牌。
下列例子定义了一个类型为 `InjectionToken` 的 `APP_CONFIG` 。

```ts
// src/app/app.config.ts
import { InjectionToken } from '@angular/core';

export const APP_CONFIG = new InjectionToken<AppConfig>('app.config');
```
可选的参数 `<AppConfig>` 和令牌描述 `app.config` 指明了此令牌的用途。
接着，用 `APP_CONFIG` 这个 `InjectionToken` 对象在组件中注册依赖提供者。
```ts
// src/app/providers.component.ts
providers: [{ provide: APP_CONFIG, useValue: HERO_DI_CONFIG }] 
```

现在，借助参数装饰器 `@Inject()`，你可以把这个配置对象注入到构造函数中。

```ts
// src/app/app.component.ts
constructor(
  @Inject(APP_CONFIG) config: AppConfig
) {
  this.title = config.title;
}
```

##### 接口和依赖注入
虽然 `TypeScript` 的 `AppConfig` 接口可以在类中提供类型支持，但它在依赖注入时却没有任何作用。在 `TypeScript` 中，接口是一项设计期工件，它没有可供 `DI` 框架使用的运行时表示形式或令牌。
当转译器把 `TypeScript` 转换成 `JavaScript` 时，接口就会消失，因为 `JavaScript` 没有接口。
由于 `Angular` 在运行期没有接口，所以该接口不能作为令牌，也不能注入它。
```ts
// Can't use interface as provider token
[{ provide: AppConfig, useValue: HERO_DI_CONFIG })]

// Can't inject using the interface as the parameter type
constructor(private config: AppConfig){ }
```

### 使用工厂提供者
要想根据运行前尚不可用的信息创建可变的依赖值，可以使用工厂提供者。
在下面的例子中，只有授权用户才能看到 `HeroService` 中的秘密英雄。授权可能在单个应用会话期间发生变化，比如改用其他用户登录。
要想在 `UserService` 和 `HeroService` 中保存敏感信息，就要给 `HeroService` 的构造函数传一个逻辑标志来控制秘密英雄的显示。
```ts
// src/app/heroes/hero.service.ts (excerpt)
constructor(
  private logger: Logger,
  private isAuthorized: boolean // 判断是否有权限
) { }

getHeroes() {
  const auth = this.isAuthorized ? 'authorized ' : 'unauthorized';
  this.logger.log(`Getting heroes for ${auth} user.`);
  return HEROES.filter(hero => this.isAuthorized || !hero.isSecret);
}
```
要实现 `isAuthorized` 标志，可以用工厂提供者来为 `HeroService` 创建一个新的 `logger` 实例。
```ts
// src/app/heroes/hero.service.provider.ts (excerpt)

const heroServiceFactory = (logger: Logger, userService: UserService) =>
  new HeroService(logger, userService.user.isAuthorized);
```
这个工厂函数可以访问 `UserService` 。你可以同时把 `Logger` 和 `UserService` 注入到工厂提供者中，这样注入器就可以把它们传给工厂函数了。

```ts
// src/app/heroes/hero.service.provider.ts (excerpt)

const heroServiceFactory = (logger: Logger, userService: UserService) =>
  new HeroService(logger, userService.user.isAuthorized);

export const heroServiceProvider =
  { provide: HeroService,
    useFactory: heroServiceFactory,
    deps: [Logger, UserService] // useFactory 的参数
  };
```
- `useFactory` 字段指定该提供者是一个**工厂函数**，其实现代码是 `heroServiceFactory。`
- `deps` 属性是一个**提供者令牌数组**。 `Logger` 和 `UserService` 类都是自己类提供者的令牌。该注入器解析了这些令牌，并把相应的服务注入到 `heroServiceFactory` 工厂函数的参数中。

通过把工厂提供者导出为变量 `heroServiceProvider` ，就能让工厂提供者变得可复用。
下面这两个并排的例子展示了在 `providers` 数组中，如何用 `heroServiceProvider` 替换 `HeroService`

```ts
// src/app/heroes/heroes.component (v3)

import { Component } from '@angular/core';
import { heroServiceProvider } from './hero.service.provider';

@Component({
  selector: 'app-heroes',
  providers: [ heroServiceProvider ],
  template: `
    <h2>Heroes</h2>
    <app-hero-list></app-hero-list>
  `
})
export class HeroesComponent { }
```

```ts
// src/app/heroes/heroes.component (v2)

import { Component } from '@angular/core';
import { HeroService } from './hero.service';

@Component({
  selector: 'app-heroes',
  providers: [ HeroService ],
  template: `
    <h2>Heroes</h2>
    <app-hero-list></app-hero-list>
  `
})
export class HeroesComponent { }
```