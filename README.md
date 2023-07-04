# Inko Regex

This is a simple pure-inko regular expression engine that covers the most common
regular expression matching cases.

## Current Status

Supports the most basic regex operations. Probably works correctly
much of the time, but I expect there to be quite a few bugs. I've
written comprehensive unit tests for all the basic cases, but haven't
fully tested the combinations of possible states.

I haven't done any deliberate performance optimizations, but it is designed—like all modern regex
engines—to run in linear time.

## Installation

The usual way to install inko packages:

```
inko pkg init # if inko.pkg doesn't exist
inko pkg add github.com/dusty-phillips/inko-regex 0.0.1
inko pkg sync
```

(check the tags on this repo to determine the latest tag in case I forget to update the README)

## Usage

The easiest interface is through the regex module:

```
let re = regex.compile('(ab).(def)+')
let result = re.run('abcdefdef').unwrap
```

Here, `result` is an option that is `None` if no match was found, and `Some` if a match was found.

If it is `Some` it'll be an instance of [regex.matcher.Match](blob/main/src/regex/matcher.inko#L82)
The key fields on this object are:

- `group(index: Int)` to get the [regex.matcher.Group](blob/main/src/regex/matcher.inko#L5) by its index
  in the string. Group indices are counted from left to right in order of open parens. This method
  returns an `Option`, as it is possible to construct regular expressions where not all groups match a given
  string (e.g: `(ab)|(cd)`)
- `result.match` which is a `Group` instance for the whole match.

The `Group` class returned by both those fields has this interface:

- `group.from_index` the index in the source string where this group started
- `group.to_index` the index in the source string where this group ended
- `group.string` the substring contained within those two indices

That's more or less it

## Supported regex syntaxes

### Concatenation

Concatenation of two regexes is indicated simply with adjacency. So `ab` is the concatenation of
the regular expressions matching `a` and `b` respectively and matches the string `ab`

### Union

Union of regular expressions is indicated with the `|` operator. So `a|b` is the union
of the regular expressions matching `a` and `b` respectively, and match either of the strings `a` or `b`.

### Grouping

Grouping is indicated with parentheses `()`. Any grouped regular expression is treated as one
unit when applying other operators. So `(abc)` matches the string `abc` and places them
in a group, while `(abc)|(def)` matches either the string `abc` or the string `def`, and places
the results in one of two groups.

In a regular expression's response, the group can be accessed with the `.group(index)` method,
where the desired group is indicated with the index of the open parenthesis counting from left to right.

> **Note**
> Named groupings are not yet supported.

### Cardinality operators

The operators `*`, `+`, and `?` are used to indicate cardinality on the preceding expression
(which may be a character or group):

- `*` matches the expression zero or more times, for unlimited repetition
- `+` matches the expression one or more times, for unlimited non-zero repetition
- `?` matches the expression exactly zero or one time, for optionality

Examples:

- `(ab)*` matches ``, `ab`, `abab`...
- `(ab)+` matches `ab, `abab`...
- `(ab)?` matches ``and`ab` and nothing else

> **Note**
> The common `{m,n}` cardinality syntax is NOT currently supported.

### Character classes

Character classes are indicated with square brackets `[]` and match any _single_ character
from inside the class.

For example `[abc]` matches the strings `a` or `b` or `c`.

Negated character classes are also supported; prefix the string with a caret `^`.

So `[^abc]` matches any character except `a` or `b` or `c`.

> **Note**
> Ranges and named character classes are not currently supported. So `[a-z]`, `[0-9]`, or `[:alpha:]`
> probably don't do what you would expect.

### Escapes

You can escape certain meta characters with a backslash to match against that character
instead of whatever the meta character would have done.

However, because `\` is already used as an escape in Inko strings, and Inko does not currently
have a concept of "raw" strings, you'll have to insert two backslashes in Inko code to get
one in the regex. This gets pretty crazy when you want to match a single backslash. You need
four backslashes in total: one to escape a backslash to Inko, a second to represent the regex
escape, a third to escape another backslash to Inko, and a fourth to represent the backslash
being escaped in the regex.

These are the currently supported escapes:

- `\\*` Match a literal `*`
- `\\+` Match a literal `+`
- `\\?` Match a literal question mark
- `\.` Match a literal period
- `\\\\` Match a literal backslash

All other escaped characters are technically supported in that they match themselves.
So `\\m` matches `m`, but why would you do that?

> **Note**
> Standard escapes for spaces, digits, and classes are not currently supported, so `\s`, `\w`, `\d`
> etc don't do what you normally expect.

## Contributing

- Bugs and bugfixes: PRs and new issues are both welcome. Please include a minimal repro.
- PRs for new PCRE features are welcome.
- Feature Requests not accompanied by a PR: will be prioritized based on my time and inclination, with a focus on making the engine as PCRE-compatible as possible rather than random syntax extensions.

### Development

`inko test` does what it's supposed to.
