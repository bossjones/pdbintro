.. include:: <s5defs.txt>

Introduction to PDB
===================

:Authors: Chris McDonough, Agendaless Consulting
:Date: 3/9/2012 (PyCon 2012)

..  footer:: Chris McDonough, Agendaless Consulting

What is PDB
-----------

- Interactive shell.

- Gives you a glimpse into the execution path of your (and others'!) code.

- Allows you to display and modify the state of variables during execution.

- Allows you to execute expressions in the context of your code.

- If you don't know how to use it, you will after this talk.

- You can be a lot more self-sufficient if you know how to use PDB even a
  little bit.

When Should I Use It
--------------------

- "I don't use a debugger"

- When is it really not reasonable?

- Alternatives: print statements, logging.

buggy.py
--------

::

  def divide_one_by(divisor):
      return 1/divisor

  if __name__ == '__main__':
      divide_one_by(0)

Invoking PDB
------------

- Script mode

- Postmortem mode

- Run mode

- Trace mode

Script Mode
-----------

``python -m pdb buggy.py``

  steps through the entire script in pdb.

::

  [chrism@thinko pdb]$ ../env27/bin/python -m pdb buggy.py 
  > buggy.py(1)<module>()
  -> def divide_one_by(divisor):
 (Pdb) 

Enhanced Script Mode (Py 3.2+)
-----------------------------------

``python -m pdb -c continue buggy.py``

  lets the script run until an exception occurs, then presents prompt.

::

 [chrism@thinko pdb]$ ../env32/bin/python -m pdb -c continue buggy.py 
 Traceback (most recent call last):
 ...
 ZeroDivisionError: division by zero
 ...
 > buggy.py(2)divide_one_by()
 -> return 1/divisor
 (Pdb) 

Enhanced Script Mode (Py 3.2+)
------------------------------

``python -m pdb -c 'until 2' buggy.py``

  lets the script run until it reaches line 2, then presents prompt.

::

 [chrism@thinko pdb]$ ../env32/bin/python -m pdb -c 'until 2' buggy.py 
 > buggy.py(4)<module>()
 -> if __name__ == '__main__':
 (Pdb)

``-c`` commands are actually ``.pdbrc`` commands on the command line (they
run after the commands in any ``.pdbrc`` file).

Postmortem Mode
---------------

``pdb.pm()``

  Used after an uncaught exception has been raised.

::

  [chrism@thinko pdb]$ python -i buggy.py
  Traceback (most recent call last):
    ...
  ZeroDivisionError: integer division or modulo by zero
  >>> import pdb; pdb.pm()
  > buggy.py(2)divide_one_by()
  -> return 1/divisor
  (Pdb)

Run Mode
--------

``pdb.run('some.expression()')``.

  Executes ``some.expression()`` under pdb control.  Uses current locals
  and globals to interpret the expression.

::

  >>> import buggy, pdb; pdb.run('buggy.divide_one_by(0)')
  > <string>(1)<module>()
  (Pdb) s

Trace Mode
----------

``pdb.set_trace()``

- Convenient to stick in development and test code.

- Just another Python expression so you can conditionalize its execution.

- I use this mode all the time.  We'll be using it from now on.

::

  def divide_one_by(divisor):
      import pdb; pdb.set_trace()
      return 1/divisor

Command Language
----------------

- ``readline`` is used in the debugger so arrow keys and other readline
  keybindings work (ctrl-a, ctrl-e, etc in emacs mode).

- Aliases can be used.

- Blank line repeats the last command issued (unless it was a ``list``
  operation).

- Full command name or its shortened version can be used at ``(Pdb)`` prompt.

- Exceptions generated when a pdb expression generates an exception don't
  impact current program state.

Help
----

::

  (Pdb) h

  Documented commands (type help <topic>):
  ========================================
  EOF    bt         cont      enable  jump  pp       run      unt   
  a      c          continue  exit    l     q        s        until 
  alias  cl         d         h       list  quit     step     up    
  args   clear      debug     help    n     r        tbreak   w     
  b      commands   disable   ignore  next  restart  u        whatis
  break  condition  down      j       p     return   unalias  where 

http://blog.doughellmann.com/2010/09/pymotw-pdb-interactive-debugger.html

(Google for "hellmann pdb")

Where Am I?
-----------

- ``list (l)``: displaying code in your current execution context

- ``where (w)``: showing the current location in the frame stack.

Command: list (l)
------------------

``->`` shown beside the line that will execute next.

::

  [chrism@thinko pdb]$ ../env27/bin/python buggy.py
  > buggy.py(3)divide_one_by()
  -> return 1/divisor
  (Pdb) l
    1  	def divide_one_by(divisor):
    2  	    import pdb; pdb.set_trace()
    3  ->	    return 1/divisor
    4  	

Command: where (w)
------------------

Where in the frame stack am I?  Read like a traceback.

::

  [chrism@thinko pdb]$ ../env27/bin/python buggy.py
  > buggy.py(3)divide_one_by()
  -> return 1/divisor
  (Pdb) w
    buggy.py(6)<module>()
  -> divide_one_by(0)
  > buggy.py(3)divide_one_by()
  -> return 1/divisor
  (Pdb) 

Displaying Things
-----------------

- ``p`` and ``pp``: displaying objects.  Accepts expression.

- ``args``: print values of current function's arguments.

Command: print (p)
------------------

::

  [chrism@thinko pdb]$ ../env27/bin/python buggy.py
  > buggy.py(3)divide_one_by()
  -> return 1/divisor
  (Pdb) p divisor
  0

Note that ``print`` != ``p``.  ``print`` is passed through to the
interpreter; ``p`` is executed by pdb.

Command: pretty-print (pp)
---------------------------

Use when results of ``p`` are too ugly.

::

  (Pdb) pp ['a'] * 16
  ['a',
   'a',
   'a',
   ...
   'a',
   'a',
   'a']

Command:  args (a)
------------------

Prints argument values passed to the current function.

::

  def divide_one_by(divisor):
      import pdb; pdb.set_trace()
      return 1/divisor

::

  [chrism@thinko pdb]$ ../env27/bin/python buggy.py
  > buggy.py(3)divide_one_by()
  -> return 1/divisor
  (Pdb) a
  divisor = 0

Execution Control
-----------------

- ``step``, ``next``, ``continue``, ``until``, ``return``

Command: step (s)
------------------

- Execute the current line, then stop at the next execution point.

- The next execution point is the first statement inside a function being
  called or the next line of the current function.

::

  > buggy_traced_main.py(6)<module>()
  -> divide_one_by(0)
  (Pdb) s
  --Call--
  > buggy_traced_main.py(1)divide_one_by()
  -> def divide_one_by(divisor):
  (Pdb) s
  > buggy_traced_main.py(2)divide_one_by()
  -> return 1/divisor

Command: next (n)
-----------------

- Like ``step``, but does not enter functions called from statement being
  executed.

- "Skips" debugging function calls.

:: 

  > buggy_traced_main.py(6)<module>()
  -> divide_one_by(0)
  (Pdb) n
  ZeroDivisionError: 'integer division or modulo by zero'
  > buggy_traced_main.py(6)<module>()
  -> divide_one_by(0)

Command: continue (c)
---------------------

- Continue until the next breakpoint.

- If no breakpoint exists, continue until exit or uncaught exception is
  raised.

::

  > buggy_traced.py(3)divide_one_by()
  -> return 1/divisor
  (Pdb) c
  Traceback (most recent call last):
    File "buggy_traced.py", line 6, in <module>
      divide_one_by(0)
    File "buggy_traced.py", line 3, in divide_one_by
      return 1/divisor
  ZeroDivisionError: integer division or modulo by zero

Command: until
--------------

- Like ``next`` but explicitly continues until execution reaches a line in
  the same function with a line number higher than the current one.

-  Basically "step past the end of the current loop".

Command: return (r)
-------------------

Continues executing until current function is about to execute a ``return``
statement then pauses.

::

  > buggy_traced.py(3)divide_one_by()
  -> return 1/divisor
  (Pdb) r
  --Return--
  > buggy_traced.py(3)divide_one_by()->None
  -> return 1/divisor
  (Pdb) l
    1  	def divide_one_by(divisor):
    2  	    import pdb; pdb.set_trace()
    3  ->	    return 1/divisor

Frame Stack Control
-------------------

- ``up``, ``down``: navigating the frame stack

Command: up (u)
---------------

::

  (Pdb) l
    1  	def divide_one_by(divisor):
    2  	    import pdb; pdb.set_trace()
    3  ->	    return 1/divisor
    ...
  (Pdb) u
  > buggy.py(6)<module>()
  -> divide_one_by(0)
  (Pdb) l
    ...
    5  	if __name__ == '__main__':
    6  ->	    divide_one_by(0)
    7  	    

Command: down (d)
-----------------

::

  (Pdb) l
    ...
    5  	if __name__ == '__main__':
    6  ->	    divide_one_by(0)
    7  	    
  (Pdb) d
  > buggy.py(3)divide_one_by()
  -> return 1/divisor
  (Pdb) l
    1  	def divide_one_by(divisor):
    2  	    import pdb; pdb.set_trace()
    3  ->	    return 1/divisor
    ...


Modifying Variables
-------------------

- ``!``-prefixing: pass directly to interpreter, used often to modify
  variables.

::

  [chrism@thinko pdb]$ ../env27/bin/python buggy.py
  > buggy.py(3)divide_one_by()
  -> return 1/divisor
  (Pdb) p divisor
  0
  (Pdb) !divisor=1
  (Pdb) p divisor
  1
  (Pdb) n

You Can Leave Now
-----------------

- The commands we've already covered will go a long, long way.

- The rest of the commands are useful but not *critical*.

Breakpoints
-----------

- ``set_trace`` is great to drop you into pdb, but you can also use
  *breakpoints* to spell a condition about where else to stop in advance of
  actually executing.

- When breakpoint is reached, the program is paused before the line
  identified by the breakpoint is executed.

- ``break``, ``tbreak``, ``ignore``, ``enable``, ``disable``, and
  ``clear``

Command: break <lineno>
------------------------

- break <lineno>

- Stops execution at lineno.

::

  > buggy_traced_main.py(6)<module>()
  -> divide_one_by(0)
  (Pdb) break 2
  Breakpoint 1 at buggy_traced_main.py:2
  (Pdb) c
  > buggy_traced_main.py(2)divide_one_by()
  -> return 1/divisor

Command: break <funcname>
-------------------------

- break <funcname>

- Stops execution withinin function funcname.

::

  > buggy_traced_main.py(6)<module>()
  -> divide_one_by(0)
  (Pdb) break divide_one_by
  Breakpoint 1 at buggy_traced_main.py:1
  (Pdb) c
  > buggy_traced_main.py(2)divide_one_by()
  -> return 1/divisor
  (Pdb) 

Command: break <fn:line>
--------------------------

- Can be full path to source file, relative path, or file available on
  sys.path.

::

  > buggy_traced_main.py(6)<module>()
  -> divide_one_by(0)
  (Pdb) break buggy.py:2
  Breakpoint 1 at buggy_traced_main.py:1
  (Pdb) c
  > buggy_traced_main.py(2)divide_one_by()
  -> return 1/divisor
  (Pdb) 

Command: break
--------------------------------

- Lists current breakpoints.

::

  > buggy_traced_main.py(6)<module>()
  -> divide_one_by(0)
  (Pdb) break 2
  Breakpoint 1 at buggy_traced_main.py:2
  (Pdb) break
  Num Type         Disp Enb   Where
  1   breakpoint   keep yes   at buggy_traced_main.py:2
  (Pdb) 


Command: disable
--------------------------------

- Disables a breakpoint by number

::

  (Pdb) disable 1
  (Pdb) break
  Num Type         Disp Enb   Where
  1   breakpoint   keep no    at buggy_traced_main.py:2

Command: enable
-----------------

- Reenables a disabled breakpoint.

::

  (Pdb) enable 1
  (Pdb) break
  Num Type         Disp Enb   Where
  1   breakpoint   keep yes    at buggy_traced_main.py:2

Command: clear
-----------------

- Removes a breakpoint entirely

::

  (Pdb) clear 1
  Deleted breakpoint 1
  (Pdb) break
  (Pdb) 

Command: tbreak
-----------------

- A breakpoint which is cleared whenever it is hit the first time.


::

  > buggy_traced_main.py(6)<module>()
  -> divide_one_by(0)
  (Pdb) tbreak 2
  Breakpoint 1 at buggy_traced_main.py:2
  (Pdb) c
  Deleted breakpoint 1
  > buggy_traced_main.py(2)divide_one_by()
  -> return 1/divisor
  (Pdb) break
  (Pdb) 

Command: break <line>,<cond>
----------------------------

- Stops execution at lineno if <condition is true>

::

  > buggy_traced_main.py(6)<module>()
  -> divide_one_by(0)
  (Pdb) break 2, divisor==0
  Breakpoint 1 at buggy_traced_main.py:2
  (Pdb) c
  > buggy_traced_main.py(2)divide_one_by()
  -> return 1/divisor
  (Pdb) 
  (Pdb) break
  Num Type         Disp Enb   Where
  1   breakpoint   keep yes   at buggy_traced_main.py:2
	stop only if divisor==0

Command: ignore
----------------

- Ignore the next N iterations through a breakpoint.

::

  > buggy_traced_main.py(6)<module>()
  -> divide_one_by(0)
  (Pdb) break 2
  Breakpoint 1 at buggy_traced_main.py:2
  (Pdb) ignore 1 1
  Will ignore next 1 crossing of breakpoint 1.
  (Pdb) c
  Traceback (most recent call last):
    File "buggy_traced_main.py", line 6, in <module>
      divide_one_by(0)
    File "buggy_traced_main.py", line 2, in divide_one_by
      return 1/divisor
  ZeroDivisionError: integer division or modulo by zero
  ...

Advanced
--------

``commands`` -- define a series of commands to be executed when a breakpoint
is hit.

``jump`` -- alter program flow at runtime without modifying code

``run`` -- restart the program from the beginning explicitly without losing
breakpoints or other settings.

``debug`` -- recursively invoke the debugger using an expression

PDB Aliases
-----------

``alias ph !help(%1)``

  ``ph <someobject>`` prints help about the object

``alias pl pp list(locals().keys())``

   ``pl`` pretty-prints a list of the names local to the frame.

``%`` args are positional starting at 1; to consume all use ``%*``.

.pdbrc File
-----------

- Can live in the current directory or in your home directory.  If both
  exist, home directory directory values are executed, then current directory
  values.

- Almost any pdb command can be executed, but .pdbrc is most useful to define
  aliases.

::

  [chrism@thinko pdb]$ cat ~/.pdbrc 
  alias ph !help(%1)
  alias pl pp list(locals().keys())

PDB, Web Apps and Threads
-------------------------

- Set server thread count to 1

Purple Bags
-----------

- Emacs ``pdbtrack``

- Enhanced shells: ipdb, pudb, winpdb, pdb++

- IDE debugger integration (Wing, Eclipse PyDev, PyCharm, etc)

- pdb-pm-like environments (Django/Flask/Pyramid debug toolbar)

