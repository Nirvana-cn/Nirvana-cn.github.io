---
title: Jest 入门指南
date: 2018-12-29 16:17:21
tags:
- jest
- unit test
categories:
- 前端,每天学一点
---

首先这并不是一篇完整的关于`Jest`的教程，只是个人在接触`Jest`时的一点学习笔记，在此对`Jest`的特性和基础用法进行一些总结。大部分内容都是对官方文档的一些翻译，只是让想了解`Jest`的人快速知道一下`Jest`是做什么的，怎么做。


<!-- more -->

## 前言
`Jest`是`Facebook`开源的一个前端测试框架，主要用于`React`和`React Native`的单元测试（同样也可以进行`JavaScript`测试），已被集成在`create-react-app`中。

`Jest`特点：
1. 易用性：基于`Jasmine`，提供断言库，支持多种测试风格
2. 适应性：`Jest`是模块化、可扩展和可配置的
3. 沙箱和快照：`Jest`内置了`JSDOM`，能够模拟浏览器环境，并且并行执行
4. 快照测试：`Jest`能够对`React`组件树进行序列化，生成对应的字符串快照，通过比较字符串提供高性能的UI检测
5. `Mock`系统：`Jest`实现了一个强大的`Mock`系统，支持自动和手动`mock`
6. 支持异步代码测试：支持`Promise`和`async/await`
7. 自动生成静态分析结果：内置`Istanbul`，测试代码覆盖率，并生成对应的报告

(非必须) `Jest`环境配置，在`package.json`中的`script`中增加 `"test: jest --config .jest.js"`

``` javascript
module.exports = {
  setupFiles: [
    './test/setup.js',
  ],
  moduleFileExtensions: [
    'js',
    'jsx',
  ],
  testPathIgnorePatterns: [
    '/node_modules/',
  ],
  testRegex: '.*\\.test\\.js$',
  collectCoverage: false,
  collectCoverageFrom: [
    'src/components/**/*.{js}',
  ],
  moduleNameMapper: {
    "\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$": "<rootDir>/__mocks__/fileMock.js",
    "\\.(css|less|scss)$": "<rootDir>/__mocks__/styleMock.js"
  },
  transform: {
    "^.+\\.js$": "babel-jest"
  },
};
```
配置说明：
- `setupFiles`：配置文件，在运行测试案例代码之前，`Jest`会先运行这里的配置文件来初始化指定的测试环境
- `moduleFileExtensions`：代表支持加载的文件名
- `testPathIgnorePatterns`：用正则来匹配不用测试的文件
- `testRegex`：正则表示的测试文件，测试文件的格式为`xxx.test.js`
- `collectCoverage`：是否生成测试覆盖报告，如果开启，会增加测试的时间
- `collectCoverageFrom`：生成测试覆盖报告是检测的覆盖文件
- `moduleNameMapper`：代表需要被`Mock`的资源名称
- `transform`：用`babel-jest`来编译文件，生成`ES6/7`的语法

** `ES6`提供的 `import` 来导入模块的方式，`Jest`本身是不支持的,可以通过安装`babel-jest、babel-core、babel-preset-env、babel-plugin-transform-runtime`这几个依赖让我们可以使用`ES6`的语法特性进行单元测试。
在项目的根目录下添加`.babelrc`文件，并在文件复制如下内容:
```
{
  "presets": ["env"],
  "plugins": ["transform-runtime"]
}
```

## Globals  API

- `describe(name, fn)`：描述块，讲一组功能相关的测试用例组合在一起
- `test(name, fn, timeout)`：别名it，用来放测试用例
- `afterAll(fn, timeout)`：所有测试用例跑完以后执行的方法
- `beforeAll(fn, timeout)`：所有测试用例执行之前执行的方法
- `afterEach(fn)`：在每个测试用例执行完后执行的方法
- `beforeEach(fn)`：在每个测试用例执行之前需要执行的方法

全局和`describe`都可以有上面四个周期函数，`describe`的`after`函数优先级要高于全局的`after`函数，`describe`的`before`函数优先级要低于全局的`before`函数，完整的例子见Example.4。

