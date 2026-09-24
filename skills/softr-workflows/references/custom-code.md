Reference of the `softr-workflows` skill: the file behind one `CUSTOM_CODE` node. Adding the node, mapping
placeholders, pushing and publishing are in `SKILL.md`.

# Custom code nodes

The workflows service stores the _body_ of a function and runs it remotely with one variable in scope, `inputData`.
Locally the body lives inside a real function so that editors, linters and agents see valid code. `push` uploads only
the body. `pull` and every `test` regenerate a header above the function.

## The file

JavaScript, `actions/<nodeId>.js`:

```js
// softr-workflows: generated header - regenerated on pull, do not edit by hand
// ...rules and one typed line per inputData entry...
// softr-workflows: end of generated header

/**
 * @typedef {Object} InputData
 * @property {string} email
 * @property {number} threshold
 */

/** @param {InputData} inputData */
export default function (inputData) {
  const domain = inputData.email.split('@')[1] ?? '';
  return { qualified: inputData.threshold > 5 && domain !== 'gmail.com', domain };
}
```

Python, `actions/<nodeId>.py`:

```python
# softr-workflows: generated header - regenerated on pull, do not edit by hand
# ...
# softr-workflows: end of generated header

def main(inputData):
    import json
    from urllib import request

    with request.urlopen(inputData["url"], timeout=10) as response:
        payload = json.load(response)
    return {"count": len(payload.get("items", []))}
```

Rules the CLI enforces on `push`:

- Exactly one function: `export default function (inputData)` in JavaScript, `def main(inputData):` in Python.
  Anything outside it (imports at the top, helpers, constants) is rejected with `file:line:col`. Put helpers inside the
  function.
- The parameter is named `inputData`. An `async` function is rejected: the runtime is synchronous.
- No workflow placeholders such as `{outputs.x:::$.y}` anywhere in the file. Map them in `inputData`.
- Do not edit the generated header. It is rewritten on every `pull`, `test` and `outputs --refresh` and is never
  uploaded.

## Input

`inputData` is the node's `inputs.inputData` map from `softr-workflows.jsonc`, resolved at run time:

```jsonc
"inputData": {
  "email": "{outputs.trigger:::$.body.email}",   // whole-value placeholder: keeps the JSON type
  "items": "{outputs.fetch:::$.body.items}",     // an array stays an array
  "threshold": 5,                                // literal
  "label": "Lead {outputs.trigger:::$.body.id}"  // text with a placeholder: becomes a string
}
```

The header types every entry from the saved test outputs of the referenced nodes: trusted when seen in test data,
marked when taken from declared metadata only, and `unknown` with a hint when the upstream node was never tested. When
the type reads `unknown`, test the upstream node before writing code against it.

## Output

The returned value becomes the node output at `$.body`; the service wraps it as `{ "statusCode": 200, "body": ... }`.
Later nodes reference it as `{outputs.<nodeId>:::$.body}` or a field of it, built with
`softr-workflows placeholder <nodeId> body <key>`. Return a JSON-compatible value: objects, arrays, strings, numbers,
booleans. A JavaScript falsy return is reported as `null`. A thrown error or a raised exception fails the node and the
run; the message shows up in `test`.

## Runtime limits

| Runtime             | Available                                                                                | Not available                                                                       |
| ------------------- | ---------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| JavaScript (ES2020) | Plain language features, `JSON`, `Math`, `Date`, regular expressions, arrays and strings | `fetch`, `require`, `import`, `Buffer`, `URL`, `crypto`, `await`, timers, Node APIs |
| Python 3.13         | The standard library, imported inside `main`; `urllib` for HTTP calls                    | Third-party packages, `pip`                                                         |

So an HTTP call from code is Python-only. For JavaScript workflows, call the API with a `CALL_API` node before the code
node and map its output into `inputData`.

## Test the node

```bash
softr-workflows push                 # upload the body
softr-workflows test <nodeId>        # run it in the service against the saved upstream samples
softr-workflows outputs <nodeId>     # what the next node can reference, with types and examples
```

`test` needs the predecessors' test data; test them first (`test --all` does it in order). A failed test prints the
error from the runtime. After a fix: push, test again. Keep functions small and deterministic; there is no logging,
so return the intermediate values you need to see while developing and trim them before publishing.
