Reference of the `softr-workflows` skill: the file behind one `CUSTOM_CODE` node. Adding the node, mapping
placeholders, pushing and publishing are in `SKILL.md`.

# Custom code nodes

The workflows service stores the _body_ of a function and runs it remotely with one variable in scope, `inputData`.
Locally the body lives inside a real function so that editors, linters and agents see valid code. `push` uploads only
the body. `pull` and every `test` regenerate a header above the function.

## The file

JavaScript, `actions/<nodeId>.js`:

```js
// softr: generated header - regenerated on pull, do not edit by hand
// ...rules and one typed line per inputData entry...
// softr: end of generated header

/**
 * @typedef {Object} InputData
 * @property {string} email
 * @property {number} threshold
 */

/** @param {InputData} inputData */
export default async function (inputData) {
  const domain = inputData.email.split('@')[1] ?? '';
  return { qualified: inputData.threshold > 5 && domain !== 'gmail.com', domain };
}
```

Python, `actions/<nodeId>.py`:

```python
# softr: generated header - regenerated on pull, do not edit by hand
# ...
# softr: end of generated header

def main(inputData):
    res = fetch(inputData["url"])
    if not res.ok:
        raise RuntimeError(f"API answered {res.status}")
    return {"count": len(res.json().get("items", []))}
```

Rules the CLI enforces on `push`:

- Exactly one function: `export default async function (inputData)` in JavaScript (a non-async function is accepted
  too), `def main(inputData):` in Python. Anything outside it (imports at the top, helpers, constants) is rejected with
  `file:line:col`. Put helpers inside the function.
- A node still on CUSTOM_CODE 1.0.0 or 1.1.0 gets a plain `function`, and `push` rejects an `async` one. Run
  `softr workflows upgrade <nodeId>` to move it to 1.2.0 (`async`, `fetch`, integrations); the body stays as it is.
- The parameter is named `inputData`.
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
`softr workflows placeholder <nodeId> body <key>`. Return a JSON-compatible value: objects, arrays, strings, numbers,
booleans. A JavaScript `undefined` return is reported as `null`. A thrown error or a raised exception fails the node and the
run; the message shows up in `test`.

## HTTP calls and integrations

Both runtimes offer `fetch(url, options)`; JavaScript awaits it, Python calls it. Options: `method`, `headers`, `body`
(a string as is, `text/plain` unless you set `Content-Type`; an object or array as JSON; `URLSearchParams` form-encoded), `integration` (an alias, see below).
The response has `status`, `ok`, `headers`, `text()`, `json()`, and `arrayBuffer()` / `bytes()`.

The node's `inputs.integrations` lists the workspace integrations the code may call:

```jsonc
"integrations": [
  { "alias": "crm", "integrationId": "9f1c2a3b-4d5e-4f60-8a71-b2c3d4e5f607" }
]
```

A call to a host of an attached integration carries that integration's credentials; the code never sees a token, and
a key must never be written into the code or `inputData`. When two attached integrations share a host, pick one with
`fetch(url, { integration: 'crm' })`. Any other host is called as written, without credentials. A REST API
integration has no provider host, so it authenticates only a call that names its alias. Integration ids: `softr integrations` lists the workspace's connected integrations (or the studio, Workspace settings → Integrations). Every integration the Call API action can authenticate with can be
attached; database connections, Softr Databases, Softr Apps, Telegram and Trello cannot.

## Runtime limits

| Runtime              | Available                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | Not available                                                                         |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| JavaScript (Node 24) | `fetch` with `await`, `crypto` (the `node:crypto` module), `Buffer`, `URL`, `URLSearchParams`, `TextEncoder`/`TextDecoder`, `structuredClone`, timers, `JSON`, `Math`, `Date`, regular expressions                                                                                                                                                                                                                                                                                                                                                                          | `require`, `import`, file system, `process.env` (empty)                               |
| Python 3.12          | `fetch`, class statements, `rsa` (the `rsa` package API: PKCS#1 v1.5 encrypt/decrypt/sign/verify, PKCS#1 or PKCS#8 keys, base64 text accepted), and these modules: `json`, `re`, `math`, `statistics`, `decimal`, `fractions`, `datetime`, `zoneinfo`, `time`, `calendar`, `random`, `secrets`, `collections`, `itertools`, `functools`, `operator`, `string`, `textwrap`, `unicodedata`, `difflib`, `html`, `base64`, `hashlib`, `hmac`, `uuid`, `struct`, `copy`, `heapq`, `bisect`, `typing`, `enum`, `dataclasses`, `csv`, `io` (`StringIO`, `BytesIO`), `urllib.parse` | `os`, `sys`, `subprocess`, `open`, `requests`, `urllib.request`, third-party packages |

Limits: about two minutes per run (then "Execution timed out"), two seconds of synchronous JavaScript or two seconds of
Python CPU time, 20 `fetch` calls per second and 10 open at once, 30 s for a target to answer one `fetch`, 5 MB per request body, about 6 MB per response.

## Test the node

```bash
softr workflows push                 # upload the body
softr workflows test <nodeId>        # run it in the service against the saved upstream samples
softr workflows outputs <nodeId>     # what the next node can reference, with types and examples
```

`test` needs the predecessors' test data; test them first (`test --all` does it in order). A failed test prints the
error from the runtime. After a fix: push, test again. Keep functions small and deterministic; there is no logging,
so return the intermediate values you need to see while developing and trim them before publishing.
