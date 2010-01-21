PHP.rb: A Ruby to PHP Code Generator
====================================

> "Will write code that writes code that writes code that writes code for money."
>
> -- Anonymous on [comp.lang.lisp](http://lispers.org/#money)

`PHP.rb` translates [Ruby][] code into [PHP][] code by obtaining the [parse
tree][ParseTree] for a Ruby expression, transforming that into an [abstract
syntax tree][AST] (AST) compatible with PHP, and then generating valid PHP
code as the final output.

* <http://github.com/bendiken/php>

Usage
-----

    require 'php'

### Generating PHP code using a Ruby block

`PHP.generate` returns the translated AST for a given Ruby block. The AST
can be turned into runnable PHP code by calling the `#to_s` method on the
resulting `PHP::Program` instance.

    PHP.generate { echo "Hello, world!\n" }   #=> PHP::Program(...)

### Evaluating generated PHP on the fly

`PHP.eval` translates Ruby code into PHP utilizing `PHP.generate` and then
executes it using the `php` executable (which must be available in
`ENV['PATH']`).

    PHP.eval { echo "Hello, world!\n" }

### Checking the available PHP version

`PHP.version` utilizes `PHP.eval` to obtain the currently available PHP
version (if any).

    PHP.version                               #=> "5.3.1"

Examples
--------

The following snippets demonstrate the PHP output generated by `PHP.rb` for
the given Ruby code. Look in the [`doc/examples/`][examples] directory for
runnable Ruby scripts containing these examples.

### Function definitions

    def unid
      return md5(uniqid(mt_rand, true))
    end

    <?php
    function unid() {
      return md5(uniqid(mt_rand(), TRUE));
    }

### `foreach` loops (1)

    for $fruit in ['apple', 'banana', 'cranberry']
      echo "#{$fruit}\n"
    end

    <?php
    foreach (array("apple", "banana", "cranberry") as $fruit) {
      echo($fruit . "\n");
    }

### `foreach` loops (2)

    for $key, $value in {'a' => 1, 'b' => 2, 'c' => 3}
      echo "#{$key} => #{$value}\n"
    end

    <?php
    foreach (array("a" => 1, "b" => 2, "c" => 3) as $key => $value) {
      echo($key . " => " . $value . "\n");
    }

### `while` loops

    $result = mysql_query("SELECT name FROM user")
    while $row = mysql_fetch_assoc($result)
      echo "User.name = #{$row['name']}\n"
    end

    <?php
    $result = mysql_query("SELECT name FROM user");
    while ($row = mysql_fetch_assoc($result)) {
      echo("User.name = " . $row["name"] . "\n");
    }

Reference
---------

The following cheat sheet demonstrates the PHP output generated for various
types of Ruby expressions. The rows marked with an asterisk, `(*)`, may have
special semantics that need to be taken into account.

### Literals

    Ruby input                           | PHP output
    -------------------------------------|--------------------------------------
    nil                                  | NULL
    false                                | FALSE
    true                                 | TRUE
    42                                   | 42
    3.1415                               | 3.1415
    "Hello, world!"                      | "Hello, world!"
    "<#{$url}>"                          | "<" . $url . ">"
    /a-z/                                | '/a-z/'
    []                                   | array()
    [1, 2, 3]                            | array(1, 2, 3)
    {}                                   | array()
    {"a" => 1, "b" => 2, "c" => 3}       | array("a" => 1, "b" => 2, "c" => 3)
    (1..10)                              | range(1, 10)
    (1...10)                             | range(1, 9)

### Variables

    Ruby input                           | PHP output
    -------------------------------------|--------------------------------------
    $foo                                 | $foo
    $foo = 123                           | $foo = 123

### Functions

    Ruby input                           | PHP output
    -------------------------------------|--------------------------------------
    lambda { |$x, $y| }                  | function($x, $y) {}
    def foo(x, y); end                   | function foo($x, $y) {}
    time                                 | time()
    time()                               | time()

### Methods

    Ruby input                           | PHP output
    -------------------------------------|--------------------------------------
    $array[$index]                       | $array[$index]
    $object[:property]                   | $object->property                 (*)
    $object.method                       | $object->method()

### String operators

    Ruby input                           | PHP output
    -------------------------------------|--------------------------------------
    $a << $b                             | $a . $b                           (*)

### Arithmetic operators

    Ruby input                           | PHP output
    -------------------------------------|--------------------------------------
    -$a                                  | -$a
    $a + $b                              | $a + $b
    $a - $b                              | $a - $b
    $a * $b                              | $a * $b
    $a / $b                              | $a / $b
    $a % $b                              | $a % $b

### Assignment operators

    Ruby input                           | PHP output
    -------------------------------------|--------------------------------------
    $a = $b                              | $a = $b

### Bitwise operators

    Ruby input                           | PHP output
    -------------------------------------|--------------------------------------
    ~$a                                  | ~$a
    $a & $b                              | $a & $b
    $a | $b                              | $a | $b
    $a ^ $b                              | $a ^ $b

### Comparison operators

    Ruby input                           | PHP output
    -------------------------------------|--------------------------------------
    $a == $b                             | $a == $b
    $a === $b                            | $a === $b                         (*)
    $a != $b                             | $a != $b
    $a < $b                              | $a < $b
    $a > $b                              | $a > $b
    $a <= $b                             | $a <= $b
    $a >= $b                             | $a >= $b

### Execution operators

    Ruby input                           | PHP output
    -------------------------------------|--------------------------------------
    `hostname`                           | `hostname`

### Logical operators

    Ruby input                           | PHP output
    -------------------------------------|--------------------------------------
    !$a                                  | !$a
    $a and $b                            | $a && $b
    $a && $b                             | $a && $b
    $a or $b                             | $a || $b
    $a || $b                             | $a || $b

### Control structures

    Ruby input                           | PHP output
    -------------------------------------|--------------------------------------
    if true then ... else ... end        | if (TRUE) { ... } else { ... }
    ... if true                          | if (TRUE) { ... }
    unless true then ... else ... end    | if (!TRUE) { ... } else { ... }
    ... unless true                      | if (!TRUE) { ... }
    while $x; ...; end                   | while ($x) { ... }
    until $x; ...; end                   | while (!$x) { ... }
    for $x in $y; ...; end               | foreach ($y as $x) { ... }
    for $k, $v in $z; ...; end           | foreach ($z as $k => $v) { ... }
    return                               | return
    return $x                            | return $x

### Classes and objects

    Ruby input                           | PHP output
    -------------------------------------|--------------------------------------
    MyClass.new                          | new MyClass
    MyClass.new($argument)               | new MyClass($argument)

Limitations
-----------

### This is *not* a Ruby runtime

`PHP.rb` is not Ruby for PHP, merely PHP for Ruby.

You don't have the Ruby standard library available in your generated code. 
To generate runnable PHP code, you can't say e.g.  `"foobar".size` and must
rather say `strlen("foobar")`.

### Method calls vs property access

Ruby method calls, e.g. `$user.name`, are in principle ambiguous when
translated into PHP, because they could resolve into either a property
access as in `$user->name` or a method call as in `$user->name()`.

Therefore `PHP.rb` defines `$user.name` to be equivalent to the latter (the
method call), and defines the syntax `$user[:name]` to be equivalent to the
former (the property access).

Note that this does not conflict with array subscript access since Ruby
symbol objects have no semantic equivalent in PHP.

Documentation
-------------

* <http://php.rubyforge.org/>

Methods
-------

* {PHP.eval}
* {PHP.exec}
* {PHP.dump}
* {PHP.generate}
* {PHP.version}

Classes
-------

* {PHP::Generator}
* {PHP::Program}
* {PHP::Expression}

Dependencies
------------

* [Open4](http://gemcutter.org/gems/open4) (>= 1.0.1)
* [ParseTree](http://gemcutter.org/gems/) (>= 3.0.4)
* [RubyParser](http://gemcutter.org/gems/) (>= 2.0.4)
* [SexpProcessor](http://gemcutter.org/gems/sexp_processor) (>= 3.0.3)

Installation
------------

The recommended installation method is via RubyGems. To install the latest
official release from Gemcutter, do:

    % [sudo] gem install php

Download
--------

To get a local working copy of the development repository, do:

    % git clone git://github.com/bendiken/php.git

Alternatively, you can download the latest development version as a tarball
as follows:

    % wget http://github.com/bendiken/php/tarball/master

Resources
---------

* <http://php.rubyforge.org/>
* <http://github.com/bendiken/php>
* <http://gemcutter.org/gems/php>
* <http://rubyforge.org/projects/php/>
* <http://raa.ruby-lang.org/project/php/>

Authors
-------

* [Arto Bendiken](mailto:arto.bendiken@gmail.com) - <http://ar.to/>

License
-------

`PHP.rb` is free and unencumbered public domain software. For more
information, see <http://unlicense.org/> or the accompanying UNLICENSE file.

[PHP]:       http://www.php.net/
[Ruby]:      http://www.ruby-lang.org/
[ParseTree]: http://www.zenspider.com/ZSS/Products/ParseTree/
[AST]:       http://en.wikipedia.org/wiki/Abstract_syntax_tree
[examples]:  http://github.com/bendiken/php/tree/master/doc/examples/
