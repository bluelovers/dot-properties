# dot-properties

JavaScript `parse()` and `stringify()` for `.properties` (`text/x-java-properties`) files as defined in [java.util.Properties](https://docs.oracle.com/javase/9/docs/api/java/util/Properties.html#load-java.io.Reader-).

To install:
```
npm install dot-properties
```

For usage examples, see [below](#example) or take a look through the project's [test suite](tests/).

## API

### `parse(src[, path])`
Parses an input string read from a .properties file into a JavaScript Object

If the second `path` parameter is true, dots `.` in keys will result in a multi-level object (use a string value to customise). If a parent level is directly assigned a value while it also has a child with an assigned value, the parent value will be assigned to its empty string `''` key. Repeated keys will take the last assigned value. Key order is not guaranteed, but is likely to match the order of the input lines.

### `parseLines(src)`
Splits the input string into an array of logical lines; useful if you want to preserve order, comments and/or empty lines while processing. Used internally by `parse()`.

Key-value pairs are `[key, value]` arrays with string values. Escape sequences in keys and values are parsed. Empty lines are included as empty strings `''`, and comments as strings that start with `#` or `!` characters. Leading whitespace is not included.

### `stringify(input[, options])`
Stringifies a hierarchical object or an array of lines to `.properties` format

If `input` is a hierarchical object, keys will consist of the path parts joined by `.` characters. With array input, string values represent blank or comment lines and string arrays are `[key, value]` pairs. Control characters and `\` will be appropriately escaped. If the `latin1` option is not set to false, all non-Latin-1 characters will also be `\u` escaped.

Output styling is controlled by the second (optional) `options` parameter; by default a spaced `=` separates the key from the value, `\n` is the newline separator, lines are folded at 80 characters (at most, splitting at nice places), with subsequent lines indented by four spaces, and comment lines are prefixed with a `#`. `''` as a key value is considered the default, and set as the value of a key corresponding to its parent object's path:
```js
const defaultOptions = {
  commentPrefix: '# ',  // could also use e.g. '!'
  defaultKey: '',       // YAML uses '='
  indent: '    ',       // tabs are also valid
  keySep: ' = ',        // should have at most one = or :
  latin1: true,         // default encoding for .properties files
  lineWidth: 80,        // use null to disable
  newline: '\n',        // Windows uses \r\n
  pathSep: '.'          // if non-default, use the same in parse()
}
```

## Example

### `example.properties`
```
# You are reading the ".properties" entry.
! The exclamation mark can also mark text as comments.
# The key characters =, and : should be written with a preceding
# backslash to ensure that they are properly loaded. However, there
# is no need to precede the value characters =, and : by a backslash.
website = https://en.wikipedia.org/
language = English
# The backslash below tells the application to continue reading
# the value onto the next line.
message = Welcome to \
          Wikipedia!
# Add spaces to the key
key\ with\ spaces = This is the value that could be looked up with \
                    the key "key with spaces".
# Unicode
tab : \u0009
# If you want your property to include a backslash, it should be
# escaped by another backslash
path c:\\wiki\\templates
# However, some editors will handle this automatically
```
_Source: [Wikipedia](https://en.wikipedia.org/wiki/.properties)_

### `example.js`
```js
const fs = require('fs')
const { parse, parseLines, stringify } = require('dot-properties')

const src = fs.readFileSync('./example.properties', 'utf8')
const obj = parse(src)
console.log('PATH:', obj.path, '\n')

const str = stringify(obj, { lineWidth: 60 })
console.log(`PROPERTIES:\n${str}\n`)

const lines = parseLines(src)
console.log('LINE:', lines[9], '\n')

const str2 = stringify(lines.slice(7, 10))
console.log(`SLICE:\n${str2}\n`)
```

### Console output
```
PATH: c:\wiki\templates

PROPERTIES:
website = https://en.wikipedia.org/
language = English
message = Welcome to Wikipedia!
key\ with\ spaces = This is the value that could be looked \
    up with the key "key with spaces".
tab = \t
path = c:\\wiki\\templates

LINE: [ 'message', 'Welcome to Wikipedia!' ]

SLICE:
# The backslash below tells the application to continue reading
# the value onto the next line.
message = Welcome to Wikipedia!
```