```javascript
const myBeverage = {
    delicious: true,
    sour: false,
};

beforeAll(() => {
    console.log('global before all');
});

beforeEach(() =>{
    console.log('global before each');
});

describe('my beverage', () => {
    beforeAll(() => {
        console.log('inner test before all');
    });

    beforeEach(() => {
        console.log('inner test before each');
    });

    test('is delicious', () => {
        expect(myBeverage.delicious).toBeTruthy();
    });

    test('is not sour', () => {
        expect(myBeverage.sour).toBeFalsy();
    });
});

// console.log describe.test.js:7
// global before all
//
// console.log describe.test.js:16
// inner test before all
//
// console.log describe.test.js:11
// global before each
//
// console.log describe.test.js:20
// inner test before each
//
// console.log describe.test.js:11
// global before each
//
// console.log describe.test.js:20
// inner test before each
```

#### describe和test的执行顺序

`Jest`在执行任何实际`test`之前，会首先执行文件中所有`describe`块的处理程序。这是在`before*`和`after*`中进行设置，而不是在`describe`块中进行设置的另一个原因。默认情况下，`describe`块执行完成后，`Jest`按照在收集阶段遇到的顺序连续运行所有`test`，等待每个`test`完成并在继续之前进行整理。实例见Example.7。

## 常见断言

1. `expect(value)`：要测试一个值进行断言的时候，要使用`expect`对值进行包裹
2. `toBe(value)`：使用`Object.is`来进行比较，如果进行浮点数的比较，要使用`toBeCloseTo`
3. `not`：用来取反
4. `toEqual(value)`：用于对象的深比较
5. `toMatch(regexpOrString)`：用来检查字符串是否匹配，可以是正则表达式或者字符串
6. `toContain(item)`：用来判断`item`是否在一个数组中，也可以用于字符串的判断
7. `toBeNull(value)`：只匹配`null`
8. `toBeUndefined(value)`：只匹配`undefined`
9. `toBeDefined(value)`：与`toBeUndefined`相反
10. `toBeTruthy(value)`：匹配任何使`if`语句为真的值
11. `toBeFalsy(value)`：匹配任何使`if`语句为假的值
12. `toBeGreaterThan(number)`： 大于
13. `toBeGreaterThanOrEqual(number)`：大于等于
14. `toBeLessThan(number)`：小于
15. `toBeLessThanOrEqual(number)`：小于等于
16. `toBeInstanceOf(class)`：判断是不是`class`的实例
17. `anything(value)`：匹配除了`null`和`undefined`以外的所有值
18. `resolves`：用来取出`promise`为`fulfilled`时包裹的值，支持链式调用
19. `rejects`：用来取出`promise`为`rejected`时包裹的值，支持链式调用
20. `toHaveBeenCalled()`：用来判断`mock function`是否被调用过
21. `toHaveBeenCalledTimes(number)`：用来判断`mock function`被调用的次数
22. `assertions(number)`：验证在一个测试用例中有`number`个断言被调用
23. `extend(matchers)`：自定义一些断言

## 异步函数测试

对于异步函数的测试，官方文档提供了四种方法，分别为`callback、promise、resolve/reject`和`async/await`函数，相比于callback，后三者语义上无疑更清晰，在Example.3中，我们分别演示了后三者。我们使用axios发起异步请求，等待响应数据。`expect.assertions(n)`验证在测试期间是否调用了一定数量的断言。这在测试异步代码时通常很有用，以确保实际调用异步中的断言。

比如，我们发起两次异步请求，但是设置`expect.assertions(1)`或`expect.assertions(3)`，这个时候就会报错，造成测试无法通过，因为实际异步请求返回应该2次。所以需要`expect.assertions()`来确保异步请求响应次数与我们的期望一致。

```javascript
describe('async test', () => {

    test('method three', async () => {
        expect.assertions(1);
        let data = await fetch()
        expect(data.name).toMatch(/ham/)
        let data2 = await fetch()
        expect(data2.username).toMatch(/Bret/)
    })
})

// Expected one assertion to be called but received two assertion calls.
```

## Mock Function

在项目中，一个模块的方法内常常会去调用另外一个模块的方法。在单元测试中，我们可能并不需要关心内部调用的方法的执行过程和结果，只想知道它是否被正确调用即可，甚至会指定该函数的返回值。此时，使用`Mock`函数是十分有必要。

`Mock`函数提供的以下三种特性，在我们写测试代码时十分有用：

- 捕获函数调用情况
- 设置函数返回值
- 改变函数的内部实现

