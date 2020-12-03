## 什么是异步编程？

异步是 相对于同步而言的，同步就是一件事一件事的执行，只有前一个任务执行完毕，才能执行后一个任务，而异步是不用等待前一个任务执行完成也能够执行。setTimeout就是一个异步任务，当js引擎顺序执行到setTimeout的时候发现他是个异步任务，则会把这个任务挂起，继续执行后面的代码。

js是单线程的，只能在js引擎的主线程上运行，所以js代码只能一行一行执行，不能再同一时间执行多个js代码任务，这就导致如果有一段耗时较长的计算，或者一个ajax请求的IO操作，如果没有异步存在，就会出现用户长时间等待，并且由于当前任务还未完成，所以这时候所有的其他操作都会无响应。

比如说打扫卫生，洗衣服，煮饭，如果是同步的进行，就会耗时很长的时间，等一个任务完成才可以去执行下一个任务，如果我们通过电饭煲煮饭，通过洗衣机洗衣服，让他们帮我们做事，相当于电饭煲和洗衣机是不同的模块，不同的模块去处理自己的任务，处理完成之后就滴滴的响，代表某个模块的任务处理完了，然后，我们再将洗好的衣服挂起来，煮好的饭放桌上。所以我们在开发过程中，比如setTimeout或者ajax请求，js会自己将任务交给对应的模块，我们的这些异步操作都处在任务队列中，主线程会不断的去任务队列中轮询，去任务队列里面找有没有新的任务，最后把任务队列返回的东西放在主线程中使用。

而我们用的promise，async/await，只是想让我们的异步操作的开发手段，在写代码的时候更加清晰，敏捷。

体验一下异步加载图片

```javascript
function loadImg(src,resolve,reject){
            let img=new Image()
            console.log(img)
            img.src=src
            img.onload=()=>{
                resolve(img)
            }
            img.onerror=reject
        }
        loadImg('https://hskj-test1.oss-cn-shanghai.aliyuncs.com/hs_sys/homeNav1599556606088.png',
        (image)=>{
            console.log('加载成功')
            document.body.appendChild(image)

        },()=>{
            console.log('加载失败')
        })
        console.log('加载')
```

什么是宏任务，什么是微任务？

首先理解js的运行机制，js是单线程的，我们已经知道了，在执行js代码的时候，主线程会不断的去任务队列里面去轮询，拿到任务队列中已经完成任务的模块的返回值，然后进行处理，但是任务队列里面又分为宏任务和微任务，在主线程中的任务相当于同步任务，相当于同步任务的优先级最高，最先执行，然后才执行宏任务和微任务，而这两个谁的优先级更高呢？

```javascript
	   setTimeout(()=>{
            console.log('setTimeout')
        },0)
        new Promise((resolve,reject)=>{
            console.log('promise')
            resolve('then')
            console.log('thenLast')
        }).then(res=>{
            console.log(res)
        })
        console.log(123)

		promise
        thenLast
        123
		then
        setTimeout

```

微任务的优先级高于宏任务，当我们的js代码执行的时候，setTimeout和构造函数promise运行的时候是同步任务，比如promise，只有当执行了resolve，才会将then方法丢到微任务队列中执行。

回调地狱

```javascript
ajax('url1',(res1)=>{
    ajx(`url2?id=${res1.id}`,(res2)=>{
          ajx(`url3?id=${res2.id}`,(res3)=>{
          	console.log(res3)
		})
	})
})
```

当一个请求所需的参数来自于上一个请求回调的参数，就会在异步回调函数中一层一层的进行网络请求，就会出现回调地狱的写法，也就是函数嵌套，这种写法不便于阅读



下面是解决回调地狱的问题

```javascript
 new Promise((resolve, reject) => {
            ajax({
                url: 'url',
                success: (res) => {
                    resolve(res)
                },
                fail: (err) => {
                    reject(err)
                }
            })
        }).then(
            res => {
                //res上一个异步请求得到的数据
                //下一个接口还需要用到上一个异步得到的数据
                return new Promise((resolve, reject) => {
                        ajax({
                            url: `url?id=${res.id}`,
                            success: (res) => {
                                resolve(res)
                            },
                            fail: (err) => {
                                reject(err)
                            }
                        })
                    },
                )
            },
            err => console.log(err)
        ).then(
            res=>console.log(res),
            err=>console.log(err)
        )
```

promise.then()也是一个promise，当我们在进行promise的promise.then().then()这种链式调用的时候，下一个then取决于上一个then的返回值，如果上一个then返回的不是一个新的promise，那么，上一个then返回的还是原来的promise对象，不过我们在开发过程中，如果存在接口请求有依赖关系的时候，也就是需要发送多个请求的时候，就采用上面的操作，这样子解决了回调地狱的问题，且直观。

