# jest + enzyme构建react单元测试

[tag]:jest|enzyme|react|单元测试
[create]:2020-07-14

[代码](../demos/jest-react)

## 环境搭建

安装依赖`npm install --save-dev jest enzyme enzyme-adapter-react-16 react-test-renderer babel-jest identity-obj-proxy`。每个人项目内依赖不一样可能会有缺失的依赖，运行的时候缺什么补什么就可以了。

在项目根目录下新建test文件夹。用来存放测试相关的文件。

新建/test/setup.js
```javascript
// 在运行测试案例代码之前，Jest会先运行这里的配置文件来初始化指定的测试环境
import Enzyme from 'enzyme';
import Adapter from 'enzyme-adapter-react-16';

Enzyme.configure({ adapter: new Adapter() });
```

根目录下新建测试配置文件`test.config.js`
```javascript
module.exports = {
  setupFiles: [ // 配置文件，在运行测试案例代码之前，Jest会先运行这里的配置文件来初始化指定的测试环境
    './test/setup.js',
  ],
  moduleFileExtensions: ['js', 'jsx', 'ts', 'tsx'], // 代表支持加载的文件名
  testPathIgnorePatterns: ['/node_modules/'], // 用正则来匹配不用测试的文件
  testRegex: '.*\\.test\\.js$', // 正则表示的测试文件，测试文件的格式为xxx.test.js
  collectCoverage: false, // 是否生成测试覆盖报告，如果开启，会增加测试的时间
  collectCoverageFrom: [ // 生成测试覆盖报告时检测的覆盖文件
    '<rootDir>/src/components/**/*.tsx',
  ],
  moduleNameMapper: { // 代表需要被Mock的资源名称
    "\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$": "<rootDir>/__mocks__/fileMock.js",
    "\\.(css|less)$": "identity-obj-proxy"
  },
  transform: {
    "^.+\\.[jt]sx?$": "babel-jest" // 用babel-jest来编译文件，生成ES6/7的语法
  },
};
```
这里需要注意的是项目中用到的静态资源文件和css文件都需要被mock，在`moduleNameMapper`属性里定义了其mock的逻辑。

