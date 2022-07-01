## 词汇解释
- `DI`: `Dependency Injection` 依赖注入
- `Provider`: 提供商（提供者）
- `Injector`: 注入器
- `Injectable`: 依赖注入服务
- `Token`: 令牌

## 依赖对象创建的四种方式
- useClass
- useValue
- useExisting
- useFactory

## 多级依赖注入设计
- ElementInjector 元素注入器
- ModuleInjector 模块注入器

### useClass
```
@Injectable()
export class ApiService {
   constructor(
      public http: Http, 
      public loadingCtrl: LoadingController) {
   }
   ...
}

@NgModule({
  ...
  providers: [
       { provide: ApiService, useClass: ApiService } // 可使用简洁的语法，即直接使用ApiService
  ]
})
export class CoreModule { }
```

### useValue
```
{ provide: 'API_URL', useValue: 'http://my.api.com/v1' }
```

### useExisting
```
{ provide: 'ApiServiceAlias', useExisting: ApiService }
```

### useFactory
```
export function configFactory(config: AppConfig) {
  return () => config.load();
}

@NgModule({
  ...
  providers: [
       { provide: APP_INITIALIZER, useFactory: configFactory, 
        deps: [AppConfig], multi: true }
  ]
})
export class CoreModule { }
```

**`provide: 'API_URL'` 中 `'API_URL'` 就是令牌**

`@Injectable()` 装饰器会指定 `Angular` 可以在 `DI` 体系中使用此类。元数据 `providedIn: 'root'` 表示 `HeroService` 在**整个应用程序中**都是**可见**的。

### 注入服务
``` TS
// src/app/heroes/hero-list.component (constructor signature)
constructor(heroService: HeroService)
```