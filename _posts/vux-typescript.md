---
title: TypeScript 应用在在 VUE 项目中的一些总结
date: 2019-06-23 21:30:57
tags:
---

### TypeScript 在 VUE 项目中的一些总结

1、使用vuex：

使用时安装`vuex-class`依赖库即可，参考用法为
```typescript
import { State, Mutation, Getter, Action } from 'vuex-class';
@Component({
  components: {}
})
export default class Home extends Vue {
  @Action('setUser') private setUser!: () => Promise<void>;
  @Mutation('SET_USER') private SET_USER!: () => void;
  @Getter('user') private user!: { name: string };
  @State('username') private username!: string;
  @State(state => state.username) private nickname!: string;
}
``` 

使用`vuex-class`不好的感受：类型无法被识别或者动态提取，每次声明都必须重新声明类型。我的解决方式：导出`store`中使用的类型

2、依赖包的类型：如果依赖包本身没有提供type类型声明，那我们就必须想办法声明依赖：一、运行`@type/[依赖包名称]`安装依赖，`DefinitelyTyped`社区已经记录了 90% 的顶级 JavaScript 库。例如我们要使用`qrcode`这个库，可以运行命令`yarn add --dev @types/qrcode`安装类型声；二、因为依赖库并不一总是有第三方的类型声明，这时候需要手写类型声明：

```typescript
declare module 'mainto-jssdk/interface/micro_HimoProduct_Admin_Files' {
  interface upyunKey {
    upyun_key: string
  }
  interface fileController {
    sign: (obj: upyunKey) => Promise<{
      msg: {
        bucket: string
        policy: string
        signature: string
      }
    }>
  }
  const fileController: fileController
  export default fileController
}
```
`vue`相关依赖库的类型声明在`DefinitelyTyped`中并不是很多，这也体现了一个库的社区优势。

3、`any`类型的使用：1、实在是没有办法了才能使用`any`类型；2、对于所有没有返回值的函数或者方法，都应该声明为 `void `类型，而不是留空，后者为 `any` 类型。

4、在写异步`Action`想到的，怎么用`TypesSript`表示`Promise`返回值类型？从类的角度看，`Promise`本身就是一个范型类，那我就可以这样写：

```typescript
// 范型
function example1 (): Promise<{ name: string }> {
  return Promise.resolve({ name: 'example1' });
}
// 或者
```

也可以通过`resolve`参数描述：

```typescript
const example2 = new Promise((resolve: (arg: string) => void ) => {
    resolve('1')
})
example2.then(res => {
    const example3: number = res // error: Type 'string' is not assignable to type 'number'.
})
```

5、`mixin`写法，因为`vue-property-decorator`并不提供`Mixin`装饰器，没办法只有通过`@Component`注入了：

```typescript
import HomeRoute from './components/HomeRoute/index.vue';
import Auth from '../../mixin/Auth';
@Component({
  components: {
    HomeRoute,
  },
  mixins: [Auth],
})
export default class App extends Vue {};
```

6、`ref`的类型问题，直接使用类似`this.$refs.app.offsetHeight`这样的语法是会报错的，因为`this.$refs.app`的类型为`Vue | Element | Vue[] | Element[]`，这时只需要把上面的代码改为：`(this.$refs.app as HTMLElement).offsetHeight`。

7、对于拓展性参数的表示，索引类型还挺好用的：

```typescript
declare module 'mainto-jssdk/interface/micro_HimoActivity_Activity' {
  interface guardFairyController {
    search: (arg: {
      activity_id: string,
      searchCond?: {
        [index: string]: string
      },
      with?: string,
      orderBy?: string,
      limit: number,
      skip?: number,
      needRank?: boolean,
      needLastItem?: boolean,
      rankCond?: {
        [index: string]: string
      },
      includeDelete?: boolean,
      count?: boolean,
      excel?: string
    }) => Promise<{
      msg: {
        _id: {
          $oid: string,
        },
        photo_path: string,
        wechat_info: {
          wechat_openid: string,
          wechat_headimgurl?: string,
          wechat_nickname?: string
        },
        name: string,
        phone: string,
        type: string,
        guard_num: string,
        rank: string,
        lastItem: any[]
      }[]
    }>
  }
  const guardFairyController: guardFairyController
  export default guardFairyController
}
```

8、多行注释在vscode中，有更更友好的提示：

```typescript
/**
 * @source  App\Rpc\Controllers\CloudSpotCheck
 */
var Common = {
    /**
     * joinSpotCheck
     * @method GET
     * @param  {Object} object
     * @support  {int} spot_id 评审id 
     * @returns
     * @memberof  Common
     */
    joinSpotCheck: function (object, option) {}
}
```