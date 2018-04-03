---
title: "记录vue的踩坑之路以及经验分享"
date: 2018-02-07T10:14:19+08:00
categories:
- technique
- vue
tags:
- vue
keywords:
- vue
thumbnailImage: /img/cover2.jpg
---

<!--more-->
### 坑
1. 路由去掉默认的#  

        ```js
        export default new Router({  
        mode: 'history',//去掉路由中默认的#/  
        routes: [
            {
            path: '/',
            name: 'HelloWorld',
            component: HelloWorld
            }
        ]
        });
        ```
2. 组件中避免for循环时出现重复键报错,可在绑定key值时加一些字符区分  

        ```html
        <template>
            <div v-for="item in items" :key="item.Id+'aabbcc'">
            </div>
        </template>
        ```  

### 组件
1. 简易下拉列表  

        ```html
        <template>
        <div>
            <div class="dropdown-wrapper">
                <div class="dropdown-selected" :class="{ show: isActive }" @click="SetActive">{{selected.Name}}</div>
                <ul class="dropdown-list">
                    <li v-for="item in items" @click="SelectLi(item.Name)" :key="item.Id">{{item.Name}}</li>
                </ul>
            </div>
        </div>
        </template>
        ```

        ```js
        export default {
        name: "",
        data() {
            return {
            isActive: false,
            selected: {
                Id: 0,
                Name: this.items[0].Name
            }
            };
        },
        props: {
            items: {
            type: Array,
            required: true
            }
        },
        methods: {
            SetActive: function(event) {
            this.isActive = !this.isActive;
            },
            SelectLi: function(val) {
            this.selected.Name = val;
            this.SetActive();
            }
        },
        mounted:function(){
            document.addEventListener('click', (e) => {
                console.log(this.$el);
                    if (!this.$el.contains(e.target)){
                        this.isActive = false
                    } 
                });
            }
        };

        ```css
        .dropdown-wrapper {
        position: relative;
        display: inline-block;
        margin-top: 20px;
        margin-left: 20px;
        }
        .dropdown-selected {
        height: 36px;
        line-height: 34px;
        width: 120px;
        border: 1px solid #afafaf;
        padding-left: 10px;
        &.show {
            ~ .dropdown-list {
            display: block;
            }
        }
        }
        .dropdown-list {
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        background-color: #fff;
        z-index: 1;
        border: 1px solid #afafaf;
        display: none;
        > li {
            height: 36px;
            line-height: 36px;
            padding-left: 10px;
        }
        }
        ```  

2. 注册全局引用组件方法  
      1. 引入组件  
            `import Component from '' `  
      2. 导出方法

            ```js  
            const Toast = {};

            // 注册Toast
            Toast.install = function (Vue) {
                // 生成一个Vue的子类
                // 同时这个子类也就是组件
                const ToastConstructor = Vue.extend(Component)
                // 生成一个该子类的实例
                const instance = new ToastConstructor();

                // 将这个实例挂载在新创建的div上
                // 并将此div加入全局挂载点内部
                instance.$mount(document.createElement('div'))
                document.body.appendChild(instance.$el)

                // 通过Vue的原型注册一个方法
                // 让所有实例共享这个方法 
                Vue.prototype.$toast = (msg, duration = 1500) => {
                    instance.message = msg;
                    instance.visible = true;

                    setTimeout(() => {
                        instance.visible = false;
                    }, duration);
                }
            }

            export default Toast
            ```  
    
    3. 在`main.js`文件内引用  
       `import Toast from ''`  
       `Vue.use(Toast)`
    4. 在组件内无需import,直接`this.$toast("")`即可  
  

3. 只允许输入数字的组件  

        ```js
        <input type="text" v-number-only>  

        export default{
            directives: {
                numberOnly: {
                    bind: function(el) {
                        el.handler = function() {
                        el.value = el.value.replace(/\D+/, "");
                        };
                        el.addEventListener("input", el.handler);
                    },
                    unbind: function(el) {
                        el.removeEventListener("input", el.handler);
                    }
                }
        }
        }
        ```   

### 记录
1. http请求  

    配置路由文件`router/config.js`  
    跨域情况下不写  

        ```js
        import axios from 'axios'

        var pName = location.pathname.replace(/\/+/g, '/');
        var id = pName.slice(0, pName.indexOf("/", 2));
        axios.defaults.baseURL = id;
        axios.defaults.withCredentials = true;

        axios.interceptors.request.use(function (config) {
            return config;
        }, function (error) {
            return Promise.reject(error);
        });//请求拦截器

        axios.interceptors.response.use(response => {
            // 请求成功返回
            //your code
            return response;
        }, (responseError) => {
        });//响应拦截器
        ```  
        
       在`main.js`文件中引入,`import config from './router/config'`  
       发布时切换生产环境,`Vue.config.productionTip = true`

        ```js  
        import axios from 'axios'
        import qs from 'qs'

        export function Get(url, data) {
            return new Promise((resolve, reject) => {
                axios.get(url, {
                    params: data
                }).then((res) => {
                    if (res) {
                        if (res.status == 200) {
                            resolve(res.data.data);
                        } else {
                            reject(res);
                        }
                    }
                }).catch((error) => {
                    reject(error);
                })
            });
        }

        export function Post(url, data) {
            return new Promise((resolve, reject) => {
                axios.post(url, qs.stringify(data), {
                    headers: {
                        'Content-Type': 'application/x-www-form-urlencoded',
                        'Accept': 'application/json'
                    }
                }).then((res) => {
                    if (res) {
                        if (res.status == 200) {
                            resolve(res.data.data);
                        } else {
                            reject(res);
                        }
                    }
                }).catch((error) => {
                    reject(error);
                })
            });
        }

        export function PostFlie(url, data) {
            return new Promise((resolve, reject) => {
                //根据data对象生成FormData对象
                var temp = new FormData();
                for (var t in data) {
                    temp.append(t, data[t]);
                }
                axios.post(url, temp).then((res) => {
                    if (res.status == 200) {
                        resolve(res.data.data);
                    } else {
                        reject(res);
                    }
                }).catch((error) => {
                    reject(error);
                })
            })
        }
        ```
        在service层只要引用相对路径就行

        ```js  
        import { Get, Post } from './http'

        export function Interface(data) {
            return Get('/url', data);
        }
        ```
        组件内引用同样只需要相对路径

        ```js
        import { Interface } from "";

        Interface(data).then(res => {});
        ```