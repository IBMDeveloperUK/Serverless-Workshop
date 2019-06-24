# Packaging an action as a Node.js module

***This step requires you to have the Node.js and NPM development tools installed.***

As an alternative to writing all your action code in a single JavaScript source file, you can write an action as a `npm` package. Consider as an example a directory with the following files:

First, `package.json`:

```
{
  "name": "my-action",
  "main": "index.js",
  "dependencies" : {
    "left-pad" : "1.1.3"
  }
}
```

Then, `index.js`:

```
function myAction(args) {
    const leftPad = require("left-pad")
    const lines = args.lines || [];
    return { padded: lines.map(l => leftPad(l, 30, ".")) }
}

exports.main = myAction;
```

Note that the action is exposed through `exports.main`; the action handler itself can have any name, as long as it conforms to the usual signature of accepting an object and returning an object (or a `Promise` of an object). Per Node.js convention, you must either name this file `index.js` or specify the file name you prefer as the `main`property in package.json.

To create an OpenWhisk action from this package:

1. Install first all dependencies locally

```
$ npm install
```

2. Create a `.zip` archive containing all files (including all dependencies):

```
$ zip -r action.zip *
```

> Please note: Using the Windows Explorer action for creating the zip file will result in an incorrect structure. OpenWhisk zip actions must have `package.json` at the root of the zip, while Windows Explorer will put it inside a nested folder. The safest option is to use the command line `zip` command as shown above.

3. Create the action:

```
$ ibmcloud wsk action create packageAction action.zip --kind nodejs:default
```

Note that when creating an action from a `.zip` archive using the CLI tool, you must explicitly provide a value for the `--kind` flag.

4. You can invoke the action like any other:

```
$ ibmcloud wsk action invoke --result packageAction --param lines "[\"and now\", \"for something completely\", \"different\" ]"
{
    "padded": [
        ".......................and now",
        "......for something completely",
        ".....................different"
    ]
}
```

Finally, note that while most `npm` packages install JavaScript sources on `npm install`, some also install and compile binary artifacts. The archive file upload currently does not support binary dependencies but rather only JavaScript dependencies. Action invocations may fail if the archive includes binary dependencies.

🎉🎉🎉 **Node.js has a huge ecosystem of third-party packages which can be used on OpenWhisk with this method. The platform does come built-in with popular packages listed [here](https://github.com/apache/incubator-openwhisk/blob/master/docs/reference.md#javascript-runtime-environments) This method also works for any other runtime.** 🎉🎉🎉

### Next Lab
[Managing actions with packages](/labs/managing-actions-with-packages.md)