具体[配置](https://jestjs.io/docs/zh-Hans/configuration.html#modulenamemapper-objectstring-string--arraystring)

这里对静态资源做了mock，所以需要声明`/__mocks__/fileMock.js`
```javascript
module.exports = 'test-file-stub';
```

然后开始写测试逻辑。为了做测试，先写了一个todoList的组件。
![todolist](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/20200714172705.jpg!trans_webp)

todolist组件主要包含两个自组件，一个就是整个的List，还有Item

## 编写Item的测试

新建`/test/item.test.js`
```javascript
import React from 'react'; // jsx语法需要引入react
import Item from '../src/components/item'; // item组件
import renderer from 'react-test-renderer'; // 渲染react组件

const props = { // item需要的props
  name: 'test name',
  complete: false,
  index: 0,
  toggleStatus: jest.fn(),
  deleteItem: jest.fn(),
};

it('item renders correctly', () => { // 生成快照，之后每次组件更新都会跟快照diff出变化 更新快照文件使用npm run test -- -u
  const tree = renderer.create(
    <Item {...props} />
  ).toJSON();
  expect(tree).toMatchSnapshot(); // 生成快照
});
```
由于item是一个非常简单展示用组件，所以这里仅仅只做了渲染是否正确的验证。

toMatchSnapshot方法，会在同级目录下生成一个__snapshots文件夹用来存放快照文件，以后每次测试的时候都会和第一次生成的快照进行比较

然后给package.json的scripts加一个`“test”: "test": "jest --config test.config.js"`。在终端输入`npm run test`就可以跑起来了。

![跑item.test.js](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/20200714174247.jpg!trans_webp)

## 编写List组件的测试
List组件相比于item组件要复杂一点，所以将整个list的验证拆分为几部分完成

1. 新建`/test/list.test.js`
```javascript
import React from 'react';
import List from '../src/components/list';
import renderer from 'react-test-renderer';
import { mount } from 'enzyme';
describe('list 组件测试', () => { // 通过 describe 块来将测试分组。主要用于有before 和 after 的块时，当 before 和 after 的块在 describe 块内部时，则其只适用于该 describe 块内的测试。
  const wrapper = mount(<List />); // shallow：浅渲染，render：静态渲染，mount：完全渲染 （关于这三种渲染的区别请看文末文章“使用Jest进行React单元测试”）
  const input = wrapper.find('input').at(0); // 查找是否存在input元素
  const button = wrapper.find('button').at(0); // 查找是否存在button元素
  it('初始渲染逻辑操作', () => {
    expect(input.exists()); // input存在
    expect(button.exists()); // button存在
  });
});
```

2. 输入验证
```javascript
  it('输入操作', () => {
    input.simulate('change', { // 输入
      target: {
        value: 'test value',
      }
    });
    expect(wrapper.instance().inputValue).toBe('test value'); // 判断输入是否正确
  });
```
simulate(event, mock)：模拟事件，用来触发事件，event为事件名称，mock为一个event object

instance()：返回组件的实例

3. 点击逻辑验证
```javascript
  it('添加操作', () => {
    button.simulate('click'); // 点击
    expect(wrapper.find('Item').length).toBe(1); // 有一个Item组件
    expect(wrapper.state('list').length).toBe(1); // 数组里面有一项
  });
```

4. 删除操作验证
```javascript
  it('删除操作', () => {
    wrapper.find('.delete').at(0).simulate('click');
    expect(wrapper.state('list').length).toBe(0); // 数组是空的
  });
```

5. 测试组件内部函数调用
```javascript
  it('测试组件内部函数调用', () => {
    const testSpy = jest.spyOn(wrapper.instance(), 'test');
    const result = wrapper.instance().test(1, 2);
    expect(testSpy).toHaveBeenCalled(); // 是否已经被调用过
    expect(result).toBe(1);
    testSpy.mockRestore();
  });
```
jest.spyOn: 创建类似于jest.fn的模拟函数，但也跟踪对object [methodName]的调用(需要注意的是，spyOn需要在目标函数调用前调用。而且在验证完成之后需要将spy函数mockRestore,不然这个spy会一直存在，并且无法对相同的方法再次进行spy)

6. 验证声明周期
```javascript
  const componentDidMountSpy = jest.spyOn(List.prototype, 'componentDidMount'); // spy componentDidMount函数要放在渲染组件之前(因为渲染了之后函数就已经被调用过了)
  // 注意componentDidMountSpy的声明赋值需要在mount(<List />)前
  // 。。。。。。
  it('测试生命周期是否被调用', () => {
    expect(componentDidMountSpy).toHaveBeenCalled();
    componentDidMountSpy.mockRestore();
  });
```
其实生命周期的调用与测试组件内部函数的调用是一样的，只是生命周期函数特殊一些

对List的验证暂时就这么多。实际开发中肯定有很多场景是需要特殊定制的，这只能翻翻文档或者谷歌一下了。

### 异步验证
在日常开发中，还有一个大头就是异步请求。关于这部分的验证单独拎出来。

假设我们有一个获取用户名的请求函数, 在`/src/utils/api.ts`
```javascript
/** 一个假的api请求方法 */
export function getUserName(id: number) {
  return new Promise((resolve) => {
    setTimeout(() => { // 没有接口，这里假装就是接口吧
      resolve({
        name: 'mock name',
        id: id,
      });
    }, 1000);
  });
}
```

我们希望能对api进行一些测试，同时对请求回来之后UI更新是否正确做一个测试。

但是日常开发中，肯定不能直接拿真实请求来测试，因为真实请求慢且不稳定，容易受网络影响。而且可能会产生很多脏数据

所以我们需要用mock的方式，模拟一个指定输入得到指定输出的mock请求方法。

首先我们需要在api.ts的同级目录新建一个`__mocks__`文件夹

在`__mocks__`文件夹内新建同名文件`api.ts`
```javascript
// /src/utils/__mocks__/api.ts 注意这里后缀名要跟需要mock的文件的后缀名一样，之前踩到api.js后缀名不一样的坑
export function getUserName(id) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({
        name: 'test name',
        id: id,
      });
    }, 10);
  });
}
```

新建`/test/async.test.js`
```javascript
jest.mock('../src/utils/api'); // 这段mock需要放在脚本import前

import { getUserName } from '../src/utils/api';

describe('异步函数测试', () => {
  it('异步测试2', () => { // promise写法
    return expect(getUserName(1020)).resolves.toEqual({ name: 'test name', id: 1020 }); // 可以看到这里的name已经是mock函数里的name了
    /** 也可以采用如下这种写法 */
    // return getUserName(1020).then(res => {
    //   expect(res).toEqual({ name: 'test name', id: 1020 });
    // });
  });

  it ('async / await', async () => { // async语法函数写法
    expect.assertions(1); // 有几个断言
    const res = await getUserName(1010);
    expect(res).toEqual({ name: 'test name', id: 1010 });
  });
});
```

异步接口与UI结合
```javascript
  it('异步函数与UI结合', () => {
    const wrapper = mount(<List />);
    return wrapper.instance().getUserInfo(10002).then(res => {
      expect(res.name).toEqual('test name');
      expect(wrapper.find('.username').text()).toEqual(res.name);
    });
  });
```

## 测试覆盖率
jest的测试覆盖率的配置十分简单，只需要在test.config.js里面把`collectCoverage`设置为true, 同时设置`collectCoverageFrom`包含需要测试的文件。

然后在package.json的scripts脚本加上`--coverage`

如：`jest --colors --coverage --config test.config.js`

`--colors`属性可以让覆盖率对于不同的覆盖率用颜色标明

最终运行代码就会生成如下的覆盖率报告了。

![覆盖率报告](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/20200715104254.jpg!trans_webp)

THE END.

## 参考资料：

[使用Jest进行React单元测试](https://juejin.im/post/5b6c39bde51d45195c079d62#heading-28)
[Jest & enzyme 进行react单元测试](https://juejin.im/post/5c417aa4f265da616a47eb4d)
[jest文档](https://jestjs.io/docs/zh-Hans/getting-started)
[enzyme文档](https://enzymejs.github.io/enzyme/)