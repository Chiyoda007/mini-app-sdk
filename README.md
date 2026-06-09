# mini-app-sdk


## 安装

### 通过 script 引用

```html
<script src="./mini-app-sdk@1.0.0.min.js"></script>
```

## 快速开始

### ESM

```ts
import { miniSDK } from 'mini-app-sdk';

const appConfig = await miniSDK.getAppConfig();

await miniSDK.goBackToApp();
```

### 浏览器全局引入

通过 `<script>` 引入全局构建后，可以直接使用 `window.miniSDK`：

```html
<script src="./dist/mini-app-sdk.global.js"></script>
<script>
  window.miniSDK.getAppConfig().then(function (config) {
    console.log(config);
  });
</script>
```

如果需要确保 SDK 文件加载完成后再调用，可以使用 `onload`：

```html
<script>
  function onMiniSDKLoaded() {
    window.miniSDK.goBackToApp();
  }
</script>
<script src="./dist/mini-app-sdk.global.js" onload="onMiniSDKLoaded()"></script>
```

如果需要自定义 bridge 配置，可以创建独立实例：

```ts
import { createMiniAppSDK } from 'mini-app-sdk';

const miniSDK = createMiniAppSDK({
  appId: 'demo-app',
  environment: 'development',
  debug: true,
  bridge: {
    timeout: 10000,
  },
});

miniSDK.updateConfig({
  debug: false,
});

const appConfig = await miniSDK.getAppConfig();
```

## 内置原生能力

### getAppConfig

获取 App 配置参数。

```ts
const appConfig = await miniSDK.getAppConfig();

console.log(appConfig.safeArea.top);
console.log(appConfig.language);
console.log(appConfig.theme);
console.log(appConfig.version);
console.log(appConfig.platform);
```

返回结构：

```ts
interface AppConfig {
  safeArea: {
    top: number;
    bottom: number;
  };
  language: 'ZH-HANS' | 'ZH-HANT' | 'EN' | 'ru-RU' | 'vi' | 'ja' | 'ko' | string;
  theme: 'light' | 'dark' | string;
  version: string;
  platform: 'ios' | 'android' | string;
}
```

### goBackToApp

返回 App。

```ts
await miniSDK.goBackToApp();
```

### getNotchScreenHeight

获取刘海屏或安全区域高度。底层调用原生 `getSafeInset`。

```ts
const safeInset = await miniSDK.getNotchScreenHeight<{ top: number; bottom: number }>();
```

### saveImage

保存图片，支持传入 `url`、`base64` 或 `arrayBuffer`。

```ts
await miniSDK.saveImage({
  url: 'https://example.com/qrcode.png',
});
```

传入 `arrayBuffer` 时，SDK 会自动转成普通数组后交给原生侧。

### login

获取用户登录凭证。原生侧会调用后端接口生成 `code` 并返回给 H5。

```ts
const { code } = await miniSDK.login();

console.log(code);
```

返回结构：

```ts
interface LoginResult {
  code: string; // 用户登录凭证
}
```

### getAssets

获取用户 USDT 资产列表。原生侧会调用后端接口并过滤出 `USDT` 币种数据后返回。

```ts
const assets = await miniSDK.getAssets();

assets.forEach((asset) => {
  console.log(asset.token_name);
  console.log(asset.balance);
  console.log(asset.total_usdt);
});
```

返回结构：

```ts
interface AssetItem {
  account: string;
  balance: string;
  balance_usdt: string;
  frozen_balance: string;
  frozen_usdt: string;
  icon: string;
  token_name: string;
  token_whole_name: string;
  total: string;
  total_usdt: string;
}

type GetAssetsResult = AssetItem[];
```


### openCashier

打开 H5 收银台弹窗，用于交易确认和资金密码输入。

```ts
const result = await miniSDK.openCashier({
  prepayId: '347djchd-3394-44-98dd-2323dc9cdjc',
  amount: 100,
  currency: 'USDT',
  description: 'test',
  attach: 'test',
  deadline: 1779527762206,
});

console.log(result.prepayId);
console.log(result.orderPayStatus);
console.log(result.currency);
console.log(result.amount);
console.log(result.description);
console.log(result.attach);
console.log(result.deadline);
```

请求参数（`prepayId`、`amount`、`currency` 必填，其余用于原生弹窗展示）：

```ts
interface OpenCashierOptions {
  prepayId: string;
  amount: number;
  currency: string;
  description?: string;
  attach?: string;
  deadline?: number;
}
```

确认后返回：

```ts
interface OpenCashierResult {
  prepayId: string;
  orderPayStatus: string;
  currency: string;
  amount: number;
  description: string;
  attach: string;
  deadline: number;
}
```

## Bridge 兼容

SDK 内部会自动兼容：

- iOS `window.WKWebViewJavascriptBridge.callHandler`
- Android `window.AndroidInterface[handlerName]`
- 自定义 `bridgeAdapter`

## API

- `miniSDK.updateConfig(config)`：更新 SDK 配置。
- `miniSDK.getAppConfig()`：调用原生 `getAppConfig`，获取 App 配置参数。
- `miniSDK.goBackToApp()`：调用原生 `goBackToApp`。
- `miniSDK.getNotchScreenHeight()`：调用原生 `getSafeInset`。
- `miniSDK.saveImage(options)`：调用原生 `saveImage`。
- `miniSDK.openCashier(options)`：调用原生收银台并返回订单支付结果。


