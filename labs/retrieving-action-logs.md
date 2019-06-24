# Retrieving Action Logs

Application logs are essential to debugging production issues. In IBM Cloud Functions, all output written by  to `stdout` and `stderr` by actions is available in the activation records.

#### Creating Activation Logs

1. Create a new action (`logs`) from the following source files.

```javascript
function main(params) {
    console.log("function called with params", params)
    console.error("this is an error message")
    return { result: true }
}
```

```
$ ibmcloud wsk action create logs logs.js
ok: created action logs
```

2. Invoke the `logs` action to generate some logs.

```
$ ibmcloud wsk action invoke -r logs -p hello world
{
    "result": true
}
```

#### Accessing Activation Logs

1. Retrieve activation record to verify logs have been recorded.

```
$ ibmcloud wsk activation get --last
ok: got activation 9fc044881705479580448817053795bd
{    
    ...   
    "logs": [
        "2018-03-02T09:49:03.021Z stdout: function called with params { hello: 'world' }",
        "2018-03-02T09:49:03.021Z stderr: this is an error message"
    ],
    ...
}
```

3. Logs can also be retrieved without showing the whole activation record, using the `activation logs` command.

```
$ ibmcloud wsk activation logs --last
2018-03-02T09:49:03.021404683Z stdout: function called with params { hello: 'world' }
2018-03-02T09:49:03.021816473Z stderr: this is an error message
```

#### Polling Activation Logs

Activation logs can be monitored in real-time, rather than manually retrieving individual activation records.

1. Run the following command to monitor logs from the `logs` actions.

```
$ ibmcloud wsk activation poll
Enter Ctrl-c to exit.
Polling for activation logs
```

2. In another terminal, run the following command multiple times.

```
$ ibmcloud wsk action invoke logs -p hello world
ok: invoked /_/logs with id 0e8d715393504f628d715393503f6227
```

3. Check the output from the `poll` command to see the activation logs.

```

Activation: 'logs' (ae57d06630554ccb97d06630555ccb8b)
[
    "2018-03-02T09:56:17.8322445Z stdout: function called with params { hello: 'world' }",
    "2018-03-02T09:56:17.8324766Z stderr: this is an error message"
]

Activation: 'logs' (0e8d715393504f628d715393503f6227)
[
    "2018-03-02T09:56:20.8992704Z stdout: function called with params { hello: 'world' }",
    "2018-03-02T09:56:20.8993178Z stderr: this is an error message"
]

Activation: 'logs' (becbb9b0c37f45f98bb9b0c37fc5f9fc)
[
    "2018-03-02T09:56:44.6961581Z stderr: this is an error message",
    "2018-03-02T09:56:44.6964147Z stdout: function called with params { hello: 'world' }"
]
```

### Next Lab
[Calling Other Actions](/labs/calling-other-actions.md)