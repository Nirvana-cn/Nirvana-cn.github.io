---
title: Jest 入门指南
date: 2018-12-29 16:17:21
tags:
- Jest
- Unit Test
categories:
- Jest
---


<!-- more -->
Jest是Facebook开源的一个前端测试框架，主要用于React和React Native的单元测试（同样也可以进行JavaScript测试），已被集成在create-react-app中。

Jest特点：
1. 易用性：基于Jasmine，提供断言库，支持多种测试风格
2. 适应性：Jest是模块化、可扩展和可配置的
3. 沙箱和快照：Jest内置了JSDOM，能够模拟浏览器环境，并且并行执行
4. 快照测试：Jest能够对React组件树进行序列化，生成对应的字符串快照，通过比较字符串提供高性能的UI检测
5. Mock系统：Jest实现了一个强大的Mock系统，支持自动和手动mock
6. 支持异步代码测试：支持Promise和async/await
7. 自动生成静态分析结果：内置Istanbul，测试代码覆盖率，并生成对应的报告

(非必须) Jest环境配置，在package.json中的script中增加"test: jest --config .jest.js"

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
- setupFiles：配置文件，在运行测试案例代码之前，Jest会先运行这里的配置文件来初始化指定的测试环境
- moduleFileExtensions：代表支持加载的文件名
- testPathIgnorePatterns：用正则来匹配不用测试的文件
- testRegex：正则表示的测试文件，测试文件的格式为xxx.test.js
- collectCoverage：是否生成测试覆盖报告，如果开启，会增加测试的时间
- collectCoverageFrom：生成测试覆盖报告是检测的覆盖文件
- moduleNameMapper：代表需要被Mock的资源名称
- transform：用babel-jest来编译文件，生成ES6/7的语法

** ES6提供的 import 来导入模块的方式，Jest本身是不支持的,
可以通过安装babel-jest、babel-core、babel-preset-env、babel-plugin-transform-runtime这几个依赖让我们可以使用ES6的语法特性进行单元测试。
在项目的根目录下添加.babelrc文件，并在文件复制如下内容:
```
{
  "presets": ["env"],
  "plugins": ["transform-runtime"]
}
```

## Globals  API

- describe(name, fn)：描述块，讲一组功能相关的测试用例组合在一起
- test(name, fn, timeout)：别名it，用来放测试用例
- afterAll(fn, timeout)：所有测试用例跑完以后执行的方法
- beforeAll(fn, timeout)：所有测试用例执行之前执行的方法
- afterEach(fn)：在每个测试用例执行完后执行的方法
- beforeEach(fn)：在每个测试用例执行之前需要执行的方法

全局和describe都可以有上面四个周期函数，describe的after函数优先级要高于全局的after函数，describe的before函数优先级要低于全局的before函数，完整的例子见Example.4。

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

Jest在执行任何实际test之前，会首先执行文件中所有describe块的处理程序。这是在before*和after*中进行设置，而不是在describe块中进行设置的另一个原因。
默认情况下，describe块执行完成后，Jest按照在收集阶段遇到的顺序连续运行所有test，等待每个test完成并在继续之前进行整理。实例见Example.7。

## 常见断言

1. expect(value)：要测试一个值进行断言的时候，要使用expect对值进行包裹
2. toBe(value)：使用Object.is来进行比较，如果进行浮点数的比较，要使用toBeCloseTo
3. not：用来取反
4. toEqual(value)：用于对象的深比较
5. toMatch(regexpOrString)：用来检查字符串是否匹配，可以是正则表达式或者字符串
6. toContain(item)：用来判断item是否在一个数组中，也可以用于字符串的判断
7. toBeNull(value)：只匹配null
8. toBeUndefined(value)：只匹配undefined
9. toBeDefined(value)：与toBeUndefined相反
10. toBeTruthy(value)：匹配任何使if语句为真的值
11. toBeFalsy(value)：匹配任何使if语句为假的值
12. toBeGreaterThan(number)： 大于
13. toBeGreaterThanOrEqual(number)：大于等于
14. toBeLessThan(number)：小于
15. toBeLessThanOrEqual(number)：小于等于
16. toBeInstanceOf(class)：判断是不是class的实例
17. anything(value)：匹配除了null和undefined以外的所有值
18. resolves：用来取出promise为fulfilled时包裹的值，支持链式调用
19. rejects：用来取出promise为rejected时包裹的值，支持链式调用
20. toHaveBeenCalled()：用来判断mock function是否被调用过
21. toHaveBeenCalledTimes(number)：用来判断mock function被调用的次数
22. assertions(number)：验证在一个测试用例中有number个断言被调用
23. extend(matchers)：自定义一些断言

## 异步函数测试

