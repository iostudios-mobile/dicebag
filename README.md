# dicebag

A dice expression parser and roller.

## Installation

    # in a local node_modules/
    npm install --save dicebag
    # globally, to use the CLI
    npm install -g dicebag

## Command-line usage

    dicebag [-p] [<dice expression>]

If a dice expression is provided, prints the result of rolling those dice and
exits. Otherwise, reads expressions from `stdin` in a loop.

* `-p` print dice pools (default behavior is to print the dice's sum)

### Examples

    $ dicebag 1d6
    1
    $ dicebag "2d8 + 1d4"
    7
    $ dicebag -p "2d8 + 1d4"
    [ 5, 3, 4 ]

## Library usage

    const { parse, pool, roll } = require('dicebag')

The API consists of three functions:

* `parse(diceExpression)` parses an expression into an object understood by the
  other two functions.
* `pool(dice)` rolls the dice and returns an array of their results.
* `roll(dice)` rolls the dice and returns their sum.

### Examples

    const d6 = parse('1d6')
    roll(d6)   // 4
    roll(d6)   // 5
    pool(d6)   // [ 2 ]
    const dice = parse('2d6 + 1d8')
    roll(dice) // 10
    pool(dice) // [ 1, 4, 7 ]

## Dice expressions

### Basics

Simple expressions involving standard dice notation as used in most roleplaying
games are supported. You can do things like:

* `XdY`: rolls `X` `Y`-sided dice (`1d6` is a single 6-sided die, `2d4` is two
  4-sided dice).
* `dX` is the same as `1dX` (so you can shorten `1d6` to `d6`).
* `dice +/- constant`: rolls the dice, adds/subtracts the constant.
* `dice +/- moreDice`: sums/takes the difference of the results of rolling
  `dice` and `moreDice`.

### Full syntax and semantics

**Note:** this is still an early version. Syntax and semantics will be expanded
in future versions. *Backwards incompatible changes are possible.*

The parser recognizes the following grammar (all whitespace between symbols is
ignored):

    Die ::= <an integer>
          | '(' Die ')'
          | Die 'd' Die
          | 'd' Die
          | Die '+' Die
          | Die '-' Die
          | '-' Die
          | Die 'E' Die

Semantics are defined in terms of the `pool` function.

* `N`, where `N` is an integer, is a die that always rolls a single value
  equal to `N`. `pool` returns an array containing just `N`.
* `DdE`, where `D` and `E` are dice expressions, is a die that rolls a number of
  dice equal to the result of rolling `D`, where each die has a number of sides
  equal to the result of rolling `E`. `pool` returns an array of `roll(D)`
  numbers, each between 1 and `roll(E)`. *Note:* if `D` or `E` evaluates to a
  negative number, the behavior is undefined.
* `dD` is equivalent to `1dD`.
* `D+E` appends the dice pool generated by `E` to the dice pool generated by
  `D`.
* `-D` returns the opposites of values generated by `D`.
* `D-E` is equivalent to `D+(-E)`.
* `DEF` (here `E` is the literal symbol `E`, `D` and `F` are dice expressions)
  is an "exploding die." First `D` is rolled.  Now each die in the dice pool
  generated by `F` is rolled repeatedly until it rolls something less than the
  value rolled on `D`. The die's result is the sum of all those rolls. *Note:*
  this could lead to an infinite evaluation if `F` always rolls higher than a
  possible result of `D`.

Additionally:

* The binary arithmetic operations (`+`, `-`) are left associative.
* The die operation `d` is right associative (`1d2d3` is equivalent to
  `1d(2d3)`, use explicit parentheses if you need `(1d2)d3`)
* `d` binds stronger than the binary arithmetic operations (`1d6+1d4` is
  equivalent to `(1d6) + (1d4)`).
