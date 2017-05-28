## Archive
Use vuejs/moonjs and Caddy's [browse](https://caddyserver.com/docs/browse) directive to dynamically generate index.

````moonjs
<script src="https://unpkg.com/moonjs"></script>

<div id="app">
  <ul>
    <li m-for="item in list"><a href="#/blog/archive/{{item.Name.slice(0,-3)}}">{{item.Name}}</a></li>
  </ul>
</div>

<script>
const app = new Moon({
  el: "#app",
  data: {
    list: []
  }
})

let posts = fetch('/blog/archive/', {
  method: 'GET',
  headers: {
    'Accept': 'application/json'
  }
})
.then(function(response) {
  return response.json()
}).then(function(json) {
  app.set('list', json)
}).catch(function(err) {
  console.log('parsing failed', err)
})

</script>
````

## Todo
Make this better!
