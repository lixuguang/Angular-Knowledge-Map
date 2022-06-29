## 用管道转换数据
管道用来对字符串、货币金额、日期和其他显示数据进行转换和格式化。
管道是一些简单的函数，可以在模板表达式中用来*接受输入值并返回一个转换后的值*。

### 常用的内置管道
- `DatePipe` ：根据本地环境中的规则格式化日期值。
- `UpperCasePipe` ：把文本全部转换成大写。
- `LowerCasePipe` ：把文本全部转换成小写。
- `CurrencyPipe` ：把数字转换成货币字符串，根据本地环境中的规则进行格式化。
- `DecimalPipe` ：把数字转换成带小数点的字符串，根据本地环境中的规则进行格式化。
- `PercentPipe` ：把数字转换成百分比字符串，根据本地环境中的规则进行格式化。

### 管道参数
```TS
{{ slice:1:5 }} // 创建一个新数组或字符串，它以第 1 个元素开头，并以第 5 个元素结尾。
```

### 管道链
```TS
{{ birthday | date | uppercase}} // 先日期格式化，然后全小写。
```

### 自定义管道
```TS
// src/app/exponential-strength.pipe.ts
import { Pipe, PipeTransform } from '@angular/core';

// ① 给管道起名
@Pipe({
	name: 'exponentialStrength'
	pure: true // 默认为true
})

// ② 创建管道类，实现 transform 抽象方法
export class ExponentialStrengthPipe implements PipeTransform {
  transform(value: number, exponent = 1): number {
    return Math.pow(value, exponent);
  }
}


// src/app/power-booster.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-power-booster',
  template: `
    <h2>Power Booster</h2>
    // ③ 调用管道
    <p>Super power boost: {{2 | exponentialStrength: 10}}</p>
  `
})
export class PowerBoosterComponent { }
```

#### 纯管道VS非纯管道
`pure`属性控制管道的是纯的还是非纯的，通过默认情况下，管道会定义成纯的(pure)，这样 Angular 只有在检测到输入值发生了纯变更时才会执行该管道。
纯变更是对原始输入值（比如 String、Number、Boolean 或 Symbol ）的变更，或是对对象引用的变更（比如 Date、Array、Function、Object）。
纯管道必须使用纯函数，它能处理输入并返回没有副作用的值。换句话说，给定相同的输入，纯函数应该总是返回相同的输出。
使用纯管道，Angular 会忽略复合对象中的变化，例如往现有数组中新增的元素，因为检查原始值或对象引用比对对象中的差异进行深度检查要快得多。Angular 可以快速判断是否可以跳过执行该管道并更新视图。

### `AsyncPipe` 从一个可观察对象中解包数据
使用内置的 AsyncPipe 接受一个可观察对象作为输入，并自动订阅输入。
AsyncPipe 是一个非纯管道，可以节省组件中的样板代码，以维护订阅，并在数据到达时持续从该可观察对象中提供值。

## 管道的优先级
管道操作符要比三目运算符(?:)的优先级高，这意味着 a ? b : c | x 会被解析成 a ? b : (c | x)。