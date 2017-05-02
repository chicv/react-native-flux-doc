# Android集成 react-native-flux 组件的知识点
  
  我们大量参考了[官方文档](https://github.com/aksonov/react-native-router-flux/blob/master/docs/REDUX_FLUX.md)，但是官方文档中并没有写明怎样去分发后台请求来的数据，也就是缺少action组件，当然不排除我没有找到相对应的文档。这个文档主要是对刚才的官方文档做的一个扩充。
  
#### <i class="icon-folder-open"></i> 目录结构

首先看一下我们的目录结构，部分代码没有粘贴出来，是因为和官方文档有重复，比如程序的入口文件```index.android.js```
```
  ├── action.js
  ├── actions
  │   └── getCartData.js
  ├── page
  │   ├── Auth
  │   ├── Cart
  │   │   ├── Edit.js
  │   │   └── Index.js
  │   └── Home
  │       └── Index.js
  ├── reducers
  │   ├── cart.js
  │   ├── index.js
  │   └── routes.js
  ```
