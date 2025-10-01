# Custom Clojure Queries for Helix Editor

This repo contains my customisations to make editing Clojure in Helix more comfortable.

## Installation

For textobject and indent queries:

- Copy the Clojure language section from `./helix/languages.toml` into your own `~/.config/helix/languages.toml`.
- Copy `./helix/runtime/grammars/clojure.so` to your `~/.config/helix/runtime/grammars` folder.
- Copy `./helix/runtime/queries/clojure/*.scm` to your `~/.config/helix/runtime/queries/clojure` folder.

For ersatz repl integration:

- Install reple from my fork at https://github.com/waddie/reple/tree/fix/rlwrap
- Copy the keymap entries from `./helix/config.toml` to your own `~/.config/helix/config.toml`, or create your
  own equivalents to suit your preferences.

## reple

Jeka’s `reple` (https://github.com/j3ka/reple) is a basic workaround for Helix’s current lack of a plugin
system, which prevents a proper repl integration like CIDER for emacs or Calva for vscode. It lets you
send a form to a repl to evaluated.

`clj` wraps `clojure` in `rlwrap` to provide history, etc. When spawned with reple, it fails to launch
because the terminal dimensions aren’t passed through. You can use plain `clojure`, but then you don’t get
history. In practice, you will want to use the repl directly because the integration is so simple.

My fork simply adds reporting of the terminal size so `clj` will start properly.

To use it with the keymap included, simply select the entire form and hit Alt-Enter to send it to the repl.

## tree-sitter queries

This fork of the https://github.com/sogaiu/tree-sitter-clojure Clojure grammar includes support for
parsing map entries as key/value pairs instead of single elements. This allows us to match around the
key/value pair in Helix.

If you prefer to build from source, you can find my branch here:
https://github.com/waddie/tree-sitter-clojure/tree/map-kv-pairs

textobjects.scm enables the following:

### match inside/around function (mif/maf)

- **Inside**: Function body only, excluding keyword, name, and parameter vector
- **Around**: Entire form including opening/closing parens and keyword

- **Supported keywords**:
  - `defn`, `defn-`, `defmacro`, `defmethod`, `defmulti`, `definline`
  - `fn`
  - `#()`

### match inside/around type (mit/mat)

- **Inside**: Everything after the type name
- **Around**: Entire form including opening/closing parens and keyword

- **Supported keywords**:
  - `deftype`, `defrecord`, `defprotocol`, `definterface`, `defstruct`

### match inside/around test (miT/maT)

- **Inside**: Everything after the test name
- **Around**: Entire form including opening/closing parens and keyword

- **Supported keywords**:
  - `deftest`

### match inside/around parameter (mia/maa)

- **Inside**: All parameters within the vector, excluding brackets
- **Around**: Entire parameter vector including brackets

**Note**: Works the same with any vec.

### match inside/around data structure entry (mie/mae)

For lists, vectors and sets:

- **Around**/**Inside**: Individual elements within collections

For maps:

- **Inside**: Key or value
- **Around**: Key/value pair

- **Supported collections**:
  - Lists: `(a b c)`
  - Vectors: `[a b c]`
  - Maps: `{:a 1 :b 2}`
  - Sets: `#{a b c}`

### Match inside/around Comment (mic/mac)

- **Inside**: Comment content excluding delimiters
- **Around**: Entire comment form or consecutive comment lines

- **Supported forms**:
  - Line comments: `; comment`
  - Discard expressions: `#_(+ 1 2)`
  - Comment special form: `(comment ...)`

indents.scm enables the following:

### Special Forms (indent body by one level)

Forms where the body is indented after the name/bindings:

- **Definitions**: `def`, `defn`, `defn-`, `defmacro`, `defmethod`, `defmulti`, `defonce`,
`defprotocol`, `deftype`, `defrecord`, `defstruct`, `definline`, `definterface`, `deftest`
- **Namespace**: `ns`
- **Bindings**: `let`, `letfn`, `binding`, `loop`
- **Iteration**: `for`, `doseq`, `dotimes`
- **Conditional bindings**: `when-let`, `if-let`, `when-some`, `if-some`
- **Resource management**: `with-*` (any symbol starting with "with-")
- **Functions**: `fn`

### Alignment Rules

1. **Two elements on first line**: Align to second element (excludes special forms above)
2. **First element is a list on its own line**: Align outer list to inner list
3. **Literal as first element**: Align to the literal (booleans, nil, strings, numbers, keywords)
4. **Vector/Map with two elements on first line**: Align subsequent elements (e.g., let bindings)

### Standard Indentation

- Vectors, maps, and sets are indented by one level
- Lists not matching special cases above are indented by one level