`Mock`函数常用方法：

1.`mockFn.mockName(value)`：设置`mock`函数的名字

2.`mockFn.getMockName()`： 返回`mockFn.mockName(value)`中设置的名字

3.`mockFn.mock.calls`：`mock`函数的调用信息

`mockFn.mock.calls`返回一个数组，数组中的每一个元素又是一个数组，包含`mock`函数的调用信息。比如，一个被调用两次的模拟函数f，参数为`f('arg1'，'arg2')`，然后使用参数`f('arg3'，'arg4')`，`mockFn.mock.calls`返回的数组形式如下：

> [['arg1', 'arg2'], ['arg3', 'arg4']]

因此，`mockFn.mock.calls.length`代表`mock`函数被调用次数，`mockFn.mock.calls[0][0]`代表第一次调用传入的第一个参数，以此类推。

4.`mockFn.mock.results`：`mock`函数的`return`值，以数组存储

5.`mockFn.mock.instances`：`mock`函数实例

```javascript
const mockFn = jest.fn();

const a = new mockFn();
const b = new mockFn();

mockFn.mock.instances[0] === a; // true
mockFn.mock.instances[1] === b; // true
```

6.`mockFn.mockImplementation(fn)`：创建一个`mock`函数

注意：`jest.fn(implementation)`是`jest.fn().mockImplementation(implementation)`的简写。

7.`mockFn.mockImplementationOnce(fn)`：创建一个`mock`函数

该函数将用作对`mocked`函数的一次调用的`mock`的实现。可以链式调用，以便多个函数调用产生不同的结果。

```javascript
const myMockFn = jest
  .fn()
  .mockImplementationOnce(cb => cb(null, true))
  .mockImplementationOnce(cb => cb(null, false));

myMockFn((err, val) => console.log(val)); // true

myMockFn((err, val) => console.log(val)); // false
```

当`mocked`函数用完使用`mockImplementationOnce`定义的实现时，如果调用它们，它将使用`jest.fn(()=> defaultValue)`或`.mockImplementation(()=> defaultValue)`执行默认实现集：
```javascript
const myMockFn = jest
  .fn(() => 'default')
  .mockImplementationOnce(() => 'first call')
  .mockImplementationOnce(() => 'second call');

// 'first call', 'second call', 'default', 'default'
console.log(myMockFn(), myMockFn(), myMockFn(), myMockFn());
```

8.`mockFn.mockReturnThis()`：`jest.fn()`的语法糖

```javascript
jest.fn(function() {
  return this;
});
```

9.`mockFn.mockReturnValue(value)`：接受一个值作为调用`mock`函数时的返回值

```javascript
const mock = jest.fn();
mock.mockReturnValue(42);
mock(); // 42
mock.mockReturnValue(43);
mock(); // 43
```

10.`mockFn.mockReturnValueOnce(value)`：接受一个值作为调用`mock`函数时的返回值，可以链式调用，以便产生不同的结果。

当不再使用`mockReturnValueOnce`值时，调用将返回`mockReturnValue`指定的值。

```javascript
const myMockFn = jest
  .fn()
  .mockReturnValue('default')
  .mockReturnValueOnce('first call')
  .mockReturnValueOnce('second call');

// 'first call', 'second call', 'default', 'default'
console.log(myMockFn(), myMockFn(), myMockFn(), myMockFn());
```

11.`mockFn.mockResolvedValue(value)`：`mock`异步函数的语法糖

实现上类似于

```javascript
jest.fn().mockImplementation(() => Promise.resolve(value));
```

用于在`test`中模拟异步函数

```javascript
test('async test', async () => {
  const asyncMock = jest.fn().mockResolvedValue(43);

  await asyncMock(); // 43
});
```

12.`mockFn.mockResolvedValueOnce(value)`：语法糖

实现上类似于

```javascript
jest.fn().mockImplementationOnce(() => Promise.resolve(value));
```

```javascript
test('async test', async () => {
  const asyncMock = jest
    .fn()
    .mockResolvedValue('default')
    .mockResolvedValueOnce('first call')
    .mockResolvedValueOnce('second call');

  await asyncMock(); // first call
  await asyncMock(); // second call
  await asyncMock(); // default
  await asyncMock(); // default
});
```

13.`mockFn.mockRejectedValue(value)`：语法糖

实现上类似于

