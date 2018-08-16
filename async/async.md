## Async Await 


A function creating a new Promisse 
```js 
 function breathe(ammount) {
            return new Promise((resolve, reject) => {
                setTimeout(() => resolve(`done for ${ammount}`), ammount)
            })
        }

        breathe(1000).then(res => {
            console.log(res)
        })
```

Chaining this function 
```js

function breathe(ammount) {
            return new Promise((resolve, reject) => {
                setTimeout(() => resolve(`done for ${ammount}`), ammount)
            })
        }

        async function go() {
            console.log('start');
            const response = await breathe(600)
            console.log(response);
        }

        go()
```


Catching errors when there is a chaining call 

```js 
function breathe(ammount) {
            return new Promise((resolve, reject) => {
                if (ammount > 500) {
                    reject('That is too small')
                }
                setTimeout(() => resolve(`done for ${ammount}`), ammount)
            })
        }

        function catchError(fn) {
            return function () {
                return fn().catch((err) => console.error(err))
            }
        }

        async function go() {
            console.log('start');
            const response = await breathe(600)
            console.log(response);
            const r2 = await breathe(600)
            console.log(r2);
            const r3 = await breathe(600)
            console.log(r3);
            console.log('end')
        }
        const wrappedFunction = catchError(go)
        wrappedFunction()

```

Now parsing arguments with the spread operator 

```js 
function breathe(ammount) {
            return new Promise((resolve, reject) => {
                if (ammount > 500) {
                    reject('That is too small')
                }
                setTimeout(() => resolve(`done for ${ammount}`), ammount)
            })
        }

        function catchError(fn) {
            return function (...args) {
                return fn(...args).catch((err) => console.error(err))
            }
        }

        async function go(name, something) {
            console.log('start');
            console.log(`${name} - ${something}`);
            const response = await breathe(600)
            console.log(response);
            const r2 = await breathe(600)
            console.log(r2);
            const r3 = await breathe(600)
            console.log(r3);
            console.log('end')
        }
        const wrappedFunction = catchError(go)
        wrappedFunction("Lucas", "Run ")
        

```


Wrapping a function in a async function 

```js 
  function getCurrentPosition() {
            return new Promise((resolve, reject) => {
                navigator.geolocation.getCurrentPosition(resolve, reject)
            })
        }

        async function go() {
            const position = await getCurrentPosition()
            console.log(position)
        }
        go()
```