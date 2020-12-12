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
- embeddable into shell syntax (bash, fish, ... not xonsh cause all bracket types are already taken)
  - that's why we use [] instead of ()
- call lord script functions from command line
- easily typeable for interactive use
- interpreter, highly dynamic, no safety, interactive, easy to embed
- functional, lazy evaluation, no explicit flow control constructs
  - or do explicit evaluation, then both eager and lazy evaluation is possible?!
- one data structure (container): ordered dictionary
- code is data!!! (need to quote code to mark is as data or vice versa -> explicit/lazy evaluation)
  [f, a1, a2]
  [f, a1=foo, a2=78.1]
  [f, foo, a2=bar]
  f foo a2=bar (for top level dicts only)
  a1=foo
  [f, [add, 1, 2]]

- ambiguities, check which to allow
  [f foo a2=bar] instead of [f, foo, a2=bar] 
  a = 1 instead of a=1

- function definition
  [def, 
    [f, 
      [arg1, arg2],   # parameters 
      [res=[add, arg1, arg2]]   # keeps symbol "res" as a reference to result of function "add" 
      [ret, res]
    ]
  ]
  evaluation of [f, ...] is deferred (no need to quote it with "'" ... ?!)

- reserved words
  - keywords
      def, ' (quote function)
  - standard lib functions
    def, ', el

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
  payload is of type string with binary encoding

- overloaded functions / inheritance
  symbol entries with same key in ordered dict 

- closures
  closure=[my_function_called_with_an_argument]    # evaluates to a dict with partial substituted args (currying)

- types / templates / inheritance
  meta dicts like in Lua

- LordRPC


TODO
===============================================================================

1. Parser
-------------------------------------------------------------------------------

- [ ] write some example scripts for parser tests