对于异步函数的测试，官方文档提供了四种方法，分别为callback、promise、resolve/reject和async/await函数

相比于callback，后三者语义上无疑更清晰，在Example.3中，我们分别演示了后三者。

我们使用axios发起异步请求，等待响应数据。

expect.assertions（n）验证在测试期间是否调用了一定数量的断言。这在测试异步代码时通常很有用，以确保实际调用异步中的断言。

比如，我们发起两次异步请求，但是设置expect.assertions(1)或expect.assertions(3)，这个时候就会报错，造成测试无法通过，因为实际异步请求返回应该2次。
所以需要expect.assertions()来确保异步请求响应次数与我们的期望一致。
```
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

在项目中，一个模块的方法内常常会去调用另外一个模块的方法。在单元测试中，我们可能并不需要关心内部调用的方法的执行过程和结果，
只想知道它是否被正确调用即可，甚至会指定该函数的返回值。此时，使用Mock函数是十分有必要。

Mock函数提供的以下三种特性，在我们写测试代码时十分有用：

- 捕获函数调用情况
- 设置函数返回值
- 改变函数的内部实现

Mock函数常用方法：
1. mockFn.mockName(value)：设置mock函数的名字
2. mockFn.getMockName()： 返回mockFn.mockName(value)中设置的名字
3. mockFn.mock.calls：mock函数的调用信息

mockFn.mock.calls返回一个数组，数组中的每一个元素又是一个数组，包含mock函数的调用信息。比如，
一个被调用两次的模拟函数f，参数为f('arg1'，'arg2')，然后使用参数f('arg3'，'arg4')，mockFn.mock.calls返回的数组形式如下：

> [['arg1', 'arg2'], ['arg3', 'arg4']]

因此，mockFn.mock.calls.length代表mock函数被调用次数，mockFn.mock.calls[0][0]代表第一次调用传入的第一个参数，以此类推。

4. mockFn.mock.results：mock函数的return值，以数组存储
5. mockFn.mock.instances：mock函数实例

```
const mockFn = jest.fn();

const a = new mockFn();
const b = new mockFn();

mockFn.mock.instances[0] === a; // true
mockFn.mock.instances[1] === b; // true
```

6. mockFn.mockImplementation(fn)：创建一个mock函数

注意：jest.fn(implementation)是jest.fn().mockImplementation(implementation)的简写。

7. mockFn.mockImplementationOnce(fn)：创建一个mock函数

该函数将用作对mocked函数的一次调用的mock的实现。可以链式调用，以便多个函数调用产生不同的结果。

```
const myMockFn = jest
  .fn()
  .mockImplementationOnce(cb => cb(null, true))
  .mockImplementationOnce(cb => cb(null, false));

myMockFn((err, val) => console.log(val)); // true

myMockFn((err, val) => console.log(val)); // false
```

当mocked函数用完使用mockImplementationOnce定义的实现时，如果调用它们，
它将使用jest.fn(()=> defaultValue)或.mockImplementation(()=> defaultValue)执行默认实现集：
```
const myMockFn = jest
  .fn(() => 'default')
  .mockImplementationOnce(() => 'first call')
  .mockImplementationOnce(() => 'second call');

// 'first call', 'second call', 'default', 'default'
console.log(myMockFn(), myMockFn(), myMockFn(), myMockFn());
```

8. mockFn.mockReturnThis()：jest.fn()的语法糖

```
jest.fn(function() {
  return this;
});
```

9. mockFn.mockReturnValue(value)：接受一个值作为调用mock函数时的返回值

```
const mock = jest.fn();
mock.mockReturnValue(42);
mock(); // 42
mock.mockReturnValue(43);
mock(); // 43
```

10. mockFn.mockReturnValueOnce(value)：接受一个值作为调用mock函数时的返回值，可以链式调用，以便产生不同的结果。

当不再使用mockReturnValueOnce值时，调用将返回mockReturnValue指定的值。

```
const myMockFn = jest
  .fn()
  .mockReturnValue('default')
  .mockReturnValueOnce('first call')
  .mockReturnValueOnce('second call');

// 'first call', 'second call', 'default', 'default'
console.log(myMockFn(), myMockFn(), myMockFn(), myMockFn());
```

11. mockFn.mockResolvedValue(value)：mock异步函数的语法糖

实现上类似于

```
jest.fn().mockImplementation(() => Promise.resolve(value));
```

用于在test中模拟异步函数

```
test('async test', async () => {
  const asyncMock = jest.fn().mockResolvedValue(43);

  await asyncMock(); // 43
});
```

12. mockFn.mockResolvedValueOnce(value)：语法糖

实现上类似于

```
jest.fn().mockImplementationOnce(() => Promise.resolve(value));
```

```
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

13. mockFn.mockRejectedValue(value)：语法糖

实现上类似于

