# jq
a Swiss-army knife for JSON manipulation



# Examples
First up, what sorts of things can you do with `jq`?


## Find all 2-syllable words that rhyme with "alabaster"

```
$ curl 'https://api.datamuse.com/words?rel_rhy=alabaster' \
  | jq '.[] | select(.numSyllables == 2) | .word'
"master"
"aster"
"plaster"
"pastor"
"castor"
# ...
```


## Count votes
```bash
$ curl 'http://data.cnn.com/ELECTION/2016/full/P.full.json' \
  | jq '
    [[.races[].candidates[]]
      | group_by(.party)[]
      | {(.[0].party): map(.votes) | add}]
    | add'
{
  "D": 131707032,
  "GR": 1420123,
  "I": 243690,
  "LB": 4282794,
  "R": 125969650
}
```


## What % of cPython commits are fixing typos?
```bash
$ curl 'https://api.github.com/repos/python/cpython/commits'\
  | jq -r '
    ([.[].commit.message | select(test("typo"; "i"))]
      | length)
    / length
    | "\(. * 100)% typos"'
4% typos
```



## Structure of a JQ program
JQ programs are _filters_ that map an input stream of JSON values to an output
stream of JSON values.

<pre style="text-align: center">
<code class="lang-json">[]; {}; null &#x27f6; 3; "hi"</code></pre>


Every filter takes a stream of JSON values as input, and produces a stream of
JSON values as output.


JQ is a terse syntax for defining composite filters.



## Composition

The most useful ways of composing filters are piping, and forking.


## Piping

🎺

The `|` operator combines two filters by feeding the output of the left-hand
side to the input of the right-hand side. Just like bash.


<pre style="text-align: center">
<code class="lang-json">.xs | add</code></pre>

<pre style="text-align: center">
<code class="lang-json">{"xs": [1,2]}; {"xs": [3,4]}
↓
.xs
↓
add
↓
3, 7</code></pre>

- feeds the input to `.xs`
- takes the output of `.xs` and feeds it to `add`


## Forking

🖖

Almost all other binary operators in JQ _fork_ the input.


<pre style="text-align: center">
<code class="lang-json">add / length</code></pre>

<pre style="text-align: center">
<code class="lang-json">[1, 2, 3, 4]
↙  ↘
add   length
↘  ↙
/
↓
2.5</code></pre>

- feeds the value `[1,2,3,4]` to `add`
- feeds the value `[1,2,3,4]` to `length`
- zips the outputs of each and produces the ratio.



## Basic filters

From these, all other filters are built.

<div style="margin: auto auto">
```text
                          .=========.
                         / (_)  (_) /| 
                        /-========-/ |
                        |          |/
                        '-========-'
```
</div>
<small><small>(image credit: Joan G. Stark)</small></small>


### The identity filter

🙆

<pre style="text-align: center">
<code>.</code></pre>
<pre style="text-align: center"><code class="lang-json">{}; {"hello": "world"}; 3
↓
{}; {"hello": "world"}; 3</code></pre>


### Constant filter

3️⃣

<pre style="text-align: center"><code>3</code></pre>
<pre style="text-align: center"><code class="lang-json">{}; {"hello": "world"}; 3
↓
3; 3; 3</code></pre>


### Key extraction

🔑

<pre style="text-align: center"><code>.foo</code></pre>
<pre style="text-align: center"><code class="lang-json">{"foo": "bar"}; {"foo": "baz"}; {}
↓
"bar"; "baz"; null</code></pre>


### Key extraction

🔑🔑

<pre style="text-align: center"><code>.foo.bar</code></pre>
<pre style="text-align: center"><code class="lang-json">{"foo": {"bar": "baz"}}
↓
"baz"</code></pre>


### Iteration
🔁

a.k.a. spread, flatten

<pre style="text-align: center"><code>.[]</code></pre>
<pre style="text-align: center"><code class="lang-json">["a", "b"]; ["c", "d"]
↓
"a"; "b"; "c"; "d"</code></pre>


### Object construction

🚧

