# Using Action Parameters

Event parameters can be passed to the action when it is invoked. Let's look at a sample action which uses the parameters to calculate the return values.

#### Passing parameters to an action (Node.js)

1. Update the file `hello.js` with the following following content:

```
function main(params) {
    return {payload:  'Hello, ' + params.name + ' from ' + params.place};
}
```

The input parameters are passed as a JSON object parameter to the `main` function. Notice how the `name` and `place` parameters are retrieved from the `params` object in this example.

2. Update the `hello` action with the new source code.

```
$ bx wsk action update hello hello.js
```

#### Invoking action with parameters

When invoking actions through the command-line, parameter values can be passed as through explicit command-line parameters `—param/-p` or using an input file containing raw JSON.

3. Invoke the `hello` action using explicit command-line parameters.

```
$ bx wsk action invoke --result hello --param name Bernie --param place Vermont
{
    "payload": "Hello, Bernie from Vermont"
}
```

4. Create a file (`parameters.json`) containing the following JSON.

```
{
    "name": "Bernie",
    "place": "Vermont"
}
```

5. Invoke the `hello` action using parameters from a JSON file.

```
$ bx wsk action invoke --result hello --param-file parameters.json
{
    "payload": "Hello, Bernie from Vermont"
}
```

*Notice the use of the `--result` option: it implies a blocking invocation where the CLI waits for the activation to complete and then displays only the result. For convenience, this option may be used without `--blocking` which is automatically inferred.*

Parameter values can be any valid JSON value, including nested objects. Let's update our action to use child properties of the event parameters.

6. Create the `hello-person` action with the following source code.

```javascript
function main(params) {
    return {payload:  'Hello, ' + params.person.name + ' from ' + params.person.place};
}
```

Now the action expects a single `person` parameter to have fields `name` and `place`.

7. Invoke the action with a single `person` parameter that is valid JSON.

```
$ bx wsk action invoke --result hello-person -p person '{"name": "Bernie", "place": "Vermont"}'
```

The result is the same because the CLI automatically parses the `person` parameter value into the structured object that the action now expects:

```
{
    "payload": "Hello, Bernie from Vermont"
}
```

🎉🎉🎉 **That was pretty easy, huh? We can now pass parameters and access these values in our serverless functions. What about parameters that we need but don't want to manually pass in every time? Guess what, we have a trick for that…** 🎉🎉🎉

#### Setting default parameters

Actions can be invoked with multiple named parameters. Recall that the `hello` action from the previous example expects two parameters: the *name* of a person, and the *place* where they're from.

Rather than pass all the parameters to an action every time, you can bind default parameters. Default parameters are stored in the platform and automatically passed in during each invocation. If the invocation includes the same event parameter, this will overwrite the default parameter value.

Let's use the `hello` action from our previous example and bind a default value for the `place` parameter.

1. Update the action by using the `—param` option to bind default parameter values.

```
$ bx wsk action update hello --param place Vermont
```

Passing parameters from a file requires the creation of a file containing the desired content in JSON format. The filename must then be passed to the `-param-file` flag:

Example parameter file called parameters.json:

```json
{
    "place": "Vermont"
}
```

```
$ bx wsk action update hello --param-file parameters.json
```

2. Invoke the action, passing only the `name` parameter this time.

```
$ bx wsk action invoke --result hello --param name Bernie
```

```
{
    "payload": "Hello, Bernie from Vermont"
}
```

Notice that you did not need to specify the place parameter when you invoked the action. Bound parameters can still be overwritten by specifying the parameter value at invocation time.

3. Invoke the action, passing both `name` and `place` values. The latter overwrites the value that is bound to the action.

```
$ bx wsk action invoke --result hello --param name Bernie --param place "Washington, DC"
{  
    "payload": "Hello, Bernie from Washington, DC"
}
```

🎉🎉🎉 **Default parameters are awesome for handling parameters like authentication keys for APIs. Letting the platform pass them in automatically means you don't have include these keys in invocation requests or include them in the action source code. Neat, huh?** 🎉🎉🎉

### Next Lab
[Retrieving Action Logs](/labs/retrieving-action-logs.md)