```
jest.fn().mockImplementation(() => Promise.reject(value));
```

```
test('async test', async () => {
  const asyncMock = jest.fn().mockRejectedValue(new Error('Async error'));

  await asyncMock(); // throws "Async error"
});
```

14. mockFn.mockRejectedValueOnce(value)：语法糖

实现上类似于

```
jest.fn().mockImplementationOnce(() => Promise.reject(value));
```

```
test('async test', async () => {
  const asyncMock = jest
    .fn()
    .mockResolvedValueOnce('first call')
    .mockRejectedValueOnce(new Error('Async error'));

  await asyncMock(); // first call
  await asyncMock(); // throws "Async error"
});
```

15. mockFn.mockClear()：重置所有存储在mockFn.mock.calls 和 mockFn.mock.instances数组中的信息

当你想要清除两个断言之间的模拟使用数据时，这通常很有用。

16. mockFn.mockReset()：完成mockFn.mockClear()所做的所有事情，还删除任何模拟的返回值或实现

当你想要将模拟完全重置回其初始状态时，这非常有用。（请注意，重置spy将导致函数没有返回值）。

17. mockFn.mockRestore()：完成mockFn.mockReset()所做的所有事情，并恢复原始（非模拟）实现

当你想在某些测试用例中模拟函数并在其他测试用例中恢复原始实现时，这非常有用。

## jest.mock and jest.spyOn

jest对象上有fn,mock,spyOn三个方法，在实际项目的单元测试中，jest.fn()常被用来进行某些有回调函数的测试；jest.mock()可以mock整个模块中的方法，
当某个模块已经被单元测试100%覆盖时，使用jest.mock()去mock该模块，节约测试时间和测试的冗余度是十分必要；
当需要测试某些必须被完整执行的方法时，常常需要使用jest.spyOn()。

