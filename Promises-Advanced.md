# Promises Advanced
#### Содержание 
* Control promises flow with returning `Promise.resolve` & `Promise.reject`
* Dont make nested "then" 
* Make delay to continue
* Insert a few records with check of previous
* Make a chain with promises
--- 

#### Links
* [9 полезных советов по Promise.resolve и Promise.reject](https://proglib.io/p/9-js-promise-advice/)
* [CSS tricks](https://css-tricks.com/why-using-reduce-to-sequentially-resolve-promises-works/)
* [HeavyMetalCoder](https://www.heavymetalcoder.com/make-array-foreach-synchronous-even-with-an-asynchronous-body/)


`array.reduce(callback[, initialValue])`
* callback - Функция, выполняющаяся для каждого элемента массива, принимает четыре аргумента:
   + accumulator - Аккумулятор, аккумулирующий значение, которое возвращает функция callback после посещения очередного элемента, либо значение initialValue, если оно предоставлено (смотрите пояснения ниже).
   + currentValue - Текущий обрабатываемый элемент массива.
   + index [Необязательный] - Индекс текущего обрабатываемого элемента массива.
   + array [Необязательный] - Массив, для которого была вызвана функция reduce

---

### Control promises flow with returning `Promise.resolve` & `Promise.reject`
```js
new Promise((res, rej) => { rej('here') })

.catch(err => {
  if (err.statusCode === 400) {
    return Promise.reject(err)
  }
  return Promise.resolve(err)
})

.then(r => console.log('then'))

.catch(err => console.log('catch'));
```

### Dont make nested "then"
```js
// Wrong
new Promise((res, rej) => { res() })
.then(() => {
 somePromise()
   .then((data) => console.log(data)) 
})
.then((data) => {
 console.log(data) // "Hello"
})
.catch((err) => console.log('Error occured'))
```

```js
// Right
new Promise((res, rej) => { res() })
.then(() => somePromise())
.then((data) => {
 console.log(data) // "somePromise" answer
})
.catch((err) => console.log('Error occured'))
```

### Make delay to continue
```js
new Promise((r, j) => setTimeout(r, 5000))
.then(...)
```

### Insert a few records with check of previous
We want to insert a few records, but want make it sequntially and check if previous inserted well, kinda transaction.
```js
function methodThatReturnsAPromise(id) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log(`Processing ${id}`);
      resolve(id);
    }, 1000*id);
  });
}

let result = [2,1,3].reduce( (accumulatorPromise, nextID) => {
  return accumulatorPromise.then((id) => {
    // here we can do check if insertion was correct
    // first will be undefined cuz initialValue is Promise.resolve()
    console.log('Id Of inserted item = ', id)

    return methodThatReturnsAPromise(nextID)
  });
}, Promise.resolve());

result.then(e => {
  console.log("Resolution is complete! Let's party.")
});

```


### Make a chain with promises
```js
// Manually created chain 
Promise.resolve()
       .then(myAsyncFunction(1))
       .then(myAsyncFunction(2))
       .then(myAsyncFunction(3));
```

```js
// Equal to above, created with "reduce"       
[1, 2, 3].reduce((promiseChain, arrayItem) =>
  promiseChain.then(() => myAsyncFunction(arrayItem)), Promise.resolve());
```

```js
// More verbose
[2, 1, 3].reduce(
  // callback
  (promiseChain, arrayItem) => {
    return promiseChain.then(() => console.log(arrayItem))
  },

  // initialValue
  Promise.resolve()
)
```