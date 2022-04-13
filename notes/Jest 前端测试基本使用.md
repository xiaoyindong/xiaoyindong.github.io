## 1. 概述

测试用例首先要明确要测的到底是什么，比如下面的sum函数是计算两个参数的和。

```js
function sum(a, b) {
    return a + b;
}

module.exports = sum;
```

测试代码一般会写在一个单独的模块中，测试模块名称要和模块名称类似，比如模块加demo.js测试模块可以命名为demo.test.js。

测试规则很简单，被测的模块要和测试模块分文件存储，在测试模块中引入要测试的函数，给函数一个输入，定义预期的输出，检车函数是否返回了预期的输出结果。

```js
const { sum } = require('./demo');

const result = sum(1, 2);

const expected = 3;

if (result !== expected) {
    throw new Error(`sum(1, 2) 的结果应该是 ${expected}，但是现在是 ${result}`)
}
```

可以将上面这个测试过程封装成一个函数，希望这个函数的使用像中文一样，比如函数名称叫做expect就是期望的意思，期望sum运行的结果是3就希望这样来写。

```js
expect(sum(1, 2)).toBe(3);
```

这就像说话一样，可以按这样的方式去封装expect函数，这样就可以了。

```js
function expect(result) {
    return {
        toBe(expected) {
            if (result !== expected) {
                throw new Error(`期望的结果应该是 ${expected}，但是收到了 ${result}`)
            }
        }
    }
}
```

expect称之为断言函数，断定一个真实的结构是期望的结果。这里还可以加上一个测试的描述信息，让测试过程更清晰，比如增加一个test函数，接收两个参数，第一个是描述信息第二个是执行函数，一旦出问题需要把第一个参数的描述信息反馈出来。

```js
test('sum', () => {
    expect(sum(1, 2)).toBe(3);
})

function test(message, callback) {
    try {
        callback();
    } catch (err) {
        console.error(`${message}: ${err.message}`)
    }
}
```

## 2. jest测试框架

jest已经把上面我们封装的函数都封装好了，直接拿过来用就可以了。可以安装jest模块，然后通过jest命令运行就可以了。下面的代码直接就可以运行，不需要自己编写test和expect也不需要单独引入。jest会自动引入使用到的方法。可以使用@types/jest添加提示，否则并不知道如何编写。

```js
const { sum } = require('./demo');

test('sum', () => {
    expect(sum(1, 2)).toBe(3);
})
```
jest是Facebook出品的一个Javascript开源测试框架，相对其他测试框架，其一大特点是内置了常用的测试工具，比如零配置，自带断言，测试覆盖率，Mock模拟等功能，实现了开箱即用。

jest使用很多项目，比如Babel，TypeScript，Node，React，Angular，Vue等。

作为一个面向前端的测试框架，jest可以利用其特有的快照测试功能通过比对UI代码生成的快照文件，实现对React常见前端框架的自动测试。此外jest的测试用例是并行执行的，而且只执行发生改变的文件所对应的测试，这样一来效率就比较搞了。

## 2. 配置文件

默认情况下jest是零配置的，当然也支持通过配置文件自定义配置，首先需要生成配置文件。填写一些预设项就可以了，基本就是是否使用ts，是node还是浏览器，测试覆盖率，引擎建议选择babel因为比较稳定等等。

```s
npx jest --init
```

jest.config.js是jest所有的配置信息的列表，需要修改的可以直接放开注释进行修改就行了。

