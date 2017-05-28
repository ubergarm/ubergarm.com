## javascript
#### vuejs demo
````vuejs
<script src="https://unpkg.com/vue@2.2.2/dist/vue.min.js"></script>
<div id="app">
  <p>{{ message }}</p>
</div>
<script>
let t0 = performance.now()
const app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue.js!'
  }
})
let t1 = performance.now()
app.message = "Vue.js took " + (t1 - t0).toFixed(2) + " milliseconds."
</script>
````

#### moonjs demo
````moonjs
<script src="https://unpkg.com/moonjs"></script>
<div id="app">
  <p>{{ message }}</p>
</div>
<script>
let t0 = performance.now()
const app = new Moon({
  el: "#app",
  data: {
    message: "Hello Moon.js!"
  }
})
let t1 = performance.now()
let result = "Moon.js took " + (t1 - t0).toFixed(2) + " milliseconds."
app.set("message", result)
</script>
````

#### console log demo
````js
console.log("This has been a test.")
````

## python

#### list comprehension
```python
def foo(val):
    print('How odd {0} seems to me.'.format(val))

_ = [foo(val) for val in range(10) if (val % 2)]
```


