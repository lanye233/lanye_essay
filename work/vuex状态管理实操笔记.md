>当你有很多参数状态需要实时改变时，你可以用vuex来管理。
# 1. 首先创建如下的目录：
```
└─store
    │ store.js
    │
    └─modules
            prop1Store.js
            prop2Store.js
            …
```
```
//store.js
import Vue from 'vue'
import Vuex from 'vuex'
import prop1Store from './modules/prop1Store'
import prop2Store from './modules/prop2Store'

Vue.use(Vuex);

export default new Vuex.Store({
    modules: {
        prop1Store,
        prop2Store
    },
});
```
# 2.编写参数绑定的vue页面：
```
//prop1.vue
//在template标签内参数绑定方式
$store.state.prop1Store.Prop1[0]
//1.{{}}中进行数据绑定；2.：class绑定；etc.

//在script标签内
import store from '../store/store'
import {mapState,mapMutations,mapGetters} from 'vuex'
export default {
    data() {
      return {
        
      }
    },
    computed:{
        ...mapGetters(['Prop1','Prop2'])
    },
    destroyed(){
    },
    methods:{
        ...mapMutations([
            'changeInfo'
        ]),
    }
}
```
# 3.编写单独的store参数页面：
```
//prop1Store.js
import Vue from 'vue'
const state = {
    prop1:[]
    //这里填写参数信息,可填写多个，多种类型
}
const getters = {
    prop1: (state) => {
        return state.prop1
    }
}
const actions = {

}
const mutations = {
    //这里写你的函数
    changeState(state) {
    //当flag为true时，每秒触发一次createInfo函数
        setInterval(() => {
            if (flag) {
                this.commit('createInfo')//在vuex中调用函数的方式
            } else {                
            //console.log("--------"+)
        }, 1000)
    },
    createInfo(state) {
        Vue.set(state.test, 0, "测试")
        //由于Vue不能检测到对象属性的添加或删除，所以不能用test[0]="测试"这种方式为参数添加新属性或新值。
    }
    changeInfo(state){
        
    }
}
export default {
    state,
    getters,
    actions,
    mutations,
```
## }

Tips：灵活运用三目运算符！

