# Using Action Sequences

OpenWhisk supports a special kind of action called a "sequence". These actions are created using a list of existing actions. When the sequence is invoked, each action in executed in a sequence. The input parameters are passed to the first action in the sequence. The output from that function is passed as the input to the next function and so on. The output from the last action in the sequence is returned as the response result.

Here's an example of defining a sequence (`my_sequence`) which will invoke three actions (`a, b, c`).

```
$ ibmcloud wsk action create my_sequence --sequence a,b,c
```

*Sequences behave like normal actions, you create, invoke and manage them as normal through the CLI.*

Let's look at an example of using sequences.  

1. Create the file (`funcs.js`) with the following contents:

```javascript
function split(params) {
  var text = params.text || ""
  var words = text.split(' ')
  return { words: words }
}

function reverse(params) {
  var words = params.words || []
  var reversed = words.map(word => word.split("").reverse().join(""))
  return { words: reversed }
}

function join(params) {
  var words = params.words || []
  var text = words.join(' ')
  return { text: text }
}
```

2. Create the following three actions

```
$ ibmcloud wsk action create split funcs.js --main split
$ ibmcloud wsk action create reverse funcs.js --main reverse
$ ibmcloud wsk action create join funcs.js --main join
```

#### Creating Sequence Actions

1. Test each action to verify it is working

```
$ ibmcloud wsk action invoke split --result --param text "Hello world"
{
    "words": [
        "Hello",
        "world"
    ]
}
$ ibmcloud wsk action invoke reverse --result --param words '["hello", "world"]'
{
    "words": [
        "olleh",
        "dlrow"
    ]
}
$ ibmcloud wsk action invoke join --result --param words '["hello", "world"]'
{
    "text": "hello world"
}
```

2. Create the following action sequence.

```
$ ibmcloud wsk action create reverse_words --sequence split,reverse,join
```

3. Test out the action sequence.

```
$ ibmcloud wsk action invoke reverse_words --result --param text "hello world"
{
    "text": "olleh dlrow"
}
```

Using sequences is a great way to develop re-usable action components that can be joined together into "high-order" actions to create serverless applications.

For example, what if you have serverless functions to implement an external API and want to enforce HTTP authentication? Rather than manually adding this code to every action, you could define an "auth" action and use sequences to define new "authenticated" actions by joining this action with the existing API actions.

🎉🎉🎉 **Sequences are an "advanced" OpenWhisk technique. Congratulations for getting this far!** 🎉🎉🎉

### Next Lab
[Packaging an action as a Node.js module](/labs/packaging-an-action-as-a-nodejs-module.md)