如Example.8中所示，event.js中调用了fetch.js的方法，fetch.js文件夹中封装的请求方法可能我们在其他模块被调用的时候，
并不需要进行实际的请求（请求方法已经通过单侧或需要该方法返回非真实数据）。此时，使用jest.mock(）去mock整个模块是十分有必要的。

```
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

jest.spyOn()方法同样创建一个mock函数，但是该mock函数不仅能够捕获函数的调用情况，还可以正常的执行被spy的函数。
实际上，jest.spyOn()是jest.fn()的语法糖，它创建了一个和被spy的函数具有相同内部代码的mock函数。

event.test.js是jest.mock()的示例代码，从控制台中可以看到console.log('fetchPostsList be called!');这行代码并没有被打印，
这是因为通过jest.mock()后，模块内的方法是不会被jest所实际执行的。这时我们就需要使用jest.spyOn()。

```
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

执行测试之后，可以看到控制台中的打印信息，说明通过jest.spyOn()，fetchPostsList被正常的执行了。

> TIPS：在jest中如果想捕获函数的调用情况，则该函数必须被mock或者spyOn！

## Timer Mock

jest可以模拟定时器从而允许自主控制时间流逝。模拟定时器运行可以方便测试，比如不必等待一个很长的延时而是直接获取结果。

jest对象上与timer mock相关的方法主要有以下个：

1. jest.useFakeTimers()：指示Jest使用标准计时器函数的假版本（setTimeout，setInterval，clearTimeout，clearInterval，nextTick，setImmediate和clearImmediate）。
2. jest.useRealTimers()：指示Jest使用标准计时器功能的真实版本。
3. jest.clearAllTimers()：从计时器系统中清除任何等待的计时器。
4. jest.runAllTicks()：执行微任务队列中的所有任务（通常通过process.nextTick在节点中连接）。
5. jest.runAllTimers()：执行宏任务队列中的所有任务。
6. jest.runAllImmediates()：通过setImmediate()执行任务队列中的所有任务。
7. jest.advanceTimersByTime(n)：执行宏任务队列中的所有任务，当该API被调用时，所有定时器被提前 n 秒。
8. jest.runOnlyPendingTimers()：仅执行宏任务中当前正在等待的任务。

举例来说(见Example.5):

```
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

我们在timer.js中设定了一个1s钟之后才会执行的定时器，但是测试代码是同步执行，通过timer mock，我们不必等待定时器执行完成就可以完成测试。

```
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

在上面的代码中，我们已经在第四行声明使用假时间，在test块中，虽然setTimeout尚未执行完毕，但是测试已经完成，
setTimeout执行一次，没有setInterval执行，这与期望一致。接下来调用jest.runAllTimers()使得所有定时器立即执行完毕，控制台打印定时器中的输出。

对于递归定时器的情况，如果使用jest.runAllTimers()，所有的定时器就无限循环了，这个时候就需要用到jest.runOnlyPendingTimers()了，
因为runOnlyPendingTimers的过程中不会产生新的定时器，从而避免了无限循环的问题，如Example.5 中pendingTimer所示。

jest.advanceTimersByTime(n)也很容易理解，就是将所有定时器提前n秒执行。如下所示，在未调用jest.advanceTimersByTime(n)之前，callback还没有被调用，
然后通过jest.advanceTimersByTime(1000)让定时器提前1s执行，因此接下来的断言不会报错。

```
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

同样的，如果使用jest.advanceTimersByTime(500)提前0.5s，上面的测试可以进行如下修改。

```
jest.advanceTimersByTime(500);

expect(callback).not.toBeCalled();
expect(callback).toHaveBeenCalledTimes(0);
```

## Manual Mock

Manual Mock用于存储模拟数据的功能。例如，您可能希望创建一个允许您使用虚假数据的手动模拟，而不是访问网站或数据库等远程资源。
这可以确保您的测试快速且稳定。

通过在紧邻模块的__mocks __ 子目录中编写模块来定义手动模拟。例如，要在models目录中模拟一个名为user的模块(如Example.9 所示)，
请创建一个名为user.js的文件并将其放在models / __ mocks__目录中。同时请注意__mocks__文件夹区分大小写。

** 注意比较Example.8 和 Example.9 中jest.mock的不同，由于Example.8中没有__mocks__文件夹，所以mock并不会真正运行，而在Example.9中会使用__mocks__下的同名文件进行替换。

```
// userMocked.test.js
import user from './models/user';

jest.mock('./models/user');

test('if user model is mocked', () => {
    expect(user.getAuthenticated()).toEqual({age: 622, name: 'Mock name'});
});
```

如果您正在模拟的模块是Node模块（例如：lodash），则mock应放在与node_modules相邻的__mocks__目录中，并将自动模拟。没有必要显式调用jest.mock（'module_name'）。
但是如果想模拟Node的核心模块（例如：fs或path），那么明确地调用例如： jest.mock（'path'）是必需的，因为默认情况下不会模拟核心Node模块。

如果您正在使用ES6模块导入，那么您通常倾向于将导入语句放在测试文件的顶部。但是通常你需要指示Jest在模块使用它之前使用模拟。
出于这个原因，Jest会自动将jest.mock调用提升到模块的顶部（在任何导入之前）。

## ES6 Class Mock

如Example.6中所示，分别有SoundPlayer类和使用该类的SoundPlayerConsumer类。我们将在SoundPlayerConsumer的测试中模拟SoundPlayer。

```
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

```
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

分别有4种方法可以创建ES6类的模拟：

1. Automatic mock

使用该方法，类中的所有方法调用状态保存在theAutomaticMock.mock.instances[index].methodName.mock.calls中。

同时，如果您在类中使用了箭头函数，它们将不会成为模拟的一部分。因为箭头函数不存在于对象的原型上，它们只是包含对函数的引用的属性。

** 注：由于Example.6中同时存在Automatic mock和Manual mock，所以在使用该方法时需要把__mocks__文件夹改个名。

2. Manual mock

在相邻的__mocks__文件夹下生成同名文件从而进行替换模拟。

** 注意：使用manual mock时确保__mocks__文件夹命名正确。

3. Calling jest.mock() with the module factory parameter

使用此种方法就是在jest.mock()中添加一个参数：jest.mock(path，moduleFactory)接受moduleFactory参数，moduleFactory返回一个模拟的函数。
为了模拟构造函数，moduleFactory必须返回构造函数，换句话说，模块工厂必须是返回函数的函数。

4. Replacing the mock using mockImplementation() or mockImplementationOnce()

对jest.mock的调用会被提升到代码的顶部。 您通过在现有mock上调用mockImplementation()（或mockImplementationOnce()）而不是使用factory参数，
从而稍后指定模拟（例如，在beforeAll()），或者在测试之间更改模拟。

## Bypassing Module Mock

Jest允许您模拟测试中的整个模块，这有助于测试代码是否正确，函数调用是否正确。
但是，有时您可能希望在测试文件中使用模拟模块的一部分，在这种情况下，您希望访问原始实现，而不是模拟版本。
jest.requireActual()允许你导入实际的版本，而不是模拟的版本。

## 推荐阅读

[使用Jest测试JavaScript (入门篇)](https://segmentfault.com/a/1190000016232248)

[使用Jest测试JavaScript(Mock篇)](https://segmentfault.com/a/1190000016717356)

[顶级测试框架Jest指南：跑通一个完美的程序，就是教出一群像样的学生](https://segmentfault.com/a/1190000016399447)

[Jest cheat sheet](https://github.com/sapegin/jest-cheat-sheet)

[Jest官方demo](https://github.com/facebook/jest/tree/master/examples)



