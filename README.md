Caribon
=======

A repetition detector written in Rust.

[![Build Status](https://travis-ci.org/lise-henry/caribon.svg?branch=master)](https://travis-ci.org/lise-henry/caribon)

Demo server
===========

If you want to have an idea of what Caribon does without downloading
anything, you can have a look at
[this instance that runs Caribon as a web service](http://vps184889.ovh.net/caribon/).

Downloading
===========

Either use `git` to get the latest version:

`$ git clone https://github.com/lise-henry/caribon.git`

or just download one of the stable(ish)
[releases](https://github.com/lise-henry/caribon/releases).

(If you only plan to use Caribon as a library for your rust program,
you don't need to worry too much about downloading and building, just
add `caribon = "*"` in your `Cargo.toml` file.)


Build
=====

You'll need Rust and Cargo, see their [install instructions](http://www.rust-lang.org/install.html). Then

`$ cargo build`

should do the job, though you can also run `cargo build --release`
directly. You can then run caribon either with:

`$ cargo run --release`

or by directly executing the binary (in `target/debug` or
`target/release`):

`$ target/release/caribon`

Installing
==========

If you have a recent version of `Cargo`, you can use:

`$ cargo install caribon`

(for the latest version on [crates.io](http://crates.io), or

`$ cargo install --git https://github.com/lise-henry/caribon`

for the latest version on GitHub)

which will download, build and install Caribon.

Cargo run
=========

If you don't want to install Caribon, `cargo run` might be the
simplest option. Note, though, that command-line arguments must
be prefixed by `--` so cargo gives them to the binary: 

`$ cargo run -- --input=some_text.txt`

Also note that, by default, `cargo run` builds and runs the program in
debug mode, which is slower. This isn't a problem for tiny files, but
if you plan to detect a repetitions in, say, a novel, using
cpu-extensive features (such as fuzzy string matching, see below), you might want
to run with `--release`:

`$ cargo run --release -- --input=big_file.html --output=output_big_file.html`

Examples
========

Here is an
[example](https://lise-henry.github.io/rust/caribon-examples/example_readme.html)
of Caribon using the HTML output of a (previous) version of this 
README, obtained with the following command:

`$ caribon --language=english --input=README.html
--output=example.html --fuzzy=0.5`

(Note that `--fuzzy=0.5`, while useful to show that fuzzy string
matching does indeed work, is not a very sensible parameter as is it
quite high (words only needs to be 50% similar to be considered the
same, matching e.g. `just` and `rust`). For real life usage, a lower value
would be recommended.)

Here is another [example](https://lise-henry.github.io/rust/caribon-examples/screenshot.png), displaying repetitions in
`README.md` to the terminal, using the following command:

`$ caribon --language=english --input=README.md --fuzzy=0.5 | more`

![example](https://lise-henry.github.io/rust/caribon-examples/screenshot.png)

While outputting to the terminal might be useful for small files, HTML
outputs gives a more useful result, as higlighting a word will show
you the other occurrences of it.

Options
=======

Caribon provides a list of options. Here's the
explanations to a few ones, from the most commons the the pretty
advanced ones:

### Language ###

* `--language=[english|french|spanish|...]`specifies the language of the
  input file. It is important for two reasons. The first one is that
  Caribon internally uses a stemming library, which will detect when
  words are derived from the same stem, e.g. "eats", "eat" and
  "eating" will be considered the same word. (More information on how
  this stemming library works can be found on the
  [Snowball project website](http://snowball.tartarus.org/).) The
  second reason is that for some languages (currently only french and
  english), Caribon provides a default list of words to ignore for
  repetition counting (e.g. in english "it", "a" and so on are on it)
  to avoid cluttering the result file. It is possible to disable
  stemming by using "no_stemmer" instead of a language. This isn't
  really advised, but it might be useful if you want to try Caribon on
  a language that isn't implemented.
* `--list-languages` prints the list of languages supported by the
  stemming library.

### Input and output ###

* `--input=[file]` specifies the input file. By default it is `stdin`,
which means you'll have directly to type your text and end it with
`control-D`. If `file` is a non-existing file, the program aborts.
* `--output=[file]` specifies the output file. It defaults to `stdout`,
printing the result to the terminal.

The input and output filenames extension determine the input and
output format, e.g. if you pass `--input=text.html --output=result.html`, Caribon will
infer that the content is in HTML and that it must also output HTML
(so `$ caribon < input.html > output.html` is NOT equivalent to `$
caribon --input=input.html --output=output.html`: in the first case,
Caribon will consider the input as raw text and will output in
`terminal` format (see below), while in the latter one it will
understand that both files are HTML).

It is possible to override this behaviour by specifying

* `--input-format=[text|html]` or
* `--output-format=[terminal|html|markdown]`.

A note on the `terminal` output format: it is designed to print text
to the terminal, by underlining and colouring some words with UNIX
terminal special characters (see screenshot above). It is, thus, only activated when no
output file name is given and Caribon prints on the standard output,
HTML output being the default for most of the cases.

### Text statistics ###

* `--print-stats`, if passed to Caribon, will also display some statistics
  on the input text on the standard output.

### Threshold and max-distance ###

The most useful algorithm of Caribon is local repetition
detections. It detects when a word is repeated in a given interval of
words. This interval is determined by

* `--max-distance=[value]` (default is currently 50).

So basically, if `max-distance` is 50 and the word 'foo' occurs twice in this
interval, each occurrence will have a "repetition value" of 2. If
'foo' is repeated a third time in a 50-words interval *after the second
occurence*, then each of these occurrences will have a repetition
value of 3. (If there is then more than 50 words without apparition of
'foo', and 'foo' appears again, the value of the latest apparition
will be reset to 1).

Words are underlined when their "repetition value" is higher than a
threshold, which can be set by:

* `--threshold=[value]`. The default is `1.9`, so a word will be underlined
  as soon that is is repeated two times locally. If you change the threshold to, say,
  `2.5`, a word will have to be repeated three times (locally) to be
  underlined.

(Why a float value for the threshold, instead of an integer one?
Because the local repetition detector will underline words in
different colors: green, orange and red according to the "severity" of
the repetitions. So setting the threshold to `1.01` or `1.99` will not
change which words are underlined, but they will be in orange or red
more quickly in the first case.)

### Fuzzy string matching ###

Caribon uses a stemming library to detect words that are part of the
same 'family'. It turns out that this algorithm is not always
enough, and particularly it doesn't detect repetitions when there is a
typo (e.g. "higlight" and "highlight" should probably be considered a
repetition, even if it is mispelled in the first case). To solve this
issue, there is the option of activating fuzzy string matching:

* `fuzzy=[value]`, where the value is a number between 0.0 and 1.0 which
  represents the maximal 'difference' between two words until they are
  no more identical: a value of 0.2 means that two words must be at
  most "20% different" until they are no more considered the same.

Internally, this algorithm uses the
[Levenshtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance)
(and more specifically the
[Rust implementation by Florian Ebelling](https://crates.io/crates/edit-distance))
which computes a distance between two strings by estimating the number
of insertions, deletions and permutations it require to go from one to
another. E.g., "dog" and "dogs" have a distance of 1, while "dog" and
"cat" have a distance of 3. This value is then divided by the length
of the string to match, and two string are considered "identical" (or,
at least, a repetition) when this value is less than the value given
to `--fuzzy=`.

E.g., with `--fuzzy=0.2`, "highlight" and "higlight" will have a
"difference" estimated to 1/9 (Levenshtein distance of 1, it only needs
one deletion to go from the first to the second, divided by the length
of "highlight", 9), so it will be a repetition. "Just" and "Rust" will
have a "difference" of 1/4, so won't be considered a repetition.

Fuzzy matching is practical, but you should not set it to a too high
value, else you will have a lot of false positives. Empirically, `0.2`
or `0.25` is a good choice.

Fuzzy matching has a drawback: it requires a lot more of CPU. Caribon
still manages to run reasonably fast (e.g., less than a second to
detect repetitions on a whole novel, with fuzzy string matching
activated) but it only uses fuzzy string matching for local
repetitions, and not for global ones (see below).

### Global repetitions ###

By default, Caribon only detects repetitions at a local level (if they
are separated by less than `max-distance` words). It is,
however, possible to activate global repetition detecting with:

* `--global-threshold=[value]`, value being (again) a number between 0.0
and 1.0.

In this case, a word will be considered a repetition (even if it is
never repeated in a `max-distance` range of words) if the relative
number of occurence is higher than the global threshold. I.e., if
`global-threshold` is set to 0.01, a word will be highlighted (in
blue) if it represents more than 1% of the total number of words in
the document.

### Ignored words ###

Some words, like "a" or "the", are unavoidably repeated a
lot and it doesn't make much sense to consider them a repetition. It
is thus useful to ignore some words. `Caribon` provides a 
default list for english and french, but it is in all cases possible
to provide your own with:

* `--ignore="list of common words"`.

This list must be separated by either spaces or commas (or, actually,
anything that isn't a letter), and must be encircled by
quotes. This list *replaces* the default one
provided by Caribon (for english and french, at least). If you want to
*add* words to these list instead of replace it, use:

* `--add-ignored="list of more ignored words"`

Another option for ignoring words is:

* `--ignore-proper=[true|false]` (default is to false)

If sets to true, Caribon will try to ignore proper nouns". That is, a word will not
count for repetition if it starts with a capital letter and
is not at the beginning of a sentence.

Library
=======

It is possible to use Caribon as a Rust library. The documentation is
available [here](http://lise-henry.github.io/rust/caribon/index.html); in order to
be certain to have the documentation version corresponding to the code
you downloaded, you can also generate it with
`cargo doc`.

Caribon library is also available on
[Crates.io](https://crates.io/crates/caribon), allowing you to easily
use it in any Cargo project: just add

`caribon = "0.7"`

in the dependencies section of your
`Cargo.toml` file.

Caribon-server
==============

If you are not a big fan of command-line interface, you can have a
look at [Caribon-server](https://github.com/lise-henry/caribon-server) that
runs Caribon as a web service (using
[Iron framework](https://github.com/iron/iron)). See [here for an
instance running it](http://vps184889.ovh.net/caribon/).

Current features
================

* Built-in list of ignored words (common words whose repetitions don't
  matter) for french and english, though they are not complete.
* Stemming support for languages supported by the Snowball (http://snowball.tartarus.org/)
  project.
* Additionally (because stemming algorithms aren't always perfect, and sometimes
  you make typos), support for fuzzy string matching (based on Levenhstein distance).
* Count repetitions locally and globally.
* Detects HTML tags in input. Normally works both for HTML fragments
  or full HTML pages.
* Outputs the detected repetitions either in an HTML file (the most
  useful option), directly to the terminal, or to a Markdown file (with less useful information).

ChangeLog
=========

[See here](ChangeLog.md).

License
=======

Caribon is licensed under the
[GNU Lesser General Public License](LICENSE), version 2.1
or (at your convenience) any ulterior version.

Credits
=======

Caribon is written by Élisabeth Henry `<liz.henry at ouvaton.org>`.

This software uses (rust bindings to) the
[C Stemming library](http://snowball.tartarus.org/dist/libstemmer_c.tgz)
written by Dr Martin Porter, licensed under the BSD License.

It also uses the [Rust implementation](https://crates.io/crates/edit-distance) of
Levenshtein distance written by Florian Ebelling, licensed under the Apache 2.0 License.

ToDo 
====

Library
-------
* Make colour highlighting more configurable
* Complete builtin lists of ignored words and provide them for other
  languages (currently, only french, and english);
* Provide algorithm to detect repetitions of expressions, not just
  single words;
* Make library callable from C (and other languages than Rust);
* Enhance documentation and add tests.

Program
-------
* Add options to select highligting colours
* Find better default values?
* Make different repositories for program and library?
* Add a variant with GUI (Gtk+?)?