错误处理catch

```javascript
new Promise((resolve,reject)=>{
            resolve('成功1')
            // reject('错误1')
        }).then(res=>{
            console.log(res)
            return new Promise((resolve,reject)=>{
                // resolve('成功2')
                reject('错误2')
            })
        }).then(res=>{
            console.log(res)
        }).catch(err=>{
            console.log(err)
        })
```

对于错误处理，比如接口请求的时候走的是fail，那么在第一个then的时候就会走错误的回调函数，根本不会去走成功的那条路，所以，也不会去进行第二个请求。还有就是我们不用在每一个then里面写第二个错误处理函数，去针对每个promise错误处理。而是交给catch统一处理，默认我们将catch写在最后面，catch就接受我们的错误，我们在这里面进行错误处理，比如第一个promise发生错误，就直接到catch，不走第一个then，更不会走第二个then，如果第一个promise成功，就会走第一个then，然后第二个promise错误，就走catch，不会走第二个then。

封装一个网络请求

```javascript
function api(url){
            return new Promise((resolve,reject)=>{
                ajax({
                    url:url,
                    success:(res)=>{
                        resolve(res)
                    },
                    fali:(err)=>{
                        reject(err)
                    }
                })
            })
        }

        api('url1')
        .then(res1=>api(`url2?id=${res1.id}`))
        .then(res2=>console.log(res2))
        .catch(err=console.log(err))
```

如何自定义错误处理

在了解这个之前先学习一下try/catch

```javascript
 function fn(name){
            try {
                if(name ==='zs'){
                    throw Error('请求参数有问题')
                }
            } catch (error) {
                console.log(err)
            }
        }
        fn('zs')
```

当执行fn函数的时候，如果try中抛出错误，那么catch就会接受错误对象，在目前小程序项目中网络请求封装就是通过这个手段去自定义错误处理。

对于错误处理还是要分为两种情况，一种是想要用户看到错误原因，一种是不想要用户看到，前端在开发调试过程中可以看。接下来我们就看看下面这种网络请求与异常处理

```javascript
   class ParamsError extends Error{
            constructor(msg){
                super(msg)
                this.name='ParamsError'
            }
        }
        class HttpError extends Error{
            constructor(msg){
                super(msg)
                this.name='HttpError'
            }
        }
        function api(url){
            return new Promise((resolve,reject)=>{
                if(!/^http/.test('url')){
                    //这里面是同步执行的所以可以直接抛出错误，然后在catch接收
                    //当然下面的网络请求就不会再执行了
                    throw new ParamsError('请求地址格式错误')
                }
                ajax({
                    url:url,
                    success:(res)=>{
                        resolve(res)
                    },fail:(err)=>{
                        if(err.status == 404){
                            //这里抛出错误必须要通过reject
                            //因为这里回调是异步执行的
                            //如果直接throw抛出。下面的catch根本不会执行
                            reject(new HttpError('用户不存在'))
                        }
                        
                    }
                })
            })
        }
        api('url')
        .then(res=>{
            console.log(res)
        })
        .catch(err=>{
            console.log(err)
            //这种抛出给开发者看
            if(err instanceof ParamsError){
                console.log(err.message)
            }
            //这种错误抛出针对给用户看
            if(err instanceof HttpError){
                alert('用户不存在')
            }
        })
```

定时器setTimeout的封装

```javascript
function timeout(delay=1000){
            return new Promise((resolve,reject)=>{
                setTimeout(resolve,delay)
            })
        }
        timeout(2000).then(()=>{
            console.log('2s执行内容')
            return timeout(2000)
        }).then(res=>{
            console.log('2s执行其他内容')
        })

```

定时器setInterval的封装

```javascript
function interval(delay=100,callback){
            return new Promise((resolve,reject)=>{
                let id=setInterval(()=>{
                    callback(id,resolve)
                },delay)
            })
        }
        interval(20,(id,reslove)=>{
            const div=document.querySelector('div')
            let left=parseInt(window.getComputedStyle(div).left)
            div.style.left=left+1+'px'
            if(left >=200){
                clearInterval(id)
                reslove(div)
            }
        }).then(res=>{
            return interval(20,(id,resolve)=>{
                let width=parseInt(window.getComputedStyle(res).width)
                res.style.width=width-1+'px'
                if(width <=20){
                    clearInterval(id)
                    resolve(res)
                }
            })
        }).then(res=>{
            console.log(res)
        })
```

