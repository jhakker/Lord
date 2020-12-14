..
   File              : README.rst
   Author            : Jörg Bakker <jorg@hakker.de>
   Date              : 12.12.2020
   Last Modified Date: 12.12.2020
   Last Modified By  : Jörg Bakker <jorg@hakker.de>

LordScript README
===============================================================================

0 General
-------------------------------------------------------------------------------

- LORD (Lisp with ORdered Dicts)
- embeddable into other languages on a source code level:
  - shell syntax (bash, fish, ... not xonsh cause all bracket types are already taken)
    - that's why we use [] instead of () here
  - different frontends for embedding into different languages (python, javascript, shell) 
    with different syntax (maybe switchable through pragma statements)
    - call lord script functions from command line
  - show current ways of generally embedding a language in another
    - tagged sections (<script>, php, EOF EOF blocks ...)
      - needs support of the runtime
    - strings
      - tedious to do parameter conversions
  - we use other higher language primitives than strings common to most languages (lists, maps) 
- code is data!!! (need to quote code to mark is as data or vice versa -> explicit/lazy evaluation)
  [f, a1, a2]
  [f, a1=foo, a2=78.1]
  [f, foo, a2=bar]
  f foo a2=bar (for top level dicts only)
  a1=foo
  [f, [add, 1, 2]]
- interpreter, highly dynamic, no safety, interactive, easy to embed
- easily typeable for interactive use
- RPC
  - build in support for rpc
  - binary data without transcoding to text representations
- functional, lazy evaluation, no explicit flow control constructs
  - or do explicit evaluation, then both eager and lazy evaluation is possible?!
- one data structure (container): ordered dictionary
- ambiguities, check which to allow
  [f foo a2=bar] instead of [f, foo, a2=bar] 
  a = 1 instead of a=1

- function definition

::
  [def, 
    [f, 
      [arg1, arg2],   # parameters 
      [res=[add, arg1, arg2]]   # keeps symbol "res" as a reference to result of function "add" 
      [ret, res]
    ]
  ]
  evaluation of [f, ...] is deferred (no need to quote it with "'" ... ?!)

def not needed, just the parameters and the code as dicts

- reserved words
  .. - keywords
  ..     def, ' (quote function)
  .. - standard lib functions
  ..   def, ', at
  
three operators to interpret the dict data as code:
  1. resolve symbol: string is not a literal string but looked up in the scope for an already defined symbol
     typically this is the $ operator, which conflicts with shells
  2. call: call a dict as a function, that is resolve the first element as a symbol for the function definition
     and put the remaining elements as function arguments. Could be a function [call func_name arg1 arg2 ...]
     but need to differ between assigning a dict to a position indexed field of a dict and calling it:
     my_dict=[
       [my_func a b]   # stored at position 0 of my_dict 
     ]
     my_dict=[
       $[my_func a b]   # calls my_func and stores the result at position 0 of my_dict 
     ]
  3. eval, can be a function: [eval arg1 arg2 ...] all args are evaluated, standard behaviour for top level
     of a module which feeds the REPL.

- execution context
  - two different execution contexts: 
    - eval all entries of a module's dict
    - call the dict
  - provide heap and/or stack for the execution context

- elementary data types
  - string, int, float
  - strings can be quoted with " but don't need to (e.g. quote white space)
  - numbers are parsed with their string representation

- one container type (ordered dict) only which also represents code
  - similar to Lua, but simpler and list is mixed with dict (same index)
  - access
    list1=[e1, e2, e3]   # define list
    el list1 1               # "el" function returns list element with index 1
    # note: passing a string to function without interpreting the argument as a symbol
    # then we need to quote it:
    # some_function, 'list1
    list2=[e1, e2='foostr, e3]   # "named" elements are (key,val) pairs of the dict
    list2=[e1, e2="foostr", e3]  # this should also work?!
    el, list2, 'e2
    el, list2, "e2"   # this should also work?!
  - ordered dict is a multimap
    - keys don't have to be unique
    - keys can be empty "", function call, lambda function definitions
  - declarative representation in other languages or JSON e.g. as list of two-tuples

::
    [
    ["", "string"],
    ["var1", "value1"]
    ]

- multiline
  first level ("main" dict)
    f
    v1=1
    v2=2
    
  higher level
    [f,
     a1,
     a2]

- comments
  # comment

- control flow
  a=1
  b=2
  for [range, 1, 10] [code_as_ordered_dict]
  if [eq, a, b] [code_if_true] [code_if_false]

- pragma / meta programming / interpreter directives
  each line starts with #@ and the rest of the line is valid lordscript
  #@ encoding='UTF8
  #@ brackets='[]{}            # add {} as valid brackit charackters -> JSON compatibility
  #@ module='my_module
  #@ include 'my_module
  #@ payload_checksum='#A12F   # checksum of binary payload in hex
  ... some code inbetween ...
  #@ payload_size='#C234       # size of payload in bytes, must be the last statement 
                               # in the code text section
  #@ heap_size=                # just dump the current heap instead of using a payload
                               # then init the module at start with the heap dump
  #@ heap_dump=                # url to heap dump
  #@ stack_size=               # same for the stack (complete process dump)
  payload / heap / stack is of type string with binary encoding

- overloaded functions / inheritance
  symbol entries with same key in ordered dict 

- closures
  closure=[my_function_called_with_an_argument]    # evaluates to a dict with partial substituted args (currying)

- types / templates / inheritance
  meta dicts like in Lua


1 First step: language translators
-------------------------------------------------------------------------------
1.1 JSON lists to LISP

reserved words:
list
escape a list (avoid function call on list) by making a list out of the arguments
[list 1 2 3] returns the list [1 2 3]

block
interpret a list using eval on each entry
[block [[setq a 1] [setq b 2]]]

symbol resolution:
instead of escaping the symbol with ', we explicitely resolve it with @
@a

TODO
===============================================================================

- [ ] write some example scripts for parser tests
- [ ] write converters:
  - JSON lists to LISP
  - write examples in python and node by using built in data initializers
  - Lua tables to LISP
- [ ] specify Lord syntax using a lean ordered dict notation and write parser
