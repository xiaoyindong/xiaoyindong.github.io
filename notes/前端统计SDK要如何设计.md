主要思路是需求转换成功能然后将功能转换为代码。

H5的sdk将数据发送统计服务，一般每天统计服务都会对数据生成一份报表，然后再根据报表的数据优化统计代码，这样就形成一个闭环。

开始之前首先要确定需求范围，要统计哪些数据。比如下面

PV：访问量
自定义事件：按钮点击量交互场景
性能统计：
错误统计：

注意pv在单页面的时候需要手动发送，不能直接调用，因为单页面路由更新并不会触发pv事件。

发送数据使用img标签，优点是可以跨域，而且使用简单。

代码演示：

```js
const PV_URL_SET = new Set();
MyStatistic {
  constructo r(productId) {
    // 标识产品线
    this.productId = productId;
    this.initError();
    this.initPerformance();
  }
  // 发送数据
  send(url, params = {}) {
    params.productId = this.productId;
    const paramArr = [];
    for (let key in params) {
      const val = params[key];
      paramArr.push(`${key}=${value}`);
    }
    const newUrl = `${url}?${paramArr.join('&')}`;
    const img = document.createElement('img');
    img.src = newUrl;
  }
  // 初始化性能
  initPerformance() {
    const url = 'yyyy';
    this.send(url, performance.timing);
  }
  // 初始化错误
  initError() {
    window.addEventListener('error', event => {
      const { error, line, col} = event;
      this.err(error, { line, col});
    })
    window.addEventListener('unhandledrejection', event => {
      this.error(new Error(event.reason), {type: 'unhandlerejection'});
    })
  }
  pv() {
    const href = localtion.href;
    if (PV_URL_SET.get(href)) { return; };
    PV_URL_SET.add(href);
    this.event('pv');
  }
  event(key, val) {
    // 自定义事件统计
    const url = 'xxx';
    this.send(url, { key, val });
  }
  error(error, info = {}) {
    const url = 'zzzz';
    const { message, stack } = err;
    this.send(url, { message, stack, ...info });
  }
}
const s = new MyStatistic('a1');
```