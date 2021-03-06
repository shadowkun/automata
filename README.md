# Automata

*Copyright 2018 Caleb Evans*  
*Released under the MIT license*

[![Build Status](https://travis-ci.org/caleb531/automata.svg?branch=master)](https://travis-ci.org/caleb531/automata)
[![Coverage Status](https://coveralls.io/repos/caleb531/automata/badge.svg?branch=master)](https://coveralls.io/r/caleb531/automata?branch=master)

Automata is a Python 3 library which implements the structures and algorithms
for finite automata, pushdown automata, and Turing machines.

Automata requires Python 3.4 or newer.

## Migration to v2

If you are using Automata v1, please note that there are some significant
changes in v2 to refine the API. If you wish to upgrade, please follow the
[migration guide](https://github.com/caleb531/automata/blob/v2/MIGRATION.md).

## Installing

You can install the latest version of Automata via pip:

```
pip install automata-lib
```

## API

- [class Automaton](#class-automatonmetaclassabcmeta)
  - [class FA](#class-faautomatonmetaclassabcmeta)
    - [class DFA](#class-dfafa)
    - [class NFA](#class-nfafa)
  - [class PDA](#class-pdaautomatonmetaclassabcmeta)
    - [class DPDA](#class-dpdapda)
  - [class TM](#class-tmautomatonmetaclassabcmeta)
    - [class DTM](#class-dtmtm)
- [Exception classes](#base-exception-classes)
    - [Turing machine exceptions](#turing-machine-exception-classes)

### class Automaton(metaclass=ABCMeta)

The `Automaton` class is an abstract base class from which all automata
(including Turing machines) inherit. As such, it cannot be instantiated on its
own; you must use a defined subclasses instead (or you may create your own
subclass if you're feeling adventurous). The `Automaton` class can be found
under `automata/base/automaton.py`.

If you wish to subclass `Automaton`, you can import it like so:

```python
from automata.base.automaton import Automaton
```

The following methods are common to all Automaton subtypes:

#### Automaton.read_input(self, input_str)

Reads an input string into the automaton, returning the automaton's final
configuration (according to its subtype). If the input is rejected, the method
raises a `RejectionException`.

#### Automaton.read_input_stepwise(self, input_str)

Reads an input string like `read_input()`, except instead of returning the final
configuration, the method returns a generator. The values yielded by this
generator depend on the automaton's subtype.

If the string is rejected by the automaton, the method still raises a
`RejectionException`.

#### Automaton.accepts_input(self, input_str)

Reads an input string like `read_input()`, except it returns a boolean instead
of returning the automaton's final configuration (or raising an exception). That
is, the method always returns `True` if the input is accepted, and it always
returns `False` if the input is rejected.

#### Automaton.validate(self)

Checks whether the automaton is actually a valid automaton (according to its
subtype). It returns `True` if the automaton is valid; otherwise, it will raise
the appropriate exception (*e.g.* the state transition is missing for a
particular symbol).

This method is automatically called when the automaton is initialized, so it's
only really useful if a automaton object is modified after instantiation.

#### Automaton.copy(self)

Returns a deep copy of the automaton according to its subtype.

### class FA(Automaton, metaclass=ABCMeta)

The `FA` class is an abstract base class from which all finite automata inherit.
The `FA` class can be found under `automata/fa/fa.py`.

If you wish to subclass `FA`, you can import it like so:

```python
from automata.fa.fa import FA
```

### class DFA(FA)

The `DFA` class is a subclass of `FA` and represents a deterministic finite
automaton. It can be found under `automata/fa/dfa.py`.

Every DFA has the following (required) properties:

1. `states`: a `set` of the DFA's valid states, each of which must be
represented as a string

2. `input_symbols`: a `set` of the DFA's valid input symbols, each of which must
also be represented as a string

3. `transitions`: a `dict` consisting of the transitions for each state. Each
key is a state name and each value is a `dict` which maps a symbol (the key) to
a state (the value).

4. `initial_state`: the name of the initial state for this DFA

5. `final_states`: a `set` of final states for this DFA

```python
from automata.fa.dfa import DFA
# DFA which matches all binary strings ending in an odd number of '1's
dfa = DFA(
    states={'q0', 'q1', 'q2'},
    input_symbols={'0', '1'},
    transitions={
        'q0': {'0': 'q0', '1': 'q1'},
        'q1': {'0': 'q0', '1': 'q2'},
        'q2': {'0': 'q2', '1': 'q1'}
    },
    initial_state='q0',
    final_states={'q1'}
)
```

#### DFA.read_input(self, input_str)

Returns the final state the DFA stopped on, if the input is accepted.

```python
dfa.read_input('01')  # returns 'q1'
```

```python
dfa.read_input('011')  # raises RejectionException
```

#### DFA.read_input_stepwise(self, input_str)

Yields each state reached as the DFA reads characters from the input string, if
the input is accepted.

```python
dfa.read_input_stepwise('0111')
# yields:
# 'q0'
# 'q0'
# 'q1'
# 'q2'
# 'q1'
```

#### DFA.accepts_input(self, input_str)

```python
if dfa.accepts_input(my_input_str):
    print('accepted')
else:
    print('rejected')
```

#### DFA.validate(self)

```python
dfa.validate()  # returns True
```

#### DFA.copy(self)

```python
dfa.copy()  # returns deep copy of dfa
```

#### DFA.minify(self)

Creates a minimal DFA which accepts the same inputs as the old one.
Unreachable states are removed and equivalent states are merged.

```python
minimal_dfa = dfa.minify()
```

#### DFA.from_nfa(cls, nfa)

Creates a DFA that is equivalent to the given NFA.

```python
from automata.fa.dfa import DFA
from automata.fa.nfa import NFA
dfa = DFA.from_nfa(nfa)  # returns an equivalent DFA
```

### class NFA(FA)

The `NFA` class is a subclass of `FA` and represents a nondeterministic finite
automaton. It can be found under `automata/fa/nfa.py`.

Every NFA has the same five DFA properties: `state`, `input_symbols`,
`transitions`, `initial_state`, and `final_states`. However, the structure of
the `transitions` object has been modified slightly to accommodate the fact that
a single state can have more than one transition for the same symbol. Therefore,
instead of mapping a symbol to *one* end state in each sub-dict, each symbol is
mapped to a *set* of end states.

```python
from automata.fa.nfa import NFA
# NFA which matches strings beginning with 'a', ending with 'a', and containing
# no consecutive 'b's
nfa = NFA(
    states={'q0', 'q1', 'q2'},
    input_symbols={'a', 'b'},
    transitions={
        'q0': {'a': {'q1'}},
        # Use '' as the key name for empty string (lambda/epsilon) transitions
        'q1': {'a': {'q1'}, '': {'q2'}},
        'q2': {'b': {'q0'}}
    },
    initial_state='q0',
    final_states={'q1'}
)
```

#### NFA.read_input(self, input_str, step=False)

Returns a set of final states the FA stopped on, if the input is accepted.

```python
nfa.read_input('aba')  # returns {'q1', 'q2'}
```

```python
nfa.read_input('abba')  # raises RejectionException
```

#### NFA.read_input_stepwise(self, input_str)

Yields each set of states reached as the NFA reads characters from the input
string, if the input is accepted.

```python
nfa.read_input_stepwise('aba')
# yields:
# {'q0'}
# {'q1', 'q2'}
# {'q0'}
# {'q1', 'q2'}
```

#### NFA.accepts_input(self, input_str)

```python
if nfa.accepts_input(my_input_str):
    print('accepted')
else:
    print('rejected')
```

#### NFA.validate(self)

```python
nfa.validate()  # returns True
```

#### NFA.copy(self)

```python
nfa.copy()  # returns deep copy of nfa
```

#### NFA.from_dfa(cls, dfa)

Creates an NFA that is equivalent to the given DFA.

```python
from automata.fa.nfa import NFA
from automata.fa.dfa import DFA
nfa = NFA.from_dfa(dfa)  # returns an equivalent NFA
```

#### class PDA(Automaton, metaclass=ABCMeta)

The `PDA` class is an abstract base class from which all pushdown automata
inherit. It can be found under `automata/pda/pda.py`.

### class DPDA(PDA)

The `DPDA` class is a subclass of `PDA` and represents a deterministic finite
automaton. It can be found under `automata/pda/dpda.py`.

Every DPDA has the following (required) properties:

1. `states`: a `set` of the DPDA's valid states, each of which must be
represented as a string

2. `input_symbols`: a `set` of the DPDA's valid input symbols, each of which
must also be represented as a string

3. `stack_symbols`: a `set` of the DPDA's valid stack symbols

4. `transitions`: a `dict` consisting of the transitions for each state; see the
example below for the exact syntax

5. `initial_state`: the name of the initial state for this DPDA

6. `initial_stack_symbol`: the name of the initial symbol on the stack for this
DPDA

7. `final_states`: a `set` of final states for this DPDA

```python
from automata.pda.dpda import DPDA
# DPDA which which matches zero or more 'a's, followed by the same
# number of 'b's (accepting by final state)
dpda = DPDA(
    states={'q0', 'q1', 'q2', 'q3'},
    input_symbols={'a', 'b'},
    stack_symbols={'0', '1'},
    transitions={
        'q0': {
            'a': {'0': ('q1', ('1', '0'))}  # transition pushes '1' to stack
        },
        'q1': {
            'a': {'1': ('q1', ('1', '1'))},
            'b': {'1': ('q2', '')}  # transition pops from stack
        },
        'q2': {
            'b': {'1': ('q2', '')},
            '': {'0': ('q3', ('0',))}  # transition does not change stack
        }
    },
    initial_state='q0',
    initial_stack_symbol='0',
    final_states={'q3'}
)
```

#### DPDA.read_input(self, input_str, step=False)

Returns a tuple containing the final state the DPDA stopped on, as well as a
`PDAStack` object representing the DPDA's stack (if the input is accepted).

```python
dpda.read_input('ab')  # returns ('q3', PDAStack(['0']))
```

```python
dpda.read_input('aab')  # raises RejectionException
```

#### DPDA.read_input_stepwise(self, input_str)

Yields tuples containing the current state and the current stack as a `PDAStack`
object, if the input is accepted.

```python
((state, stack.copy()) for state, stack in dpda.read_input_stepwise('ab'))
# yields:
# ('q0', PDAStack(['0']))
# ('q1', PDAStack(['0', '1']))
# ('q3', PDAStack(['0']))
```

Please note that each tuple contains a reference to (not a copy of) the current
`PDAStack` object. Therefore, if you wish to store the stack at every step, you
must copy the stack as you iterate over the automaton configurations (as shown
above).

#### DPDA.accepts_input(self, input_str)

```python
if dpda.accepts_input(my_input_str):
    print('accepted')
else:
    print('rejected')
```

#### DPDA.validate(self)

```python
dpda.validate()  # returns True
```

#### DPDA.copy(self)

```python
dpda.copy()  # returns deep copy of dpda
```

### class TM(Automaton, metaclass=ABCMeta)

The `TM` class is an abstract base class from which all Turing machines inherit.
It can be found under `automata/tm/tm.py`.

### class DTM(TM)

The `DTM` class is a subclass of `TM` and represents a deterministic Turing
machine. It can be found under `automata/tm/dtm.py`.

Every DTM has the following (required) properties:

1. `states`: a `set` of the DTM's valid states, each of which must be
represented as a string

2. `input_symbols`: a `set` of the DTM's valid input symbols represented as
strings

3. `tape_symbols`: a `set` of the DTM's valid tape symbols represented as
strings

4. `transitions`: a `dict` consisting of the transitions for each state; each
key is a state name and each value is a `dict` which maps a symbol (the key) to
a state (the value)

5. `initial_state`: the name of the initial state for this DTM

6. `blank_symbol`: a symbol from `tape_symbols` to be used as the blank symbol
for this DTM

7. `final_states`: a `set` of final states for this DTM

```python
from automata.tm.dtm import DTM
# DTM which matches all strings beginning with '0's, and followed by
# the same number of '1's
dtm = DTM(
    states={'q0', 'q1', 'q2', 'q3', 'q4'},
    input_symbols={'0', '1'},
    tape_symbols={'0', '1', 'x', 'y', '.'},
    transitions={
        'q0': {
            '0': ('q1', 'x', 'R'),
            'y': ('q3', 'y', 'R')
        },
        'q1': {
            '0': ('q1', '0', 'R'),
            '1': ('q2', 'y', 'L'),
            'y': ('q1', 'y', 'R')
        },
        'q2': {
            '0': ('q2', '0', 'L'),
            'x': ('q0', 'x', 'R'),
            'y': ('q2', 'y', 'L')
        },
        'q3': {
            'y': ('q3', 'y', 'R'),
            '.': ('q4', '.', 'R')
        }
    },
    initial_state='q0',
    blank_symbol='.',
    final_states={'q4'}
)
```

#### DTM.read_input(self, input_str, step=False)

Returns a tuple containing the final state the machine stopped on, as well as a
`TMTape` object representing the DTM's internal tape (if the input is accepted).

```python
dtm.read_input('01')  # returns ('q4', TMTape('xy.'))
```

```python
dtm.read_input('011')  # raises RejectionException
```

#### DTM.read_input_stepwise(self, input_str)

Yields a tuple containing the current state and the current tape as a `TMTape`
object.

```python
(state, tape.copy()) for state, tape in dtm.read_input_stepwise('01')
# yields:
# ('q0', TMTape('01'))
# ('q1', TMTape('x1'))
# ('q2', TMTape('xy'))
# ('q0', TMTape('xy'))
# ('q3', TMTape('xy'))
# ('q3', TMTape('xy.'))
```

Please note that each tuple contains a reference to (not a copy of) the current
`TMTape` object. Therefore, if you wish to store the tape at every step, you
must copy the tape as you iterate over the machine configurations (as shown
above).

#### DTM.accepts_input(self, input_str)

```python
if dtm.accepts_input(my_input_str):
    print('accepted')
else:
    print('rejected')
```

#### DTM.validate(self)

```python
dtm.validate()  # returns True
```

#### DTM.copy(self)

```python
dtm.copy()  # returns deep copy of dtm
```

### Base exception classes

The library also includes a number of exception classes to ensure that errors
never pass silently (unless explicitly silenced). See
`automata/base/exceptions.py` for these class definitions.

To reference these exceptions (so as to catch them in a `try..except` block or
whatnot), simply import `automata.base.exceptions` however you'd like:

```python
import automata.base.exceptions as exceptions
```

#### class AutomatonException

A base class from which all other automata exceptions inherit (including finite
automata and Turing machines).

#### class InvalidStateError

Raised if a state is not a valid state for this automaton.

#### class InvalidSymbolError

Raised if a symbol is not a valid symbol for this automaton.

#### class MissingStateError

Raised if a state is missing from the automaton definition.

#### class MissingSymbolError

Raised if a symbol is missing from the automaton definition.

#### class InitialStateError

Raised if the initial state fails to meet some required condition for this type
of automaton.

#### class FinalStateError

Raised if a final state fails to meet some required condition for this type of
automaton.

#### class RejectionException

Raised if the automaton stopped on a non-final state after validating input.

### Turing machine exception classes

The `automata.tm` package also includes a module for exceptions specific to
Turing machines. You can reference these exception classes like so:

```python
import automata.tm.exceptions as tm_exceptions
```

#### class TMException

A base class from which all other Turing machine exceptions inherit.

#### class InvalidDirectionError

Raised if a direction specified in this machine's transition map is not a valid
direction.
