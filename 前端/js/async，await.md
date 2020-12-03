## jasync/await

其实就是promise的语法糖，为了避免promise的回调函数的写法

注意：await只能放在当前async函数中

```javascript
async function fn(){
        let name1=await new Promise((resolve,reject)=>{
            setTimeout(()=>{
                resolve('zs')
            },1000)
        })
        let name2=await new Promise((resolve,reject)=>{
            setTimeout(()=>{
                resolve('lisi')
            },1000)
        })
    }
    fn()
```

由上面的代码可知，如果一个async函数里面有多个await，并且await后面跟着一个promise对象实例，那么下一个awai必须等到上一个await后面的promise构造函数resolve，才会往下执行。说白了，可以这样理解：async相当于new Promise，await相当于then，如果awai后面跟着的不是一个promise对象，而是一个值，那么，await 值通过变量保存就可以直接得到这个值，如果有多个await，后面都跟着一个值，那么就可以依次得到每一个值，但一般我们await后面都会跟着一个函数，这个函数返回一个promise对象，或者像上面这样写，直接new Promise得到一个promise对象，其实也是一样的，少写了个函数。而变量保存的await promise对象得到的其实是这个promise构造函数resolve()出来的值



错误处理

```javascript
 async function fn(){
        throw new Error('fail')
    }

    fn().catch(err=>{
        console.log(err)
    })


async function fn(){
        try {
            let name=await new Promise((resolve,reject)=>{
                reject('fail')
            })
            // throw new Error('失败了')
        } catch (error) {
            console.log(error)
        }
    }
    fn()
```

当async函数内部发生错误的时候，我们可以在函数外边通过catch去捕获错误，但是标准的错误处理是使用try/catch，只要try里面发生了错误，有reject，或者我们手动抛出错误，catch都可以捕获到。

await并行执行技巧

```javascript
function p1(){
        return new Promise((resolve,reject)=>{
            setTimeout(()=>{
                resolve('p1')
            },1000)
        })
    }
    function p2(){
        return new Promise((resolve,reject)=>{
            setTimeout(()=>{
                resolve('p2')
            },1000)
        })
    }

    async function fn(){
        let arr=await Promise.all([p1(),p2()])
        console.log(arr)
        // let h1=p1()
        // let h2=p2()
        // let value1=await h1
        // let value2=await h2
        // console.log(value1,value2)
    }
    fn()
```

并行执行就是里面的多个await一起执行，而不是想最上面的那个，要等到上一个await执行完了，再执行下一个await。