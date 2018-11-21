---
title: vue 函数配置项watch以及函数 $watch 源码分享
date: 2018-11-19 17:26:30
---
## Vue双向榜单的原理 ##

在watch的处理中将运用到Vue的双向榜单原理，所以再次回顾一下：



    // 为data的的所有属性添加getter 和 setter
    function defineReactive( obj,key,val,customSetter,shallow
    ) {
        //
        var dep = new Dep();

        /*....省略部分....*/
        var childOb = !shallow && observe(val);  //为对象添加备份依赖dep
        Object.defineProperty(obj, key, {
            enumerable: true,
            configurable: true,
            get: function reactiveGetter() {
                var value = getter ? getter.call(obj) : val;
                if (Dep.target) {
                    dep.depend();  // 
                    if (childOb) {
                        childOb.dep.depend(); //依赖dep 添加watcher 用于set ,array改变等使用
                        if (Array.isArray(value)) {
                            dependArray(value);
                        }
                    }
                }
                return value
            },
            set: function reactiveSetter(newVal) {
                var value = getter ? getter.call(obj) : val;
                /* eslint-disable no-self-compare */
                if (newVal === value || (newVal !== newVal && value !== value)) {
                    return
                }
                /* eslint-enable no-self-compare */
                if ("development" !== 'production' && customSetter) {
                    customSetter();
                }
                if (setter) {
                    setter.call(obj, newVal);
                } else {
                    val = newVal;
                }
                childOb = !shallow && observe(newVal);
                dep.notify();//有改变触发watcher进行更新
            }
        });
    }


<font  size=3 face="黑体" style="line-height:28px">
    在vue进行实例化的时候，将调用 initWatch(vm, opts.watch);进行初始化watch的初始化，该函数最终将调用  vm.$watch(expOrFn, handler, options) 进行watch的配置，下面我们将讲解  vm.$watch方法
</font>

    Vue.prototype.$watch = function (
            expOrFn,
            cb,
            options
        ) {
            var vm = this;
            if (isPlainObject(cb)) {
                return createWatcher(vm, expOrFn, cb, options)
            }
            options = options || {};
            options.user = true;
            //为需要观察的 expOrFn 添加watcher ，expOrFn的值有改变时执行cb，
            //在watcher的实例化的过程中会对expOrFn进行解析，并为expOrFn涉及到的data数据下的def添加该watcher
            var watcher = new Watcher(vm, expOrFn, cb, options);

            //immediate==true 立即执行watch handler
            if (options.immediate) {  
                cb.call(vm, watcher.value);
            }

            //取消观察函数
            return function unwatchFn() {
                watcher.teardown();
            }
        };


来看看实例化watcher的过程中（只分享是观察函数中的实例的watcher）

      var Watcher = function Watcher(
        vm,
        expOrFn,
        cb,
        options,
        isRenderWatcher
    ) {
        this.vm = vm;
        if (isRenderWatcher) {
            vm._watcher = this;
        }
        vm._watchers.push(this);
        // options
        if (options) {
            this.deep = !!options.deep; //是否观察对象内部值的变化
            this.user = !!options.user;
            this.lazy = !!options.lazy;
            this.sync = !!options.sync;
        } else {
            this.deep = this.user = this.lazy = this.sync = false;
        }
        this.cb = cb;
        this.id = ++uid$1; // uid for batching
        this.active = true;
        this.dirty = this.lazy; // for lazy watchers
        this.deps = [];
        this.newDeps = [];
        this.depIds = new _Set();
        this.newDepIds = new _Set();
        this.expression = expOrFn.toString();
        // parse expression for getter
        if (typeof expOrFn === 'function') {
            this.getter = expOrFn;
        } else {
            // 将需要观察的数据：string | Function | Object | Array等进行解析  如：a.b.c, 并返回访问该表达式的函数  
            this.getter = parsePath(expOrFn);  
            if (!this.getter) {
                this.getter = function () { };
                "development" !== 'production' && warn(
                    "Failed watching path: \"" + expOrFn + "\" " +
                    'Watcher only accepts simple dot-delimited paths. ' +
                    'For full control, use a function instead.',
                    vm
                );
            }
        }
        // this.get()将访问需要观察的数据 
        this.value = this.lazy
            ? undefined
            : this.get(); 
    };

    /**
     * Evaluate the getter, and re-collect dependencies.
     */
    Watcher.prototype.get = function get() {
        //this为$watch方法中实例化的watcher
        pushTarget(this);讲this赋给Dep.target并缓存之前的watcher
        var value;
        var vm = this.vm;
        try {
            //访问需要观察的数据,在获取数据的getter中执行dep.depend();将$watch方法中实例化的watcher添加到对应数据下的dep中
            value = this.getter.call(vm, vm); 
        } catch (e) {
            if (this.user) {
                handleError(e, vm, ("getter for watcher \"" + (this.expression) + "\""));
            } else {
                throw e
            }
        } finally {
            // "touch" every property so they are all tracked as
            // dependencies for deep watching
            if (this.deep) {
                traverse(value);
            }
            popTarget(); //将之前的watcher赋给Dep.target
            this.cleanupDeps();
        }
        return value
    };