```javascript
jest.fn().mockImplementation(() => Promise.reject(value));
```

```javascript
test('async test', async () => {
  const asyncMock = jest.fn().mockRejectedValue(new Error('Async error'));

  await asyncMock(); // throws "Async error"
});
```

14.`mockFn.mockRejectedValueOnce(value)`：语法糖

实现上类似于

```javascript
jest.fn().mockImplementationOnce(() => Promise.reject(value));
```

```javascript
test('async test', async () => {
  const asyncMock = jest
    .fn()
    .mockResolvedValueOnce('first call')
    .mockRejectedValueOnce(new Error('Async error'));

  await asyncMock(); // first call
  await asyncMock(); // throws "Async error"
});
```

15.`mockFn.mockClear()`：重置所有存储在`mockFn.mock.calls` 和 `mockFn.mock.instances`数组中的信息

当你想要清除两个断言之间的模拟使用数据时，这通常很有用。

16.`mockFn.mockReset()`：完成`mockFn.mockClear()`所做的所有事情，还删除任何模拟的返回值或实现

当你想要将模拟完全重置回其初始状态时，这非常有用。（请注意，重置`spy`将导致函数没有返回值）。

17.`mockFn.mockRestore()`：完成`mockFn.mockReset()`所做的所有事情，并恢复原始（非模拟）实现

当你想在某些测试用例中模拟函数并在其他测试用例中恢复原始实现时，这非常有用。

## jest.mock and jest.spyOn

`jest`对象上有`fn,mock,spyOn`三个方法，在实际项目的单元测试中，`jest.fn()`常被用来进行某些有回调函数的测试；`jest.mock()`可以`mock`整个模块中的方法，当某个模块已经被单元测试100%覆盖时，使用`jest.mock()`去`mock`该模块，节约测试时间和测试的冗余度是十分必要；当需要测试某些必须被完整执行的方法时，常常需要使用`jest.spyOn()`。

如Example.8中所示，`event.js`中调用了`fetch.js`的方法，`fetch.js`文件夹中封装的请求方法可能我们在其他模块被调用的时候，并不需要进行实际的请求（请求方法已经通过单侧或需要该方法返回非真实数据）。此时，使用`jest.mock()`去`mock`整个模块是十分有必要的。

```javascript
// event.test.js
import events from './event';
import fetch from './fetch';

jest.mock('./fetch.js');

test('mock 整个 fetch.js模块', async () => {
    expect.assertions(2);
    await events.getPostList();
    expect(fetch.fetchPostsList).toHaveBeenCalled();
    expect(fetch.fetchPostsList).toHaveBeenCalledTimes(1);
});
```

`jest.spyOn()`方法同样创建一个`mock`函数，但是该`mock`函数不仅能够捕获函数的调用情况，还可以正常的执行被`spy`的函数。实际上，`jest.spyOn()`是`jest.fn()`的语法糖，它创建了一个和被`spy`的函数具有相同内部代码的`mock`函数。

`event.test.js`是j`est.mock()`的示例代码，从控制台中可以看到`console.log('fetchPostsList be called!')`;这行代码并没有被打印，这是因为通过`jest.mock()`后，模块内的方法是不会被`jest`所实际执行的。这时我们就需要使用`jest.spyOn()`。

```javascript
// event2.test.js
import events from './event';
import fetch from './fetch';

test('使用jest.spyOn()监控fetch.fetchPostsList被正常调用', async() => {
    expect.assertions(2);
    const spyFn = jest.spyOn(fetch, 'fetchPostsList');
    await events.getPostList();
    expect(spyFn).toHaveBeenCalled();
    expect(spyFn).toHaveBeenCalledTimes(1);
})
```

执行测试之后，可以看到控制台中的打印信息，说明通过`jest.spyOn()`，`fetchPostsList`被正常的执行了。

> TIPS：在jest中如果想捕获函数的调用情况，则该函数必须被mock或者spyOn！

## Timer Mock

`jest`可以模拟定时器从而允许自主控制时间流逝。模拟定时器运行可以方便测试，比如不必等待一个很长的延时而是直接获取结果。

`jest`对象上与`timer mock`相关的方法主要有以下个：

