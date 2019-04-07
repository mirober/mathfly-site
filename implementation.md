---
layout: page
title: Implementation details
---

This page gives some details about the implementation of Mathfly, as well as keeping track of alterations as they happen. It will hopefully be of use if anybody needs to maintain or extend Mathfly.

There are two basic building blocks, apart from Dragon. [Natlink](https://qh.antenna.nl/unimacro/installation/installation.html), a C++ tool originally written by an employee of Dragon Systems called Joel Gould in 1999, allows grammars written in Python to be imported directly into Dragon. As far as I know Nuance does not endorse this but also hasn\'t made any effort to stop it, presumably because it\'s users still need to buy Dragon, which is all they really care about. Secondly, [dragonfly](https://github.com/dictation-toolbox/dragonfly) is a Python library which provides easy abstractions for building grammars to be imported using Natlink.

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

To this end, Mathfly uses a couple of elements from the [Caster](https://github.com/dictation-toolbox/Caster) voice programming project. The most important of these are the MergeRule and the CCRMerger. The caster documentation explains these in detail but the basic idea is that instead of simply adding a rule to a grammar and loading it, rules are registered with a central object which then manages them. This enables rules to be enabled and disabled with a voice command, and also allows for multiple rules to be merged together into one rule and then converted to enable continuous recognition.


The smart code for creating CCR rules is here, at the bottom of `ccrmerger.py`. Given an input rule, it creates a new rule with only one specification, which can be any number of repetitions of any of the commands in the input rule. When the rule is triggered, it executes each of the individual commands in sequence. Only this new rule is loaded in a grammar, not the input rule.
```
def _create_repeat_rule(self, rule):
    ORIGINAL, SEQ, TERMINAL = "original", "caster_base_sequence", "terminal"
    alts = [RuleRef(rule=rule)]
    single_action = Alternative(alts)
    max = SETTINGS["max_ccr_repetitions"]
    sequence = Repetition(single_action, min=1, max=max, name=SEQ)
    original = Alternative(alts, name=ORIGINAL)
    terminal = Alternative(alts, name=TERMINAL)

    class RepeatRule(CompoundRule):
        spec = "[<" + ORIGINAL + "> original] [<" + SEQ + ">] [terminal <" + TERMINAL + ">]"
        extras = [sequence, original, terminal]

        def _process_recognition(self, node, extras):
            original = extras[ORIGINAL] if ORIGINAL in extras else None
            sequence = extras[SEQ] if SEQ in extras else None
            terminal = extras[TERMINAL] if TERMINAL in extras else None
            if original is not None: original.execute()
            if sequence is not None:
                for action in sequence:
                    action.execute()
            if terminal is not None: terminal.execute()
```

## Configuration files
Another limitation of basic dragonfly rules is that they cannot be reloaded on-the-fly, requiring a full restart of Dragon for changes to be implemented, and that you need to edit Python source files to modify them.

Mathfly gets around this by defining the vast majority of commands in `.toml` configuration files. I had never used Vocola before creating Mathfly, but it was later pointed out that this is very similar to their system. Another example of convergent evolution I guess.

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
To save digging through old pull requests, I\'ll put the text of any major updates here.

## Fix integer bug - 5
The regular dragonfly `IntegerRef` class allows for example \"162\" to be dictated as \"hundred sixty two\", dropping the \"one\". This provides some convenience but when you allow multiple numbers side-by-side in a repeat rule, as Mathfly does, it causes problems. Since Dragon cannot distinguish between \"four hundred thirty three\" and \"four, hundred thirty three\", it produces \"4133\" instead of \"433\" as intended.

I\'ve re-implemented the class in `lib/integers.py` so that it now requires \"133\" to be dictated as \"one hundred thirty three\", solving the problem. It should now be possible to dictate numbers any way you want:

* `four six two`
* `four sixty two`
* `four hundred sixty two`
* `forty six two`


## Nested rules - 6
Cases have come up a few times where I wanted to create a rule which included arbitrary strings of other active CCR commands within the same rule. This is a first pass at implementing this and definitely needs some more abstraction.

Current examples for testing purposes:
* "sum from india equals one to november"
* "integral from minus infinity to infinity", "integral from two to four"
* "limit from november to infinity", "limit from x-ray to two"
* "differential x-ray by yankee", "differential big lima squared by squared greek beta"

At the moment these accept other commands before them but require a pause afterwards, because otherwise they can't distinguish where to finish the last subcomponent.

Update:
I have added a new NestedRule class in `lib/merge/nestedrule` to achieve this. This rule type allows for the creation of rules which can take arbitrary sequences of commands from another rule. They are declared without extras and added in the `nested` property of a merge rule (same as `non`), the merged form of which will be used to form the extras list in `ccrmerger._create_repeat_rule`.

The specifications in the rule\'s mapping must include `<sequence1>` and `<sequence2>`, `<before>` and `<after>` are optional and allow for other commands to be spoken before or after the rule.
Example command:
```
"[<before>] integral from <sequence1> to <sequence2>":
    [Text("\\int _"), Key("right, caret"), Key("right")],
```

Any commands which come before will be executed first, then the first action in the list, then the first sequence, then the second action in the list, then the second sequence, then the final action.  At the moment sequences have a maximum length of 6 commands and before and after 8.

When the RepeatRule for the main rule is created, the following adds `Repetition` elements of all of its commands to the extras of the nested rule. I\'m not 100% happy with the fact that the names and maximums for this are hardcoded, but at the moment I\'m erring on the side of simplicity and assuming that the cases where this will be useful will be limited and soon exhausted.

```
if rule.nested is not None:
    bef  = Repetition(single_action, min=1, max=8, name="before")
    aft  = Repetition(single_action, min=1, max=8, name="before")
    seq1 = Repetition(single_action, min=1, max=6, name="sequence1")
    seq2 = Repetition(single_action, min=1, max=6, name="sequence2")

    rule.nested.extras = [bef, aft, seq1, seq2]
    nested = rule.nested()
    rules.append(nested)
```

## Vocabulary bug - 7
A slightly strange bug was found recently. The new nested rules were all working apart from any which contained the phrase \"one to one\". \"one two one\" was found to also not work, but instead produced the text \"one-to-one\".

There\'s the problem. Dragon is interpreting \"one-to-one\" and is somehow missing the fact that this string of words could be interpreted in a different way. It would obviously be possible to hardcode \"integral from one-to-one\" etc, but this would be a major pain and wouldn\'t scale. It is also possible to simply delete the word from your vocabulary, but this leaves a confusing problem for every user who ever tries to use a command with that phrase in.

Luckily, and with a pointer from @danesprite, natlink exposes a function for deleting words from the active vocabulary. There is now a `delete_words` list in `settings.toml` and on start-up the following will be run to delete all of the words in it from the active vocabulary (`_mathfly_main.py`):
```
import natlink
SETTINGS = utilities.load_toml_relative("config/settings.toml")
    for word in SETTINGS["delete_words"]:
        try:
            natlink.deleteWord(word)
        except:
            pass
```

At the moment the list only contains \"one-to-one\", but this may be expanded if we find any more of these.