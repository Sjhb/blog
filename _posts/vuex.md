---
title: Vuex简单应用
date: 2018-11-11 00:09:46
tags: vuex
---

### 为什么要使用Vuex

在Vue全家桶中，Vue只负责核心的部分，也就是只关心视图层。当我们的应用比较小的时候，一个Vue框架是完全够用的，但是当我们要构建复杂的单页应用的时候，光靠Vue就显得捉襟见肘。

Vuex这个框架就只关注于数据层，它提供了一种更加有条理的方式来帮助我们构建复杂的单页应用。

### 一个demo

光说不练假把式，让我们来看下面的例子：

实现一个购物车，要求有三个页面：列表页、购物车页、详情页面，具体要求如下：

- 列表页：展示一些商品，提供购物车页面入口，点击商品进入商品详情页面

- 详情页面：展示商品详情，并提供加入购物车的功能

- 购物车页面： 展示已经加入购物车的商品

开始动手之前我们整理一下思路：三个页面肯定是使用路由跳转、准备mock数据、使用Vuex管理整个数据

> 代码已上传到仓库：`https://github.com/Sjhb/vuex-demo1`

mock数据准备如下：

```javascript
// product.js
const products = [
  {
    id: 1,
    name: '童装/女童 袜子(2双装) 405454 优衣库UNIQLO',
    source: '//g-search2.alicdn.com/img/bao/uploaded/i4/i3/196993935/TB1w4LVfaLN8KJjSZFmXXcQ6XXa_!!0-item_pic.jpg_360x360Q90.jpg_.webp',
    price: '35.00',
    brand: '优衣库'
  },
  // ...
  {
    id: 8,
    name: '【双十一】佳丽宝Kanebo连裤袜110D*3双装 日本进口保暖',
    source: '//g-search1.alicdn.com/img/bao/uploaded/i4/imgextra/i1/32888477/O1CN01ECV5vx2CUVvP9rtIM_!!0-saturn_solar.jpg_460x460Q90.jpg_.webp',
    price: '49.00',
    brand: '未完待续125810'
  }
]
// 获取所有商品数据
export function getAllProduct () {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(products)
    }, 1000)
  })
}
```

`store`对象准备，代码比较多，我挑重点说明一下：
 
- `state`保存数据，我们必须通过`mutations`来改变里面的值。

- `mutations`是同步的，里面不包含异步操作。

- 为了方便我们存取数据，我声明了两个`getter`，一个是可传参数动态获取，一个是如同属性一般直接获取，可以将其看作计算属性，只要state里面的数据改动，这些getter就会重新计算，避免了计算资浪费

- `actions`可以进行一步操作，常常我们需要获取远程资源，以此来更新state中的数据，这时候就可以利用actions可以异步的特点来做。

```javascript
import Vue from 'vue'
import Vuex from 'vuex'
import * as mock from '../mock/index.js'
Vue.use(Vuex)
export default new Vuex.Store({
  state: {
    products: [],
    cart: {}
  },
  mutations: {
    SET_PRODUCT (state, products) {
      state.products = products
    },
    ADD_PRODUCT (state, prod) {
      let cart = state.cart
      let prodId = prod.id
      if (cart[prodId]) {
        let oldProd = cart[prodId]
        state.cart = Object.assign({}, state.cart, {
          [prodId]: {
            num: ++oldProd.num,
            money: oldProd.money + Number(prod.price),
            ...prod
          }
        })
      } else {
        state.cart = Object.assign({}, state.cart, {
          [prodId]: {
            num: 1,
            money: Number(prod.price),
            ...prod
          }
        })
      }
    }
  },
  getters: {
    getProdById: state => {
      return prodId => state.products.find(prod => Number(prod.id) === Number(prodId))
    },
    prodInCart: state => {
      return Object.values(state.cart)
    }
  },
  actions: {
    // @description 获取所有商品
    getProducts (context) {
      return new Promise((resolve, reject) => {
        mock.getAllProduct().then(res => {
          context.commit('SET_PRODUCT', res)
          resolve()
        }).catch(err => reject(err))
      })
    },
    // @description 添加购物车
    addProdToCart (context, prod) {
      context.commit('ADD_PRODUCT', prod)
    }
  }
})

```

商品列表页(只关注js部分)，`mapActions`是`vuex`官方提供的api，可以将利用它可以得到一个包含我们想要的`actions`的对象，这里我们利用ES7的对象展开运算符即可映射到`methods`属性中

```javascript
import { mapActions } from 'vuex'
export default {
  name: 'prodList',
  components: {},
  computed: {
    products () {
      return this.$store.state.products
    }
  },
  methods: {
    ...mapActions(['getProducts']),
    goDetail (id) {
      this.$router.push({
        name: 'prodDetail',
        query: {
          id
        }
      })
    }
  },
  created () {
    if (this.products.length === 0) {
      this.getProducts()
    }
  }
}
```

### 模块划分

上面的例子中，所有的数据都是写在同一个state中，一般应用不是很大的时候可以这么做。但是在复杂的应用中，我们要管理的数据是很多的，而且还有很多异步的操作和复杂的业务逻辑。这时候把state、action、mutatio、getter写在一起会很臃肿而难以维护，这时候我们就需要分模块了。

参照vuex官方的例子，再将我们上面的store分成两个模块：cart和product分别进行维护，这里贴出cart和store入口的代码

```javascript
// store/cart/index.js
export default {
  state: {
    cart: {}
  },
  mutations: {
    ADD_PRODUCT (state, prod) {
      let cart = state.cart
      let prodId = prod.id
      if (cart[prodId]) {
        let oldProd = cart[prodId]
        state.cart = Object.assign({}, state.cart, {
          [prodId]: {
            num: ++oldProd.num,
            money: oldProd.money + Number(prod.price),
            ...prod
          }
        })
      } else {
        state.cart = Object.assign({}, state.cart, {
          [prodId]: {
            num: 1,
            money: Number(prod.price),
            ...prod
          }
        })
      }
    }
  },
  getters: {
    prodInCart: state => {
      return Object.values(state.cart)
    }
  },
  actions: {
    // @description 添加购物车
    addProdToCart (context, prod) {
      context.commit('ADD_PRODUCT', prod)
    }
  }
}

// store/index.js
import Vue from 'vue'
import vuex from 'vuex'
import product from './product/index.js'
import cart from './cart/index.js'

Vue.use(vuex)
export default new vuex.Store({
  modules: {
    product,
    cart
  }
})
```

对应的，我们再取出数据的时候也要相应更改写法，例如：`this.$store.state.product`就要改为`this.$store.state.product.product`，中间加上的就是模块名。

模块拆分到这里就结束了吗？不是的。有些情况下，像上面这样拆分仍然是不够的。我们还要继续拆分，actions和mutation这样的独立功能的部分要拆出来作为独立的文件。

这里有一篇文章专门在我学习拆分的vuex时候点播了我，感兴趣可以看一下：[手把手教你怎么拆分 vuex@2.x](https://github.com/muzqi/Article/blob/master/blog/vuex.md)

### 一些痛点

单页应用是基于无刷新的Ajax，而vuex中的数据也是保存在浏览器当前页面的运行环境当中。一旦用户刷新页面，将会导致vuex中的数据丢失，从而导致依赖这些数据的页面发生错误，所以，尽可能地，我们需要做一些错误捕获机制，避免刷新之后页面死掉。

### 写在后面

vuex的基础功能其实比较简单，推荐学习facebook推出flux思，有助于我们学习使用vuex。还有一点，当我们把数据、请求都交给vuex维护之后，vue组件编写测试会更加容易一点。