1. `jest.useFakeTimers()`：指示`Jest`使用标准计时器函数的假版本（`setTimeout，setInterval，clearTimeout，clearInterval，nextTick，setImmediate和clearImmediate`）。
2. `jest.useRealTimers()`：指示`Jest`使用标准计时器功能的真实版本。
3. `jest.clearAllTimers()`：从计时器系统中清除任何等待的计时器。
4. `jest.runAllTicks()`：执行微任务队列中的所有任务（通常通过`process.nextTick`在节点中连接）。
5. `jest.runAllTimers()`：执行宏任务队列中的所有任务。
6. `jest.runAllImmediates()`：通过`setImmediate()`执行任务队列中的所有任务。
7. `jest.advanceTimersByTime(n)`：执行宏任务队列中的所有任务，当该`API`被调用时，所有定时器被提前 `n` 秒。
8. `jest.runOnlyPendingTimers()`：仅执行宏任务中当前正在等待的任务。

举例来说(见Example.5):

```javascript
// timer.js
function timerGame(callback) {
    console.log('Ready....go!');
    setTimeout(() => {
        console.log('Times up -- stop!');
        callback && callback();
    }, 1000);
}

module.exports = timerGame;
```

我们在`timer.js`中设定了一个1s钟之后才会执行的定时器，但是测试代码是同步执行，通过`timer mock`，我们不必等待定时器执行完成就可以完成测试。

```javascript
const timer = require('./timer');
const callback = jest.fn();

jest.useFakeTimers();

test('calls the callback after 1 second', () => {
    timer(callback);

    expect(callback).not.toBeCalled();

    expect(setTimeout).toHaveBeenCalledTimes(1);
    expect(setInterval).toHaveBeenCalledTimes(0);
    expect(setTimeout).toHaveBeenLastCalledWith(expect.any(Function), 1000);

    jest.runAllTimers();

    expect(callback).toBeCalled();
    expect(callback).toHaveBeenCalledTimes(1);
});
```

在上面的代码中，我们已经在第四行声明使用假时间，在`test`块中，虽然`setTimeout`尚未执行完毕，但是测试已经完成，`setTimeout`执行一次，没有`setInterval`执行，这与期望一致。接下来调用`jest.runAllTimers()`使得所有定时器立即执行完毕，控制台打印定时器中的输出。

对于递归定时器的情况，如果使用`jest.runAllTimers()`，所有的定时器就无限循环了，这个时候就需要用到`jest.runOnlyPendingTimers()`了，因为`runOnlyPendingTimers`的过程中不会产生新的定时器，从而避免了无限循环的问题，如Example.5 中`pendingTimer`所示。

`jest.advanceTimersByTime(n)`也很容易理解，就是将所有定时器提前`n`秒执行。如下所示，在未调用`jest.advanceTimersByTime(n)`之前，`callback`还没有被调用，然后通过`jest.advanceTimersByTime(1000)`让定时器提前`1s`执行，因此接下来的断言不会报错。

```javascript
// timer3.test.js
const timerGame = require('./timer');
const callback = jest.fn();
jest.useFakeTimers();

test('calls the callback after 1 second via advanceTimersByTime', () => {
    timerGame(callback);

    // At this point in time, the callback should not have been called yet
    expect(callback).not.toBeCalled();

    // Fast-forward until all timers have been executed
    jest.advanceTimersByTime(1000);

    // Now our callback should have been called!
    expect(callback).toBeCalled();
    expect(callback).toHaveBeenCalledTimes(1);
});
```

同样的，如果使用`jest.advanceTimersByTime(500)`提前`0.5s`，上面的测试可以进行如下修改。

```javascript
jest.advanceTimersByTime(500);

expect(callback).not.toBeCalled();
expect(callback).toHaveBeenCalledTimes(0);
```

## Manual Mock

`Manual Mock`用于存储模拟数据的功能。例如，您可能希望创建一个允许您使用虚假数据的手动模拟，而不是访问网站或数据库等远程资源。这可以确保您的测试快速且稳定。

通过在紧邻模块的`__mocks __`子目录中编写模块来定义手动模拟。例如，要在`models`目录中模拟一个名为`user`的模块(如Example.9 所示)，请创建一个名为`user.js`的文件并将其放在`models / __ mocks__`目录中。同时请注意`__mocks__`文件夹区分大小写。

** 注意比较Example.8 和 Example.9 中`jest.mock`的不同，由于Example.8中没有`__mocks__`文件夹，所以`mock`并不会真正运行，而在Example.9中会使用`__mocks__`下的同名文件进行替换。

