# Asynchronous actions

#### Returning asynchronous results

JavaScript functions that run asynchronously may need to return the activation result after the `main` function has returned. You can accomplish this by returning a Promise in your action.

1. Save the following content in a file called `asyncAction.js`.

```
function main(args) {
     return new Promise(function(resolve, reject) {
       setTimeout(function() {
         resolve({ done: true });
       }, 2000);
    })
 }
```

Notice that the `main` function returns a Promise, which indicates that the activation hasn't completed yet, but is expected to in the future.

The `setTimeout()` JavaScript function in this case waits for two seconds before calling the callback function. This represents the asynchronous code and goes inside the Promise's callback function.

The Promise's callback takes two arguments, resolve and reject, which are both functions. The call to `resolve()`fulfills the Promise and indicates that the activation has completed normally.

A call to `reject()` can be used to reject the Promise and signal that the activation has completed abnormally.

#### Testing asynchronous timeouts

1. Run the following commands to create the action and invoke it:

```
$ bx wsk action create asyncAction asyncAction.js
```

```
$ bx wsk action invoke --result asyncAction
{
    "done": true
}
```

Notice that you performed a blocking invocation of an asynchronous action.

2. Fetch the activation log to see how long the activation took to complete:

```
$ bx wsk activation list --limit 1 asyncAction
activations
b066ca51e68c4d3382df2d8033265db0             asyncAction
```

```
$ bx wsk activation get b066ca51e68c4d3382df2d8033265db0
{
     "duration": 2026,
     ...
}
```

Checking the `duration` field in the activation record, you can see that this activation took slightly over two seconds to complete.

**Actions have a `timeout` parameter that enforces the maximum duration for an invocation.** This value defaults to 60 seconds and can be changed to a maximum of 5 minutes.

Let's look at what happens when an action invocation takes longer than the `timeout`.

1. Update the `asyncAction` timeout to 1000ms.

```
$ bx wsk action update asyncAction --timeout 1000
ok: updated action asyncAction
```

2. Invoke the action and block on the result.

```
$ bx wsk action invoke asyncAction --result
{
    "error": "The action exceeded its time limits of 1000 milliseconds."
}
```

The error message returned by the platform indicates the action didn't return a response within the user-specified timeout. If we change the `timeout` back to a value higher than the artificial delay in the function, it should work again.

1. Update the `asyncAction` timeout to 10000ms.

```
$ bx wsk action update asyncAction --timeout 10000
ok: updated action asyncAction
```

2. Invoke the action and block on the result.

```
$ bx wsk action invoke asyncAction --result
{
    "done": true
}
```

🎉🎉🎉 **Asynchronous actions are necessary for calling other APIs or cloud services. Don't forget about that timeout though! Let's have a look at using an asynchronous action to invoke another API…** 🎉🎉🎉

#### Using actions to call an external API

The examples so far have been simple functions. You can also create an action that call external APIs and services.

This example invokes a Yahoo Weather service to get the current conditions at a specific location.

1. Save the following content in a file called `weather.js`.

```javascript
var request = require('request');

function main(params) {
    var location = params.location || 'Vermont';
    var url = 'https://query.yahooapis.com/v1/public/yql?q=select item.condition from weather.forecast where woeid in (select woeid from geo.places(1) where text="' + location + '")&format=json';

    return new Promise(function(resolve, reject) {
        request.get(url, function(error, response, body) {
            if (error) {
                reject(error);
            }
            else {
                var condition = JSON.parse(body).query.results.channel.item.condition;
                var text = condition.text;
                var temperature = condition.temp;
                var output = 'It is ' + temperature + ' degrees in ' + location + ' and ' + text;
                resolve({msg: output});
            }
        });
    });
}
```

Note that the action in the example uses the JavaScript `request` library to make an HTTP request to the Yahoo Weather API, and extracts fields from the JSON result. The [References](https://github.com/apache/incubator-openwhisk/blob/master/docs/reference.md#javascript-runtime-environments) detail the Node.js packages that you can use in your actions.

This example also shows the need for asynchronous actions. The action returns a Promise to indicate that the result of this action is not available yet when the function returns. Instead, the result is available in the `request` callback after the HTTP call completes, and is passed as an argument to the `resolve()` function.

1. Run the following commands to create the action and invoke it:

```
$ bx wsk action create weather weather.js
```

```
$ bx wsk action invoke --result weather --param location "Brooklyn, NY"
{
 "msg": "It is 28 degrees in Brooklyn, NY and Cloudy"
}   
```

🎉🎉🎉 **This is more like a real-world example. This action could be invoked thousands of times in parallel and it would just work! No having to manage infrastructure to build a scalable cloud API.** 🎉🎉🎉

### Next Lab
[Using Action Sequences](/labs/using-action-sequences.md)