```js
export default {
  // All imported modules in your tests should be mocked automatically
  // automock: false,

  // Stop running tests after `n` failures
  // bail: 0,

  // The directory where jest should store its cached dependency information
  // cacheDirectory: "/private/var/folders/q8/81bhkkfn7d9czrdh5wk0k3m40000gn/T/jest_dx",

  // Automatically clear mock calls and instances between every test
  clearMocks: true,

  // Indicates whether the coverage information should be collected while executing the test
  // collectCoverage: false,

  // An array of glob patterns indicating a set of files for which coverage information should be collected
  // collectCoverageFrom: undefined,

  // The directory where jest should output its coverage files
  // coverageDirectory: undefined,

  // An array of regexp pattern strings used to skip coverage collection
  // coveragePathIgnorePatterns: [
  //   "/node_modules/"
  // ],

  // Indicates which provider should be used to instrument code for coverage
  // coverageProvider: "babel",

  // A list of reporter names that jest uses when writing coverage reports
  // coverageReporters: [
  //   "json",
  //   "text",
  //   "lcov",
  //   "clover"
  // ],

  // An object that configures minimum threshold enforcement for coverage results
  // coverageThreshold: undefined,

  // A path to a custom dependency extractor
  // dependencyExtractor: undefined,

  // Make calling deprecated APIs throw helpful error messages
  // errorOnDeprecated: false,

  // Force coverage collection from ignored files using an array of glob patterns
  // forceCoverageMatch: [],

  // A path to a module which exports an async function that is triggered once before all test suites
  // globalSetup: undefined,

  // A path to a module which exports an async function that is triggered once after all test suites
  // globalTeardown: undefined,

  // A set of global variables that need to be available in all test environments
  // globals: {},

  // The maximum amount of workers used to run your tests. Can be specified as % or a number. E.g. maxWorkers: 10% will use 10% of your CPU amount + 1 as the maximum worker number. maxWorkers: 2 will use a maximum of 2 workers.
  // maxWorkers: "50%",

  // An array of directory names to be searched recursively up from the requiring module's location
  // moduleDirectories: [
  //   "node_modules"
  // ],

  // An array of file extensions your modules use
  // moduleFileExtensions: [
  //   "js",
  //   "jsx",
  //   "ts",
  //   "tsx",
  //   "json",
  //   "node"
  // ],

  // A map from regular expressions to module names or to arrays of module names that allow to stub out resources with a single module
  // moduleNameMapper: {},

  // An array of regexp pattern strings, matched against all module paths before considered 'visible' to the module loader
  // modulePathIgnorePatterns: [],

  // Activates notifications for test results
  // notify: false,

  // An enum that specifies notification mode. Requires { notify: true }
  // notifyMode: "failure-change",

  // A preset that is used as a base for jest's configuration
  // preset: undefined,

  // Run tests from one or more projects
  // projects: undefined,

  // Use this configuration option to add custom reporters to jest
  // reporters: undefined,

  // Automatically reset mock state between every test
  // resetMocks: false,

  // Reset the module registry before running each individual test
  // resetModules: false,

  // A path to a custom resolver
  // resolver: undefined,

  // Automatically restore mock state between every test
  // restoreMocks: false,

  // The root directory that jest should scan for tests and modules within
  // rootDir: undefined,

  // A list of paths to directories that jest should use to search for files in
  // roots: [
  //   "<rootDir>"
  // ],

  // Allows you to use a custom runner instead of jest's default test runner
  // runner: "jest-runner",

  // The paths to modules that run some code to configure or set up the testing environment before each test
  // setupFiles: [],

  // A list of paths to modules that run some code to configure or set up the testing framework before each test
  // setupFilesAfterEnv: [],

  // The number of seconds after which a test is considered as slow and reported as such in the results.
  // slowTestThreshold: 5,

  // A list of paths to snapshot serializer modules jest should use for snapshot testing
  // snapshotSerializers: [],

  // The test environment that will be used for testing
  testEnvironment: "jsdom",

  // Options that will be passed to the testEnvironment
  // testEnvironmentOptions: {},

  // Adds a location field to test results
  // testLocationInResults: false,

  // The glob patterns jest uses to detect test files
  // testMatch: [
  //   "**/__tests__/**/*.[jt]s?(x)",
  //   "**/?(*.)+(spec|test).[tj]s?(x)"
  // ],

  // An array of regexp pattern strings that are matched against all test paths, matched tests are skipped
  // testPathIgnorePatterns: [
  //   "/node_modules/"
  // ],

  // The regexp pattern or array of patterns that jest uses to detect test files
  // testRegex: [],

  // This option allows the use of a custom results processor
  // testResultsProcessor: undefined,

  // This option allows use of a custom test runner
  // testRunner: "jest-circus/runner",

  // This option sets the URL for the jsdom environment. It is reflected in properties such as location.href
  // testURL: "http://localhost",

  // Setting this value to "fake" allows the use of fake timers for functions such as "setTimeout"
  // timers: "real",

  // A map from regular expressions to paths to transformers
  // transform: undefined,

  // An array of regexp pattern strings that are matched against all source file paths, matched files will skip transformation
  // transformIgnorePatterns: [
  //   "/node_modules/",
  //   "\\.pnp\\.[^\\/]+$"
  // ],

  // An array of regexp pattern strings that are matched against all modules before the module loader will automatically return a mock for them
  // unmockedModulePathPatterns: undefined,

  // Indicates whether each individual test should be reported during the run
  // verbose: undefined,

  // An array of regexp patterns that are matched against all source file paths before re-running tests in watch mode
  // watchPathIgnorePatterns: [],

  // Whether to use watchman for file crawling
  // watchman: true,
};
```