```javascript
// userMocked.test.js
import user from './models/user';

jest.mock('./models/user');

test('if user model is mocked', () => {
    expect(user.getAuthenticated()).toEqual({age: 622, name: 'Mock name'});
});
```

如果您正在模拟的模块是`Node`模块（例如：`lodash`），则`mock`应放在与`node_modules`相邻的`__mocks__`目录中，并将自动模拟。没有必要显式调用`jest.mock('module_name')`。但是如果想模拟`Node`的核心模块（例如：`fs`或`path`），那么明确地调用例如： `jest.mock('path')`是必需的，因为默认情况下不会模拟核心`Node`模块。

如果您正在使用`ES6`模块导入，那么您通常倾向于将导入语句放在测试文件的顶部。但是通常你需要指示`Jest`在模块使用它之前使用模拟。出于这个原因，`Jest`会自动将`jest.mock`调用提升到模块的顶部（在任何导入之前）。

## ES6 Class Mock

如Example.6中所示，分别有`SoundPlayer`类和使用该类的`SoundPlayerConsumer`类。我们将在`SoundPlayerConsumer`的测试中模拟`SoundPlayer`。

```javascript
// sound-player.js
export default class SoundPlayer {
  constructor() {
    this.foo = 'bar';
  }

  playSoundFile(fileName) {
    console.log('Playing sound file ' + fileName);
  }
}
```

```javascript
// sound-player-consumer.js
import SoundPlayer from './sound-player';

export default class SoundPlayerConsumer {
  constructor() {
    this.soundPlayer = new SoundPlayer();
  }

  playSomethingCool() {
    const coolSoundFileName = 'song.mp3';
    this.soundPlayer.playSoundFile(coolSoundFileName);
  }
}
```

分别有4种方法可以创建`ES6`类的模拟：

1.Automatic mock

使用该方法，类中的所有方法调用状态保存在`theAutomaticMock.mock.instances[index].methodName.mock.calls`中。同时，如果您在类中使用了箭头函数，它们将不会成为模拟的一部分。因为箭头函数不存在于对象的原型上，它们只是包含对函数的引用的属性。

** 注：由于Example.6中同时存在`Automatic mock和Manual mock`，所以在使用该方法时需要把`__mocks__`文件夹改个名。

2.Manual mock

在相邻的`__mocks__`文件夹下生成同名文件从而进行替换模拟。

** 注意：使用`manual mock`时确保`__mocks__`文件夹命名正确。

3.Calling jest.mock() with the module factory parameter

使用此种方法就是在`jest.mock()`中添加一个参数：`jest.mock(path，moduleFactory)`接受`moduleFactory`参数，`moduleFactory`返回一个模拟的函数。为了模拟构造函数，`moduleFactory`必须返回构造函数，换句话说，模块工厂必须是返回函数的函数。

4.Replacing the mock using mockImplementation() or mockImplementationOnce()

对`jest.mock`的调用会被提升到代码的顶部。 您通过在现有`mock`上调用`mockImplementation()`（或`mockImplementationOnce()`）而不是使用`factory`参数，从而稍后指定模拟（例如，在`beforeAll()`），或者在测试之间更改模拟。

## Bypassing Module Mock

`Jest`允许您模拟测试中的整个模块，这有助于测试代码是否正确，函数调用是否正确。但是，有时您可能希望在测试文件中使用模拟模块的一部分，在这种情况下，您希望访问原始实现，而不是模拟版本。`jest.requireActual()`允许你导入实际的版本，而不是模拟的版本。

## 推荐阅读

使用Jest测试JavaScript (入门篇)：[>>>点我进入](https://segmentfault.com/a/1190000016232248)

使用Jest测试JavaScript(Mock篇)：[>>>点我进入](https://segmentfault.com/a/1190000016717356)

顶级测试框架Jest指南：[>>>点我进入](https://segmentfault.com/a/1190000016399447)

Jest cheat sheet：[>>>点我进入](https://github.com/sapegin/jest-cheat-sheet)

Jest官方demo地址：[>>>点我进入](https://github.com/facebook/jest/tree/master/examples)

本文源码地址：[>>>点我进入](https://github.com/Nirvana-cn/Unit_Testing_with_Jest)