<pre style="text-align: center"><code class="lang-json">{name: .first}</code></pre>
<pre style="text-align: center"><code class="lang-json">{"first": "J", "last": "A"}
↓
{"name": "J"}</code></pre>


### Array construction

🚧 🚧 🚧 🚧

<pre style="text-align: center"><code class="lang-json">[.[].foo]</code></pre>
<pre style="text-align: center"><code class="lang-json">[{"foo": 1}, {"foo": 2}]
↓
[1, 2]</code></pre>


### Concatenating streams

⏩

<pre style="text-align: center"><code class="lang-json">.foo, .bar</code></pre>
<pre style="text-align: center"><code class="lang-json">{"foo": "f", "bar": "b"}; {"foo": "o", "bar": "r"}
↓
"f"; "b"; "o"; "r"</code></pre>



## Functions


### select

<pre style="text-align: center"><code class="lang-json">select(. > 4)</code></pre>
<pre style="text-align: center"><code class="lang-json">2; 3; 4; 5; 6
↓
5; 6</code></pre>


### sort, unique, reverse
<pre style="text-align: center"><code class="lang-json">unique_by(.foo) | reverse</code></pre>
<pre style="text-align: center"><code class="lang-json">[{"foo": 3}, {"foo": 2}, {"foo": 3}]
↓
[{"foo": 3}, {"foo": 2}]</code></pre>


### string processing

split, join, explode, \\(string interpolation), @csv, regex
<pre style="text-align: center"><code class="lang-json">split(",") | .[] | select(test(".x")) | explode
  | @csv | "CSV row: \(.)"</code></pre>
<pre style="text-align: center"><code class="lang-json">"jax,hax,jq"
↓
["jax", "hax", "jq"]
↓
"jax"; "hax"; "jq"
↓
"jax"; "hax"
↓
[106, 97, 120]; [104, 97, 120]
↓
"106,97,120"; "104,97,120"
↓
"CSV row: 106,97,120"; "CSV row: 104,97,120"
</code></pre>



## Logic

It does all that Boolean stuff

<pre style="text-align: center"><code class="lang-json">==, !=, <=, >=
&nbsp;
if A then B else C end</code></pre>



## invocation

--raw-output (-r), --compact-output (-c)

--slurp (-s), --null-input (-n)

--arg, --argjson



## jq is a real language!

It has:
- variables
- functions
- recursion
- tail-call optimization
- closures
- lazy evaluation
- modules
- a compiler, linker & bytecode



## Useful links

https://stedolan.github.io/jq/manual/

https://github.com/stedolan/jq/wiki/Advanced-Topics



## Ephemera


### The try filter

`expr?` (or `try expr`) acts like `expr`, but errors are suppressed (i.e.
transformed to the empty stream)

<pre style="text-align: center"><code class="lang-json">.foo?</code></pre>
<pre style="text-align: center"><code class="lang-json">{"foo": 3}; []; "hi"
↓
3
</code></pre>


### Non-numeric arithmetic

In jq, additive arithmetic operators `+`, `-` and `*` work on most types, not
just numbers.


`null` is the additive identity for every type.

```json
["hi"] + null ⇒ ["hi"]
```


Adding two arrays concatenates them. Subtracting an array from an array
produces an array without the elements in the subtracted array.

```json
[1,2] + [3,4] ⇒ [1,2,3,4]
[1,2] - [2]   ⇒ [1]
```


Adding two objects merges them. Multiplying two objects recursively merges
them.

```json
{"a": 1} + {"b": 2} ⇒ {"a": 1, "b": 2}

{"a": {"b": 1}} * {"a": {"c": 1}}
  ⇒ {"a": {"b": 1, "c": 1}}
```


### Indexing by a stream

<small>The program `.[4,2]` produces the 5th and 3rd elements of the input array. But
`4,2` is a stream! (Or to be specific, a constant filter that produces `4`
followed by `2` regardless of what its input is).</small>

<small>We can put whatever filter we want in there:</small>

<pre style="text-align: center"><code class="lang-json">.[.[1]&lbrack;]]</code></pre>
<pre style="text-align: center"><code class="lang-json">[1, [2, 3], 4, 5]
↓
4; 5</code></pre>
