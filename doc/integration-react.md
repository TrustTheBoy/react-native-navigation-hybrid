# 集成到以 RN 为主的项目

你想用 React Native 实现大部分业务，原生代码主要起到搭桥的作用。

假设你是通过 `react-native init MyApp` 创建的项目，目录结构是这样的：

```
MyApp/
|—— android/
|—— ios/
|—— node_modules/
|—— package.json
```

## 添加依赖

```
npm install react-native-navigation-hybrid --save
# or
yarn add react-native-navigation-hybrid
```

## RN 项目配置

打开 index.js 这个文件，通常，它就在 package.json 旁边。

以前，你是这么注册 React 组件

```javascript
import { AppRegistry } from 'react-native';
import App from './App';
import { name as appName } from './app.json';

AppRegistry.registerComponent(appName, () => App);
```

现在，你需要像下面那样

```javascript
import { ReactRegistry, Garden, Navigator, BarStyleDarkContent } from 'react-native-navigation-hybrid';
import App from './App';

// 配置全局样式
Garden.setStyle({
  topBarStyle: BarStyleDarkContent,
});

// 重要必须
ReactRegistry.startRegisterComponent();

// 注意，你的每一个页面都需要注册
ReactRegistry.registerComponent('App', () => App);

// 重要必须
ReactRegistry.endRegisterComponent();
```

通过 `Navigator#setRoot` 来设置 UI 层级

```javascript
Navigator.setRoot({
  stack: {
    children: [{ screen: { moduleName: 'App' } }],
  },
});
```

具体应用请查看 [Navigator#setRoot](./navigation.md)

> 注意：如果你需要等待异步数据结果来决定显示什么样的 UI 层级，那么 `ReactRegistry.endRegisterComponent` 也需要放到异步结果的代码中执行，在 `Navigator.setRoot` 之前。注意不要多次调用 `ReactRegistry.endRegisterComponent`。

### 支持 Redux

想要为每个页面都注入相同的属性，可以利用 `ReactRegistry.startRegisterComponent()` 这个方法，它接受一个 [HOC](https://reactjs.org/docs/higher-order-components.html) 作为参数。

想要支持 Redux，像下面这样配置即可

```jsx
function withRedux(WrappedComponent) {
  return props => (
    <Provider store={store}>
      <WrappedComponent {...props} />
    </Provider>
  );
}

ReactRegistry.startRegisterComponent(withRedux);
```

其中 `withRedux` 就是一个 [HOC](https://reactjs.org/docs/higher-order-components.html)

## Android 项目配置

假设你已经配置好了 React 项目

修改 MainActivity.java 文件

```diff
- import com.facebook.react.ReactActivity;
+ import com.navigationhybrid.ReactAppCompatActivity;

- public class MainActivity extends ReactActivity {
+ public class MainActivity extends ReactAppCompatActivity {
-   @Override
-   protected String getMainComponentName() {
-       return "MyApp";
-   }
}
```

修改 MainApplication.java 文件

```diff
  import com.facebook.react.ReactNativeHost;
+ import com.navigationhybrid.ReactBridgeManager;

public void onCreate() {
    super.onCreate();
    SoLoader.init(this, /* native exopackage */ false);

+   ReactBridgeManager bridgeManager = ReactBridgeManager.get();
+   bridgeManager.install(getReactNativeHost());
}
```

## iOS 项目配置

修改 AppDelegate.h 文件

```diff
- #import <React/RCTBridgeDelegate.h>
#import <UIKit/UIKit.h>

- @interface AppDelegate : UIResponder <UIApplicationDelegate, RCTBridgeDelegate>
+ @interface AppDelegate : UIResponder <UIApplicationDelegate>

@property (nonatomic, strong) UIWindow *window;

@end
```

修改 AppDelegate.m 文件

```diff
  #import <React/RCTBridge.h>
  #import <React/RCTBundleURLProvider.h>
- #import <React/RCTRootView.h>
+ #import <NavigationHybrid/NavigationHybrid.h>

  @implementation AppDelegate

  - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
  {
-  RCTBridge *bridge = [[RCTBridge alloc] initWithDelegate:self launchOptions:launchOptions];
-  RCTRootView *rootView = [[RCTRootView alloc] initWithBridge:bridge
-                                                   moduleName:@"RN60"
-                                            initialProperties:nil];
-
-  rootView.backgroundColor = [[UIColor alloc] initWithRed:1.0f green:1.0f blue:1.0f alpha:1];
-
+  NSURL *jsCodeLocation;
+  jsCodeLocation = [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index" fallbackResource:nil];
+  [[HBDReactBridgeManager sharedInstance] installWithBundleURL:jsCodeLocation launchOptions:launchOptions];
+
   self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
+  self.window.windowLevel = UIWindowLevelStatusBar + 1;
   UIViewController *rootViewController = [UIViewController new];
-  rootViewController.view = rootView;
+  rootViewController.view.backgroundColor = UIColor.whiteColor;
   self.window.rootViewController = rootViewController;
   [self.window makeKeyAndVisible];
   return YES;
 }

- -(NSURL *)sourceURLForBridge:(RCTBridge *)bridge
- {
- #if DEBUG
-   return [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index" fallbackResource:nil];
- #else
-   return [[NSBundle mainBundle] URLForResource:@"main" withExtension:@"jsbundle"];
- #endif
- }
 @end
```

修改 Info.plist 文件

![controller-base](../screenshot/controller-base.jpg)

## 关于闪屏

<a name="migrate-native"></a>

如果懂原生开发，建议像 playground 这个项目那样配置闪屏。
