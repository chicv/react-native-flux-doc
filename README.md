# Android集成 react-native-flux 组件的知识点
  
  我们大量参考了[官方文档](https://github.com/aksonov/react-native-router-flux/blob/master/docs/REDUX_FLUX.md)，但是官方文档中并没有写明怎样去分发后台请求来的数据，也就是缺少action组件，当然不排除我没有找到相对应的文档。这个文档主要是对刚才的官方文档做的一个扩充。
  
####目录结构

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
1. ```action.js```只是定义了一些action的常量信息，供```actions```文件夹里的获取数据进行标示和```reducers```文件夹里的reducer进行数据关联
---------------------
  actions/getCartData.js
  ```
    import * as types from '../action';

    export function getCartData(){
        // request server
        return {
            type : types.GET_CART_DATA,
            data : {
                name : 'Hello World'
            }
        }
    }
    export function updateCartData(_name){
        return {
            type : types.GET_CART_DATA,
            data : {
                name : _name
            }
        }
    }
  ```
  reducers/cart.js  
  ------------
```javascript
    import { ActionConst } from 'react-native-router-flux';
    import * as types from '../action';
    const initialState = {
        cartData: {},
    };

    export default function reducer(state = initialState, action = {}) {
        switch (action.type) {
            case types.GET_CART_DATA:
                return{
                    ...state,
                    cartData : action.data,
                };    
            default:
                return state;
        }
    }
```
2.```page```文件夹中相当于view层，所有页面渲染都在当前，```Index.js```
------------------
  ```javascript
  import React, { Component, PropTypes } from 'react';
  import { Text, View,Button } from 'react-native';
  import {getCartData,updateCartData} from '../../actions/getCartData';
  import { connect } from 'react-redux';
  import {
      Actions,
  } from 'react-native-router-flux';
  class Cart extends Component {
    static propTypes = {
        routes: PropTypes.object,
        dispatch : PropTypes.func,
    };
    static contextTypes = {
        routes: PropTypes.object.isRequired,
    };
    componentDidMount(){
        this.props.dispatch(getCartData());


        //TODO:实验是否可以异步更新数据
        RestApi.doGet("productindex"
            ,{}
            ,(code, message) => {
                //Actions
                console.log('helo')
                this.props.dispatch(updateCartData("fangfang"))
                
            }
        );

    }
    render() {
        console.log(this.props,'this is index props')
        const { routes } = this.context;
        //console.log(routes,'this is refresh')
        //var name = this.props.routes ? this.props.routes.scene.name : '';
        var requireRefresh = this.props.routes.scene.requireRefresh;
        return (
            <View>
                  <Text>
                        The current scene is titled {this.props.routes.scene.title}.
                    The current scene is titled {this.props.routes.scene.title}.
                    The current scene is titled {this.props.routes.scene.title}.
                    The current scene is titled {this.props.cart.cartData.name}.
                </Text>
                <Button onPress={() => routes.cartEdit({ "data": "hello" })} title="编辑"/>    
                    <Text>{requireRefresh ? 'refresh' : ''}</Text>
            </View>

        );
    }
}

export default connect(({ routes,cart }) => ({ routes,cart }))(Cart);
  ```
这里要注意最后导出组件的时候，用```connect```方法将```reducer```中的数据进行关联
3.```reducer```文件夹中的文件说明。
--------------
```javascript
import { combineReducers } from 'redux';
import routes from './routes';
// ... other reducers
import cart from './cart';

export default combineReducers({
  routes,
  // ... other reducers
  cart, //这里每定义新的actions中，都要在这里声明
});


//cart.js
import { ActionConst } from 'react-native-router-flux';
import * as types from '../action';
const initialState = {
    cartData: {},
};

export default function reducer(state = initialState, action = {}) {
    switch (action.type) {
        case types.GET_CART_DATA:
            //console.log(action.data,'this is actions in cart.js   ==== = = == = = == = =')
            return{
                ...state,
                cartData : action.data,
            };    
        default:
            return state;
    }
}

```

  
  
