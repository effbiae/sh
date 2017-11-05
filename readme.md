# p 

- a native library to interface kdb+ and python
- plus an experimental convenience wrapper

## Goal

Make it easier to use python libraries in q than in native python.

## Contents

  - [p.c](p.c) a library to interface kdb+ and python
  - [p.k](p.k) adds a 'pythonic' feel to python in q (experimental)
  - [p.q](p.q) load this file to define .p (deprecated - use pyutil)
  - [t.q](t.q) contains tests for `make test`
  - [configure](configure) (and [configure.q](configure.q)) a script to produce a makefile to build the library

p.q and p.k are independent: each depends only on the native library

## Quick start

  `$ ./configure && make test`

### Two ways of doing the same thing:

```
  $ q p.k
  q).p.import`sys
  `sys
  q)sys`:repr
  "<module 'sys' (built-in)>"
  q)sys[`stdout`write]"hi\n"
  hi
  3
```
- low level

```
  $ q p.q
  q).p.repr .p.import`sys
  "<module 'sys' (built-in)>"
  q).p.getj .p.call[.p.getattr[.p.getattr[.p.import`sys;`stdout];`write];enlist"hi\n";()!()]
  hi
  3
```


### Example

*Get the kx docs for adverbs and find the `<h2>` headings*

- in python

```
>>> import bs4,urllib    # bs4 is BeautifulSoup - a library to parse markup
>>> h=bs4.BeautifulSoup(urllib.request.urlopen('http://code.kx.com/q/ref/adverbs/'),'html.parser').find_all('h2')
>>> [x.attrs for x in h]
[{'id': 'case'}, {'id': 'compose'}, {'id': 'each-both'}, {'id': 'each-left'}, {'id': 'each-right'}, {'id': 'each-parallel'}, {'id': 'each-prior'}, {'id': 'converge-repeat'}, {'id': 'over'}, {'id': 'fold'}, {'id': 'converge-iterate'}, {'id': 'scan'}, {'id': 'derivatives'}]
```

- in q

```
  $ q p.k
  q).p.import`bs4
  q)`$bs4[`BeautifulSoup;.Q.hg`:http://code.kx.com/q/ref/adverbs/;"html.parser"][`$"find_all";"h2"]@\:`attrs
  id              
  ----------------
  case            
  compose         
  each-both       
  each-left       
  each-right      
  each-parallel   
  each-prior      
  converge-repeat 
  over            
  fold            
  converge-iterate
  scan            
  derivatives     
```

### A Closer Look

- .p.import is the only function needed to use python modules in q.

```
  q).p.import`bs4
  q)bs4[`:ismodule]  /first arg is symbol(s).  if `:, call function on object.
  1b
  q)bs4`:dir
  "BeautifulSoup"
  "BeautifulStoneSoup"
  "CData"
  "Comment"
  ..
  q)bs4`:repr
  "<module 'bs4' from '/usr/lib/python3/dist-packages/bs4/__init__.py'>"


  /first argument is getattr().  last args call the function
  q)bs4[`BeautifulSoup;"<html></html>";"html.parser"]`:repr
  "<html></html>"

  /get the foreign soup object
  q)bs4[`BeautifulSoup;"<html></html>";"html.parser"]`
  foreign

  /get a list of foreigns.  note the recursive object style calling - x.f(..).g(..)
  q)bs4[`BeautifulSoup;.Q.hg`:http://code.kx.com/q/ref/adverbs/ ;"html.parser"][`find_all;"h2"]@\:`
  foreign
  foreign
  foreign
  foreign
  ..

  q)bs4[`BeautifulSoup;.Q.hg`:http://code.kx.com/q/ref/adverbs/ ;"html.parser"][`find_all;"h2"]@\:`attrs
  id                
  ------------------
  "case"            
  "compose"         
  ..

  /passing foreigns to functions
  q)import`bs4
  q)bs:bs4[`BeautifulSoup]
  q)import`urllib
  q)s:bs[urllib[`request`urlopen;"http://code.kx.com/q/ref/adverbs/"]`;"html.parser"]
  q)h:s[`find_all;`h2]
```

## p.c

*c code may not documented?*

```
runs        | code
set         | code
import      | code
getattr     | code
..
```
