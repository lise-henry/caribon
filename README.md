Caribon
=======

A repetition detector written in Rust.

Why?
====

I don't think it's really the case in english, but in french (and
possibly other languages), it is considered poor style to repeat a
word too often in a text (particularly a literary text). The purpose
of this tool is to assist a writer in detecting those repetitions.

Why the name?
=============

A text is composed of words, themselves composed of characters, which
in french are called *caractères*. In french, *good* is *bon* so
*caribon* is essentially *good characters*.

Alright, this doesn't make much sense, I'll admit I just found the
name funny.

How?
====

Internally, Caribon use a stemming library
(https://github.com/lady-segfault/stemmer-rs, the Rust bindings for
Snowball C implementation: http://snowball.tartarus.org/) to reduce
words to their stems, which allows e.g. to see a singular and a plural
as the "same" word. Then it's just counting the repetitions, and
outputting HTML.

Build
=====

You'll need Rust and Cargo, see [install instructions](http://www.rust-lang.org/install.html). Then

`$ cargo build`

should do the job (it works with Rust 1.1). You can then run caribon either with:

`$ cargo run`

or by directly executing the binary (in `target/debug` or
`target/release`). Since, at this moment either write on standard
output or takes arguments, the
latter method is preferred:

`$ target/debug/caribon < some_text.txt > output.html`

or

`$ targe/debug/caribon --input=some_text.txt --output=output.html`

If you want to use `cargo run`, you can obtain the same results by giving `caribon` some
command-line arguments, which must be prefixed by `--` so cargo gives
it to the binary: 

`$ cargo run -- --input=some_text.txt --output=output.html`

You can also install the `caribon` binary somewhere in your path
(e.g. `/usr/local/bin`) but currently there is no install/uninstall
option, so you'll have to do it manually.

Once you have generated an HTML file, just open it with your favorite
browser and see your repetitions. Note that the default binary is
configured for french, if you want to use another language, you'll
have to pass an option (see below). Note that though a variety of input
languages are supported thanks to the Snowball stemming library, at
this time only french has a (incomplete) list of common words to
ignore. 

Usage
=====

```
Caribon, version 0.3.0 by Élisabeth Henry <liz.henry@ouvaton.org>

Reads a file on stdint and outputs an HTML file showing the repetitions

Options:
--help: displays this message
--version: displays program version
--list_languages: lists the implemented languages
--language=[language]: sets the language of the text (default: french)
--input=[filename]: sets input file (default: stdin)
--output=[filename]: sets output file (default: stdout)
--algo=[global|local|leak]: sets the detection algoritm (default: local)
--leak=[value]: sets leak value (only used by leak algorithm) (default: 0.95)
--max_distance=[value]: sets max distance (only used by local algorithm) (default: 50)
--global_count=[relative|absolute]: sets repetitions count as absolute or relative ratio of words
                       (only used by global algorithm) (default: absolute)
--threshold=[value]: sets threshold value for underlining repetitions (default: 1.9)
--html=[true|false]: enables/disable HTML input (default: true)
--ignore_proper=[true|false]: if true, try to detect proper nouns and don't count them (default: false)
```

Example
=======

Here is an [example](https://lady-segfault.github.io/caribon-examples/example_readme.html) of Caribon used on a (previous) version of this
README, using the following command:

`target/debug/caribon --language=english --algo=local
--max_distance=25 --threshold=2.0 --ignore_proper=true < README.html  > example.html`

Library
=======
It is possible to use Caribon as a library. The documentation is
available [here](http://lady-segfault.github.io/caribon/); in order to
get the latest version, you can also generate it with
`cargo doc`.

Basically, it's pretty easy:

You create a new parser with `Parser::new("language")` (the only
trick is that it returns an `Option`, as all languages are not
implemented, see `Parser::list_languages()` to get a vector of those
that are implemented by the stemming library.

You can then set some parameters for the parser, e.g:

```rust
let parser = Parser::new("french")
    .unwrap()
    .with_html(true) // enable html in input (default value, so it's useless)
    .with_ignore_proper(true); // don't count repetitions for proper nouns 
```

The first step is to "tokenize" the string you want to parse:

```rust
let words = parser.tokenize("Some string which may or may not contain repetitions");
```

The second step is to detect the repetitions, using one of the three
algorithms:

```rust
let detected_words = parser.detect_local(words); 
let detected_words = parser.detect_global(words, false); 
let detected_words = parser.detect_leak(words);
```

The final step is to display this vector of words. `Caribon` provides
a function that generates an HTML file, which also takes as argument a
threshold above which words are underlined:

```rust
println!("{}", caribon::words_to_html(&detected_words, 1.5));
```

(A note on this threshold: its choices depends on the detection
algorithm you use (and possibly your taste and the language you write
in, of course). Generally, it should be a bit above 1.0, except for
`detect_global` (in which case, it depends whether you set
`is_relative` to true or false).



Current features
================

* Built-in list of ignored words (common words whose repetitions don't
  matter) for french and english, though they are not complete.
* Basic support for other
  languages supported by the Snowball (http://snowball.tartarus.org/)
  project.
* Count repetitions either locally (either by ignoring repetitions
  after a given distance, or using some leak-based algorithm) or globally.
* Detects HTML tags in input. It doesn't work with a full HTML file
  (containing `<html>`, `<body>` and so on) but it works fine if you
  use e.g `pandoc -o file.html file.md`.
* Outputs a basic HTML files which higlights the detected repetitions.

ChangeLog
=========

[See here](ChangeLog.md).

License
=======

Caribon is licensed under the [GNU General Public License](LICENSE), version 2.0
or (at your convenience) any ulterior version.

Author
======

Élisabeth Henry <liz.henry at ouvaton.org>.

This software uses (rust bindings to) the
[C Stemming library](http://snowball.tartarus.org/dist/libstemmer_c.tgz)
written by Dr Martin Porter, licensed under the BSD License. 

ToDo 
====

Library
-------
* Render prettier output files;
* Allow tokenizer to have in input "full" (with <html>,<head>,<body>
  tags) HTML documents;
* Correctly escapes &nbsp; and such in HTML input;
* Enhance the way language-dependent list of ignored words are
  treated, and provide them for other languages (currently, only
  french, and it should be completed);
* Provide algorithm to detect repetitions of expressions, not just
  single words;
* Find better default values;
* Enhance documentation and add tests.

See also 
---------

[caribon-server](https://github.com/lady-segfault/caribon-server), a
work-in-progress project that runs Caribon as a web service. Hopefully
we can soon have a demo server on the Internet!