这里面的内容不需要都了解，用到的时候查一下，改一下就可以了。比如testMatch是匹配哪些测试文件，如果需要修改就放开修改就可以了。同样testPathIgnorePatterns是忽略哪些文件的测试。[文档在这(https://jestjs.io/docs/configuration)](https://jestjs.io/docs/configuration)。

当然除了配置文件还可以通过命令行的参数指定一些简单的命令，也就是jest cli选项。[文档在这(https://jestjs.io/docs/cli)](https://jestjs.io/docs/cli)。

```s
jest my-test #or
jest path/to/my-test.js
```

## 3. 监视模式

监视模式有两种方式，一种是--watchAll会在监视文件更改之后去重新运行所有的测试文件，即便其他文件没有改也会跑一遍。

```js
npx jest --watchAll
```

第二种是--watch，需要git支持，会监视git仓库中的文件更改，并重新运行与已更改文件相关的测试。

```js
npx jest --watch
```

监视模式使用测试时jest提供了一些辅助命令，这些命令在测试运行起来之后控制台也会有提示。按不同的按键会有不同的功能。比如敲击回车会重新跑一遍测试用例。按f只运行失败的测试，在这种模式下还要运行一下f才能退出看到其他的测试。按o只运行与更改文件相关的测试。按q会退出测试模式。按p会以文件名正则表达式的模式进行过滤，可以输入，并且是以绝对路径的方式进行检测，如果文件夹与名称匹配会将所有文件作为测试运行。按t以测试名称正则表达式进行测试，就是过滤test函数中的第一个参数字符串。

## 4. 使用ES6模块

如果想要在jest测试模块中使用ES6需要额外配置，首先需要安装@babel/jest，@babel/core，@babel/preset-env。

```s
npm install -D @babel/jest @babel/core @babel/preset-env
```

babel.config.js

```js
module.exports = {
    preset: [
        ['@babel/preset-env'], 
        {
            targets: { node: 'current' }
        }
    ]
}
```

jest在运行测试测时候会自动找到Babel将ES6代码转换为ES5执行。jest结合Babel的运行原理，运行测试之前，结合Babel先把代码做一次转换，模块被转换为CommonJS后再运行转换后的测试用例代码。

```js
import { sum } from './demo';
test('sum', () => {
    expect(sum(1, 2)).toBe(3);
})
```

## 5. 全局API

在测试文件中jest会将所有方法和对象放入全局环境中，无需要求或导入任何内容即可使用，当然如果喜欢显式导入也是可以的。

```js
import { describe, expect, test } from 'jest';
```

### 1. test

test是测试用例，每个测试文件都至少要有一个测试用例，函数别名是it，接收三个参数it(name, fn, timeout);

```js
it('global-api test', () => {
    console.log('global api');
})

it.only('global-api test', () => { // 仅运行这一个，其他的都不运行，当多个用例在一个文件中时

})
```

### 2. expect

测试用例中同样需要检查值是否满足某些条件，expect可以访问多个匹配器验证不同的内容。

```js
test('two plug two is four', () => {
    expect(2 + 2).toBe(4);

    expect({ name: 'jack' }).toEqual({ name: 'jack' });

    expect('Christoph').toMatch(/stop/);

    expect(4).toBeGreaterThan(3);

    expect(4).toBeGreaterThan(5);
})
```

### 3. describe

创建一个将几个相关测试组合在一起的块，可以简单的理解为分组。

```js
const myBeverage = {
    deliciout: true,
    sour: false,
}

describe('my beverage', () => {
    it('is deliciout', () => {
        expect(myBeverage.deliciout)
    })

    it('is not sour', () => {
        expect(myBeverage.sour)
    })
})
```

### 4. 生命周期

afterAll(fb, timeout)，afterEach(fb, timeout)， beforeAll(fb, timeout)，beforeEach(fb, timeout)。在测试用例之前或之后执行。

### 5. jest对象

jest对象自动位于每个测试文件中的范围内，jest对象中的方法有助于创建模拟，并让您控制jest的整体行为，也可以直接导入，提供了很多的辅助工具方法。

```js
import { jest } from '@jest/globales';

jest.autoMockOn();

jest.useFakeTimers();
```

## 6. 常用匹配器

这里介绍一下断言函数的常用匹配器介绍比如toBe，toEquals等。

### 1. toBe

表示完全的相等匹配，可以匹配基本数据类型不能匹配对象。

```js
it('test', () => {
    expect(2).toBe(2);
    expect('Hello').toBe('Hello');
})
```

### 2. 判断null, undefined

```js
it('test', () => {
    const n = null;
    expect(n).toBeNull(); // 是否为null
    expect(n).not.toBeUndefined(); // 是不为undefined
    expect(n).toBeDefined(); // 是否定义了
    expect(n).not.toBeTruthy(); // 不是false
    expect(n).toBeFalsy(); // 不是false
})
```

### 3. Numbers

数字相关的

```js
test('two plus two', () => {
  const value = 2 + 2;
  expect(value).toBeGreaterThan(3); // 大于3
  expect(value).toBeGreaterThanOrEqual(3.5); // 大于等于3.5
  expect(value).toBeLessThan(5); // 小于5
  expect(value).toBeLessThanOrEqual(4.5); // 小于等于 4.5 

  // 数字的相等可以用toBe 也可以用 toEqual
  expect(value).toBe(4); // 
  expect(value).toEqual(4); // 
});
```

javascript中数字是双精度的，并不是准确的，小数需要使用toBeCloseTo进行判断。

```js
test('adding floating point numbers', () => {
  const value = 0.1 + 0.2;
  expect(value).toBeCloseTo(0.3);
});
```

### 4. Strings

字符串可以用toMatch匹配一个正则表达式。

```js
test('there is no I in team', () => {
  expect('team').not.toMatch(/I/);
});

test('but there is a "stop" in Christoph', () => {
  expect('Christoph').toMatch(/stop/);
});
```

### 5. Arrays and iterables

数组可以用toContain判断是否包含一个元素。

```js
const shoppingList = [
  'diapers',
  'kleenex',
  'trash bags',
  'paper towels',
  'milk',
];

test('the shopping list has milk on it', () => {
  expect(shoppingList).toContain('milk');
  expect(new Set(shoppingList)).toContain('milk');
  expect(shoppingList.length).toBe(5);
});
```

### 6. Exceptions

判断异常

```js
function compileAndroidCode() {
  throw new Error('you are using the wrong JDK');
}

test('compiling android goes as expected', () => {
  expect(() => compileAndroidCode()).toThrow();
  expect(() => compileAndroidCode()).toThrow(Error);

  // 也可以通过字符串或者正则匹配异常的message
  expect(() => compileAndroidCode()).toThrow('you are using the wrong JDK');
  expect(() => compileAndroidCode()).toThrow(/JDK/);
});
```

[详细文档(https://jestjs.io/docs/expect)](https://jestjs.io/docs/expect)

## 7. 异步测试

测试异步任务也比较简单，test执行函数中可以接收一个done的函数参数，在异步操作调用结束之后执行done函数。

```js
function afn (callback) {
    setTimeout(() => {
        callback({ foo: 'bar' });
    }, 2000)
}

it('async', (done) => {
    afn(data => {
        done();
        expect(data).toEqual({ foo: 'bar' })
    })
})
```

对于Promise来说就是在then回调中执行就好了。甚至也不需要使用done函数。

```js
function afn () {
    return Promise((resolve, reject) => {
        setTimeout(() => {
            resolve({ foo: 'bar' });
        }, 2000)
    })
    
}

it('async', () => {
    // afn().then(data => {
    //     done();
    //     expect(data).toEqual({ foo: 'bar' })
    // })
    return afn().then(data => {
        expect(data).toEqual({ foo: 'bar' })
    })
})
```

对于Promise来说还可以使用resolves或rejects。

```js
it('async', () => {
    return expect(afn()).resolves.toEqual({ foo: 'bar' });
})
```

async/await也是支持的。

```js
it('async', async () => {
    const data = await afn();
    expect(data).toEqual({ foo: 'bar' });
})
```

await与resolves组合也是可以的。

```js
test('the data is peanut butter', async () => {
  await expect(fetchData()).resolves.toBe('peanut butter');
});
```

## 8. 定时器

上面虽然可以解决异步测试问题，但是如果定时器时间特别长，这么一直等肯定是不理想的，所以可以通过Timer Mocks来控制指定时间运行，注意这种操作是不能用await方式的。可以使用useFakeTimers()来mock定时器，通过runAllTimers快进所有定时器结束，无论定时器设置多长时间立刻就会执行。

```js
function afn () {
    return Promise((resolve, reject) => {
        setTimeout(() => {
            resolve({ foo: 'bar' });
        }, 2000)
    })
}

// mock定时器
jest.useFakeTimers();

it('async', () => {
    // 在这个用例中至少要有一次断言，保证书写忘记的
    expect.assertions(1);
    afn().then(data => {
        expect(data).toEqual({ foo: 'bar' })
    })
    // 快进所有定时器结束，无论定时器设置多长时间立刻就会执行
    jest.runAllTimers();
})
```

有时候可能出现循环定时器，针对这种情况就需要使用runOnlyPendingTimers，意思是快进当前进行的定时器，不等待其他的定时器、

```js
function afn () {
    return Promise((resolve, reject) => {
        setTimeout(() => {
            resolve({ foo: 'bar' });
            // 循环一次定时器
            afn();
        }, 2000)
    })
}

// mock定时器
jest.useFakeTimers();

it('async', () => {
    // 在这个用例中至少要有一次断言，保证书写忘记的
    expect.assertions(1);
    afn().then(data => {
        expect(data).toEqual({ foo: 'bar' })
    })
    // 快进当前进行的定时器，不等待其他的定时器
    jest.runOnlyPendingTimers();
})
```

还可以通过advanceTimersByTime快进指定的时间。

```js
function afn () {
    return Promise((resolve, reject) => {
        setTimeout(() => {
            resolve({ foo: 'bar' });
        }, 2000)
    })
}

// mock定时器
jest.useFakeTimers();

it('async', () => {
    // 在这个用例中至少要有一次断言，保证书写忘记的
    expect.assertions(1);
    afn().then(data => {
        expect(data).toEqual({ foo: 'bar' })
    })
    // 将定时器快进1s
    jest.advanceTimersByTime(1000);
    // 在快进1s
    jest.advanceTimersByTime(1000);
})
```

## 9. mock

这里有一个forEach的实现代码，接收一个数组和一个回调函数，然后将遍历到了每一项传入函数中。想要测试forEach是需要一个数组的，还需要一个函数。

```js
function forEach(items, callback) {
  for (let index = 0; index < items.length; index++) {
    callback(items[index]);
  }
}
```

可以创建一个fn。

```js
it('mock', () => {
    const items = [1, 2, 3];
    const mockFn = jest.fn();
    forEach(items, mockFn);
    expect(mockFn.mock.calls.length).toBe(items.length);
})
```

当做异步请求操作的时候不可能真的等到请求结束再去执行测试用例，一般都是通过mock数据来执行，通过jest.mock('axios')将axios请求库进行mock，这样就不会真的发请求。

```js
const users = () => {
    return axios.get('/user.json').then(resp => resp.data)
}
```

```js
import axios from 'axios';
import users from './users;

jest.mock('axios');
it('fetch', () => {
    const ret = [{name: 'Bob'}];
    const resp = { data: ret };
    // 拦截get
    axios.get.mockResolvedValue(resp);
    const data = await users();
     expect(data).toEqual(ret)
})
```

Mock Implementations是模拟实现，可以模拟一个模块，假设有一个模块特别复杂，我们不想去使用，可以直接mock这个模块，就像上面的axios一样。

```js
jest.mock('./foo');
import foo from './foo';
```

mock之后再加载的foo就变成mock的foo了。接着可以mockImplementation实现这个foo，这个时候再调用foo就是返回123了。

```js
jest.mock('./foo');
import foo from './foo';

foo.mockImplementation(() => {
    return 123;
})
```

也可以设置每次返回不同的值。

```js
const myMockFn = jest.fn().mockImplementationOnce(cb => cb(null, true)).mockImplementationOnce(cb => cb(null, false));
```

可以修改报错时候的名称mockName。

```js
const myMockFn = jest.fn().mockReturnValue('default').mockImplementation(scalar => 42 + scalar).mockName('add42');
```

还有一些匹配器

```js
// 函数是否被调用
expect(mockFunc).toHaveBeenCalled();

// 函数被调用的参数是什么
expect(mockFunc).toHaveBeenCalledWith(arg1, arg2);

// 最后一次被调用的时候参数是什么
expect(mockFunc).toHaveBeenLastCalledWith(arg1, arg2);

// 快照测试
expect(mockFunc).toMatchSnapshot();
```

## 9. 钩子函数

比如下面一段代码，第一个测试用例修改了原始值，第二个测试用例并不知道，仍然判断原有的值，这就会有问题。

```js
const user = {
    foo: 'bar'
}

it('test1', () => {
    user.foo = 'baz';
    expect(user.foo).toBe('baz');
})

it('test2', () => {
    expect(user.foo).toBe('bar');
})
```

为了解决这样的问题可以使用Object.assign对对象进行copy。

```js
const data = {
    foo: 'bar'
}
const user = null;

it('test1', () => {
    user = Object.assign({}, data);
    user.foo = 'baz';
    expect(user.foo).toBe('baz');
})

it('test2', () => {
    user = Object.assign({}, data);
    expect(user.foo).toBe('bar');
})
```

可以使用beforeEach在每个测试用例之前进行copy工作，这就是钩子函数的作用。

```js
const data = {
    foo: 'bar'
}
const user = null;

beforeEach(() => {
    user = Object.assign({}, data);
})

it('test1', () => {
    // user = Object.assign({}, data);
    user.foo = 'baz';
    expect(user.foo).toBe('baz');
})

it('test2', () => {
    // user = Object.assign({}, data);
    expect(user.foo).toBe('bar');
})
```

除了beforeEach还有afterAll(fb, timeout)，afterEach(fb, timeout)， beforeAll(fb, timeout)。afterAll会在所有测试用例执行之后运行一次，beforeAll会在所有测试用例执行之前运行一次。afterEach会在每个测试用例运行之后执行一次。

可以设置钩子函数的作用域，只需要将钩子函数和测试用例包裹在describe中就可以了。

```js
// 所有的test都会使用这个beforeEach
beforeEach(() => {
  return initializeCityDatabase();
});

test('city database has Vienna', () => {
  expect(isCity('Vienna')).toBeTruthy();
});

test('city database has San Juan', () => {
  expect(isCity('San Juan')).toBeTruthy();
});

describe('matching cities to foods', () => {
  // 之后下面两个test才会使用这个beforeEach
  beforeEach(() => {
    return initializeFoodDatabase();
  });

  test('Vienna <3 veal', () => {
    expect(isValidCityFoodPair('Vienna', 'Wiener Schnitzel')).toBe(true);
  });

  test('San Juan <3 plantains', () => {
    expect(isValidCityFoodPair('San Juan', 'Mofongo')).toBe(true);
  });
});
```

## 10. DOM测试

jest里面用到了一个第三方的jsdom包，可以用来直接操作dom，里面模拟了浏览器环境的domAPI，jest里面已经安装好了，不需要单独安装，直接使用就可以了。

```js
const div = document.createElement('div');
div.innerHTML = `<h1>Hello World</h1>`;
document.body.appendChild(div);

it('DOM', () => {
    // 可以直接使用document
    console.log(document.body.innerHTML);
    // 测试
    expect(document.querySelector('h1').innerHTML).toBe('Hello World');
})
```

有了dom测试就可以很方便的测试前端组件了，比如测试vue组件。

```js
import Vue from 'vue/dist/vue';

function renderVueComponent() {
    document.body.innerHTML = `<div id="app"></div>`;
    new Vue({
        template: `<div id="app"><h1>{{ message }}</h1></div>`,
        data: () => {
            return {
                message: 'Hello World'
            }
        }
    }).$mount('#app')
}

it('Vue', () => {
    renderVueComponent();
    expect(document.body.innerHTML).toMatch(/Hello World/);
})
```

## 11. 快照测试

快照测试指的就是缓存渲染的DOM结构，在修改的时候提示开发者。通过toMatchSnapshot方法生成快照文件，也就是DOM的字符串。这个API在第一次运行的时候会生成并保存页面DOM结构，在后面运行的时候会和前一次进行对比。

```js
import Vue from 'vue/dist/vue';

function renderVueComponent() {
    document.body.innerHTML = `<div id="app"></div>`;
    new Vue({
        template: `<div id="app"><h1>{{ message }}</h1></div>`,
        data: () => {
            return {
                message: 'Hello World'
            }
        }
    }).$mount('#app')
}

it('snapshot', () => {
    renderVueComponent();
    // 第一次运行的时候会 生成快照 - DOM字符串
    // 下一次运行测试的时候会自动进行比对
    expect(document.body.innerHTML).toMatchSnapshot();
})
```

如果修改了dom会导致测试执行失败，如果需要修改可以使用updateSnapshot更新快照。

```s
npx jest --updateSnapshot
```

这样快照就会更新了。
