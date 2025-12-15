# Grammar

_What you will find:_ a formal description of directive value shapes and handler syntax as implemented in Sercrod v1.06. This page is normative, concise, and tutorial-free.

## 0. Legend (tokens and notation)

- `Expr` - a JavaScript expression evaluated by the runtime (statements are out of scope).
- `Ident` - a JavaScript identifier (ASCII suggestion; Unicode identifiers follow JS rules).
- `String` - an attribute string literal shown as `"..."` here (actual HTML quoting rules apply).
- `LValue` - a writable property reference inside host data:  
  `LValue ::= Ident { ('.' Ident) | ('[' Expr ']') }`
- `Mods` - event modifier list: zero or more `.`-prefixed modifier names.
- `Element(attr)` - an element node that declares the attribute described by `attr`.
- `{ ... }` means repeat zero or more times. `[ ... ]` means optional. Literal commas are required where shown.
- All directive names are lowercase and case-sensitive.
- Every `*name` directive has an equivalent `n-name` form (not repeated below).
- Notation: this document mixes EBNF-style `{}` and `[]` with the BNF-style definition symbol `::=`.

## 1. Common rules

- Attribute values are parsed as strings and then interpreted per rule below.
- Whitespace around separators is permitted unless a rule says otherwise.
- Truthiness and comparison use JavaScript semantics unless noted (for example, `*switch` cases use `===`).
- Value-less directives are written without `="..."` and are equivalent to the presence of the flag.

## 2. Control flow

```
IfChain           ::= IfHead { ElseIf } [ Else ]
IfHead            ::= Element('*if="' Expr '"')
ElseIf            ::= Element('*elseif="' Expr '"')
Else              ::= Element('*else')

SwitchGroup       ::= SwitchHead { CaseClause }
SwitchHead        ::= Element('*switch="' Expr '"')
CaseClause        ::= Element('*case="' Expr '"')
                    | Element('*case.break="' Expr '"')
                    | Element('*default')
```

Constraints:
- `IfChain` is formed by contiguous sibling elements under the same parent: one `IfHead`, followed by zero or more `ElseIf`, and at most one `Else`. The chain ends at the first sibling that is not `ElseIf` or `Else`.
- `SwitchGroup` is formed by one `SwitchHead`, followed by contiguous sibling elements that are any of `CaseClause`. The group ends at the first sibling that is not a `CaseClause`.
- Matching in `SwitchGroup` uses strict equality (`===`).

## 3. Iteration

```
Each              ::= '*each="' Expr ' as ' Ident [ ',' Ident ] '"'
For               ::= '*for="' Expr '"'
```

Notes:
- In `Each`, the first `Ident` binds the item. The optional second `Ident` binds the index or key.

## 4. Scope and variables

```
Let               ::= '*let="' AssignList '"'
Global            ::= '*global="' AssignList '"'
Literal           ::= '*literal="' Expr '"'
Rem               ::= '*rem="' String '"'

AssignList        ::= Assign { ',' Assign }
Assign            ::= LHS '=' Expr
LHS               ::= LValue
```

Notes:
- During `*let` evaluation, `$parent` is injected (see expressions reference).

## 5. Input and staging

```
Input             ::= '*input="' LValue '"'
Lazy              ::= '*lazy'
Eager             ::= '*eager'
Stage             ::= '*stage'
Apply             ::= '*apply'
Restore           ::= '*restore'
Save              ::= '*save'
Load              ::= '*load'
```

Notes:
- `*input` requires an `LValue` (pure expressions are not valid here).

## 6. Output

```
Print             ::= '*print="' Expr '"'
Compose           ::= '*compose="' Expr '"'
TextCont          ::= '*textContent="' Expr '"'
InnerHTML         ::= '*innerHTML="' Expr '"'
```

## 7. Attributes and shorthands

```
ClassBind         ::= ':class="' Expr '"'   | 'n-class="' Expr '"'
StyleBind         ::= ':style="' Expr '"'   | 'n-style="' Expr '"'
```

## 8. Communication (HTTP/API)

```
FetchEmpty        ::= '*fetch'
FetchExpr         ::= '*fetch="' Expr '"'
Post              ::= '*post="' Expr '"'
Api               ::= '*api="' Expr '"'
Into              ::= '*into="' Ident '"'
```

Notes:
- `*into` accepts identifier only. Dynamic names via expressions are not supported.

## 9. Files

```
Upload            ::= '*upload="' Expr '"'
Download          ::= '*download="' Expr '"'
```

## 10. WebSocket

```
WebSocket         ::= '*websocket="' Expr '"'
WsSend            ::= '*ws-send="' Expr '"'
```

Notes:
- `Expr` resolves to a URL string or an implementation-defined config object.

