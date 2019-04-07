---
layout: page
title: Implementation details
---

This page gives some details about the implementation of mathfly, as well as keeping track of alterations as they happen. It will hopefully be of use if anybody needs to maintain or extend mathfly.

There are two basic building blocks of mathfly. Natlink, a C++ tool originally written by an employee of Dragon Systems called Joel Gould in 1999, allows grammars written in Python to be imported directly into Dragon. As far as I know Nuance does not endorse this but also hasn't made any effort to stop it, presumably because it's users still need to buy Dragon, which is all they really care about. Secondly, dragonfly is a Python library which provides easy abstractions for building grammars to be imported using Natlink.

## Dragonfly
Once you have set a user directory in the Natlink configuration, all Python files in that directory whose names begin with an underscore will be imported whenever Dragon is started.

A very basic rule is constructed like this:

Firstly, we import the elements from dragonfly which we will need.
```
from dragonfly import Key, Text, MappingRule, Grammar, AppContext
```

Secondly, we create a rule which contains our desired commands in a dictionary. The first command will type some text, the second executes a keystroke, and the third combines both.
```
class ExampleRule(MappingRule):
    mapping = {
        "Command one": Text("example command one"),
        "Command two": Key("enter"),
        "Command three": Text("example command one") + Key("enter"),
    }
```

Finally, we create a grammar and load the rule.
```
grammar = Grammar("example")
grammar.add_rule(ExampleRule)
grammar.load()
```

It is also possible to make a grammar context specific, so that its commands will only be recognised when a particular program is the active window, by specifying an app context.
```
grammar = Grammar("example", context=AppContext("Notepad"))
```

## Controlling applications
These basic Key and Text objects are the way that the vast majority of Mathfly functionality is implemented. For example, to create an integral symbol in LyX, you would type a backslash, int, and then a space. This is done with `Text("\\int ")`.

## Caster
Rules created this way do not have CCR (continuous command recognition) however, meaning that you would have to pause between each command. For many commands this is not a problem, but for dictating mathematics we want to be able to dictate continuously without pauses.

To this end, Mathfly uses a couple of elements from the Caster voice programming project. The most important of these are the MergeRule and the CCRMerger. The caster documentation explains these in detail but the basic idea is that instead of simply adding a rule to a grammar and loading it, rules are registered with a central object which then manages them. This enables rules to be enabled and disabled with a voice command, and also allows for multiple rules to be merged together into one rule and then converted to enable continuous recognition.

## Configuration files
Another limitation of basic dragonfly rules is that they cannot be reloaded on-the-fly, requiring a full restart of Dragon for changes to be implemented, and that you need to edit Python source files to modify them.

Mathfly gets around this by defining the vast majority of commands in `.toml` configuration files. I had never used Vocola before creating mathfly, but it was later pointed out that this is very similar to their system. Another example of convergent evolution I guess.

The configuration file (`lyx.toml`) looks like this:
```
...
[tex_symbols1]
"square root"                  = "sqrt"
"[generic] root"               = "root"
"integral"                     = "int"
"double integral"              = "iint"
"triple integral"              = "iiint"
"times"                        = "times"
"divide"                       = "div"
"stop"                         = "cdot"
"plus or minus"                = "pm"
"partial"                      = "partial"
"infinity"                     = "infty"
"(nice frack | nice fraction)" = "nicefrac"
"binomial"                     = "binom"
...
```

With the spoken form on the left and the command to be executed on the right. This is then imported and implemented in `lyx_mathematics.py` with:
```
BINDINGS = utilities.load_toml_relative("config/lyx.toml")

class lyx_mathematics(MergeRule):
    mapping = {
        BINDINGS["symbol1_prefix"] + " <symbol1>":
                    Text("\\%(symbol1)s "),
    }
    extras = [
        Choice("symbol1", BINDINGS["tex_symbols1"]),
    ]

control.nexus().merger.add_app_rule(lyx_mathematics())
```

This looks slightly more complex, but has the same basic structure. The command specification (left of the configuration file) is mapped to a text object which types a backslash, followed by the text on the right of the configuration file, followed by a space. The extras are another element of dragonfly rules which allow for multiple options within each command specification. In this case, the command will be triggered whenever Dragon detects any of the words from `[tex_symbols1]` in `lyx.toml`.

# Updates
To save digging through old pull requests, I'll put the text of any major updates here.

## Fix integer bug - 5
The regular dragonfly IntegerRef class allows for example "162" to be dictated as "hundred sixty two", dropping the "one". This provides some convenience but when you allow multiple numbers side-by-side in a repeat rule, as mathfly does, it causes problems. Since Dragon cannot distinguish between "four hundred thirty three" and "four, hundred thirty three", it produces "4133" instead of "433" as intended.

I've re-implemented the class in `lib/integers.py` so that it now requires "133" to be dictated as "one hundred thirty three", solving the problem. It should now be possible to dictate numbers any way you want:

* `four six two`
* `four sixty two`
* `four hundred sixty two`
* `forty six two`