```javascript
//针对一个页面在不同的时间内多次进行同一个接口请求，我们通过缓存去减少对服务器的压力

        function query(name){
            const cache=query.cache || (query.cache=new Map())//定义成map类型
            if(cache.has(name)){
                console.log('走缓存了')
                return Promise.resolve(cache.get(name))
            }
            return new Promise((resolve,reject)=>{
                //这里面进行异步操作
                let getPrams={a:1,b:2}
                cache.set(name,getPrams)
                console.log('没走缓存'))
                resolve(getPrams)
            })
        }
        query('zs').then(res=>{
            console.log(res)
        })
        setTimeout(()=>{
            query('zs').then(res=>{
                console.log(res)
            })
        },2000)
```

请求超时处理

```javascript
//原理
        function api(){
            return new Promise((resolve,reject)=>{
                ajax({
                    url:url,
                    success:(res)=>{
                        resolve(res)
                    },
                    fail:(err)=>{
                        reject(err)
                    }
                })
            })
        }   

        let promise=[ 
            api('url'),
            new Promise((resolve,reject)=>{
                setTimeout(()=>{
                    console.log('请求超时')
                },3000)
            })
        ]

        Promise.race(promise).then(res=>{
            console.log(res)
        }).catch(err=>{
            console.log(err)
        })

        //封装
        function query(url,delay=2000){
            let promise=[
                api(url),
                new Promise((resolve,reject)=>{
                    setTimeout(()=>{
                        reject('请求超时')
                    },delay)
                })
            ]
            return Promise.race(promise)
        }
        query('url',3000).then(res=>{
            console.log(res)
        }).catch(er=>{
            console.log(err)
        })
```

Promise.race()数组里面谁先加载就返回谁。可以用来做超时处理

Promise队列原理

使用map封装一个promise队列

```javascript
let p=Promise.resolve('zs')
      p=p.then(res=>{
          return new Promise((resolve,reject)=>{
              setTimeout(() => {
                  console.log(1)
                  resolve()
              }, 1000);
          })
      })
      p.then(res=>{
          return new Promise((resolve,reject)=>{
              setTimeout(() => {
                  console.log(2)
                  resolve()
              }, 1000);
          })
      })


封装成一个函数
function queue(num){
        let promise=Promise.resolve()
        //前面都走完才能走下一个，变量promise必须保存上一个成员返回的promise对象
        num.map(v=>{
            promise=promise.then(res=>{
                return new Promise((resolve,reject)=>{
                    setTimeout(() => {
                        console.log(v)
                        resolve()
                    }, 1000);
                })
            })
        })
    }
    queue([1,2,3])



function queue(num){
        let promise=Promise.resolve()
        //前面都走完才能走下一个，变量promise必须保存上一个成员返回的promise对象
        num.map(v=>{
            promise=promise.then(res=>{
                return v()
            })
        })
    }
    function p1(){
        return new Promise((resolve,reject)=>{
            setTimeout(()=>{
                console.log('p1')
                resolve()
            },1000)
        })
    }
    function p2(){
        return new Promise((resolve,reject)=>{
            setTimeout(()=>{
                console.log('p2')
                resolve()
            },1000)
        })
    }
    queue([p1,p2])
```

队列里面的每一个成员都要是promise，下一个成员依赖上一个成员的状态改变，也就是说队列的含义即上一个成员执行了，才可以下一个成员才可以执行，上一个成员必须返回一个promise，并且resolve()，只有这样，下一个成员才会被执行。

使用reduce封装一个promise队列

```javascript
function queue(num){
        let promise=Promise.resolve()
        //前面都走完才能走下一个，变量promise必须保存上一个成员返回的promise对象
        num.reduce((promise,n)=>{
            return promise.then(res=>{
                return new Promise((resolve,reject)=>{
                    setTimeout(()=>{
                        console.log(n)
                        resolve()
                    },1000)
                })
            })
        },Promise.resolve())
    }
  
    queue([1,2,3,4])
```

使用队列渲染数据

```javascript
class User{
        api(user){
            return new Promise((resolve,reject)=>{
                ajax({
                    url:`url?name=${user}`,
                    success:(res)=>{
                        resolve(res)
                    },fail:(err)=>{
                        reject(err)
                    }
                })
            })
        }
        render(users){
            users.reduce((promise,user)=>{
                return promise.then((res)=>{
                    return this.api(user)
                }).then(res=>{
                    return this.view(res)
                })
            })
        }
        view(user){
            return new Promise((resolve,reject)=>{
                //得到的后台数据user
                console.log(user)
            })
        }
    }
    new User().render(['zs','lisi'])
```