## 11. Events and modifiers

```
Handler           ::= '@' EvName Mods? '="' Expr '"'
EvName            ::= Name
Name              ::= ASCII letter followed by { ASCII letter | digit | '-' | ':' }
Mods              ::= { '.' Mod }
Mod               ::= 'prevent' | 'stop' | 'once' | 'capture' | 'passive' | 'update' | 'noupdate'
```

Notes:
- `EvName` must not contain `.` (dot) - dots are reserved for modifiers.
- Modifiers are order-independent. Duplicates are ignored.
- `update` and `noupdate` are mutually exclusive.

## 12. Strict mode

`*strict` is a compact and rigid call syntax. The runtime performs a single-pass tokenizer with tight validation. The grammar and constraints below reflect the implementation.

### 12.1 Grammar

```
Strict            ::= '*strict="' StrictCall '"'

StrictCall        ::= FnName '(' [ ArgList ] [ Callback ] ')'

FnName            ::= Letter { Letter | Digit | '_' }
ArgList           ::= Arg { ',' Arg }
Arg               ::= IdentLike
IdentLike         ::= [A-Za-z0-9_]+    // must not start with '_'

Callback          ::= ',' '()=>{' CbBody '}' 
                    | ',' '(' [WS] Param [WS] ')' [WS] '=>' [WS] '{' CbBody '}'

Param             ::= IdentLike        // must not start with '_'
CbBody            ::= any chars with balanced braces, length-limited
WS                ::= space or tab characters only

// Whitespace rules - see 12.2
```

### 12.2 Whitespace and ordering rules

- No leading whitespace at the beginning of the `*strict` value.
- No whitespace between `FnName` and `(`. Example: `fn(` is valid, `fn (` is invalid.
- No whitespace inside `ArgList`. The following are valid: `fn(a,b,c)` and `fn()`. The following are invalid: `fn(a, b)`, `fn( a,b )`.
- The optional callback must be introduced with a comma immediately after the last argument. Only two forms are accepted:
  - `,()=>{ ... }` - no whitespace between comma and `()=>`.
  - `,(x)=>{ ... }` - no whitespace between comma and `(`. Inside this callback header, limited whitespace is allowed at the marked `WS` positions: around the parameter and around `=>`, and before `{`.
- After the callback block, optional whitespace is allowed before the final `)`.
- After the final `)`, no trailing characters are allowed.

### 12.3 Identifier and argument rules

- `FnName` must start with a letter. It can contain letters, digits, and underscore. It must not start with `_`.
- Each `Arg` token consists of one or more characters in `[A-Za-z0-9_]`. It must not start with `_`.
- Property access and indexing are not allowed in arguments. Dots and brackets are rejected. For example: `user` is allowed, `user.name` and `user['name']` are invalid.
- Each argument name is resolved from the current effective scope. If the resolved value is a function, the call is rejected for that argument.

### 12.4 Limits and errors

- Maximum total expression length: 256 characters. Exceeding this fails fast.
- Maximum number of arguments: 16.
- Maximum callback body length: 2048 characters.
- An empty argument between commas is an error.
- Unbalanced or malformed callback braces are errors.
- Unrecognized characters in `FnName` or arguments are errors.

### 12.5 Resolution order

The function is resolved in this order:
1. `this.fnRegistry[FnName]`
2. `this.constructor.strictRegistry[FnName]`
3. `Sercrod.strictRegistry[FnName]` (shared reference to `window.__Sercrod_strict`)
4. Compatibility fallback: `this.constructor.fnRegistry[FnName]`
5. Final fallback: `globalThis[FnName]`

### 12.6 Examples

Valid:
- `*strict="doThing(a,b)"`  
- `*strict="doThing()"`  
- `*strict="doThing(a,()=>{ x = a })"`  
- `*strict="doThing(a,(x)=>{ x += 1 })"`

Invalid:
- `*strict=" doThing(a)"` - leading space
- `*strict="doThing (a)"` - space before `(`
- `*strict="doThing(a, b)"` - space inside argument list
- `*strict="doThing(a, () => { })"` - spaces after comma and around `=>`
- `*strict="doThing(user.name)"` - property access not allowed
- `*strict="doThing(_a)"` - identifier starts with underscore
- `*strict=""` - empty value

## 13. Shorthand equivalence

- Every directive appears in two equivalent spellings: `*name` and `n-name`.  
  Example: `*if="Expr"` ⇔ `n-if="Expr"`.

## 14. Non-goals of this grammar

- It does not describe expression semantics, filters, scheduler behavior, or event payloads.  
  See: [Expressions](./expressions.md), [Filters](./filters.md), [Lifecycle](./lifecycle.md), and [Events](./events.md).

---
Back to index: [`README.md`](./README.md)
