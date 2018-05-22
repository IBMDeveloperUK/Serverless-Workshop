# Calling Other Actions

Using serverless platforms to implement reusable functions means you will often want to invoke one action from another. IBM Cloud Functions provides a [RESTful API](http://petstore.swagger.io/?url=https://raw.githubusercontent.com/openwhisk/openwhisk/master/core/controller/src/main/resources/apiv1swagger.json) to invoke actions programmatically.

Rather than having to [manually construct the HTTP requests](https://github.com/apache/incubator-openwhisk/blob/master/docs/rest_api.md#actions) to invoke actions from within the IBM Cloud Functions runtime, client libraries are pre-installed to make this easier.

These libraries make it simple to invoke other actions, fire triggers and access all other platform services.

#### Proxy Example

Let's look an example of creating a "proxy" action which invokes another action if a "password" is present in the input parameters.

1. Create the following new action `proxy` from the following source files.

```javascript
var openwhisk = require('openwhisk');

function main(params) {
  if (params.password !== 'secret') {
    throw new Error("Password incorrect!")
  }

  var ow = openwhisk();
  return ow.actions.invoke({name: "hello", blocking: true, result: true, params: params})
}
```

*The JavaScript library for Apache OpenWhisk is here: [https://github.com/apache/incubator-openwhisk-client-js/](https://github.com/apache/incubator-openwhisk-client-js/).* *This library is pre-installed in the IBM Cloud Functions runtime and does not need to be manually included.*

```
$ bx wsk action create proxy proxy.js
```

2. Invoke the proxy with an incorrect password.

```
$ bx wsk action invoke proxy -p password wrong -r
{
    "error": "Password is incorrect!"
}
```

3. Invoke the proxy with the correct password.

```
$ bx wsk action invoke proxy -p password secret -p name Bernie -p place Vermont -r
{
    "greeting": "Hello Bernie from Vermont"
}
```

4. Review the activations list to show both actions were invoked.

```
$ bx wsk activation list -l 2
activations
8387302c81dc4d2d87302c81dc4d2dc6 hello
e0c603c242c646978603c242c6c6977f proxy
```

*These libraries can also be used to invoke triggers to fire events from actions.*

### Next Lab
[Asynchronous Actions](/labs/asynchronous-actions.md)