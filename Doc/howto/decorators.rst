================
Decorators HOWTO
================

:Author: Andrés Delfino
:Contact: <adelfino at gmail dot com>

.. Contents::

Abstract
--------

This HOWTO explains how to write decorators and decorator factories.

Background
----------

Before discussing decorators, let's go over the things that make decorators possible in Python.

The most important thing is that functions, coroutines and classes are first-class citizens in Python, which allows us to:

* Pass them as arguments::

     print(print)

* Return them as values::

     def f():
         return print

     f()('Hello World')

* Bound them to new identifiers::

     my_print = print
     my_print('Hello World')

Also, functions, coroutines and classes can be defined inside a function::

   def nesting_func():
       def nested_func():
           print('Hello World')

       nested_func()
       print('This is getting old...')

   nesting_func()

Variables accesed by a nested function but not defined in it ("free variables") are looked up in the nesting function, recursively, until reaching the non-nested function, which looks for the variable in the global namespace::

   y = 'y'

   def nesting_func():
       x = 'x'

       def nested_func():
           print(x, y)

       nested_func()

   nesting_func()

The values that free variables had at the nested function definition time are stored in what is known as "closures"::

   import inspect

   y = '1'

   def nesting_func():
       x = 'a'

       def nested_func():
           print(x, y, z)

       return nested_func

   my_func = nesting_func()
   y = '2'

   print(inspect.getclosurevars(my_func))

Decorators
----------

Decorators let you add logic to a function, coroutine or class.

A decorator is a function that requires only one parameter, or a class whose constructor requires only one parameter. Said parameter is the function, coroutine, or class to be decorated.

Function decorator::

   def decorator(obj):
       def decorated_object(*arg, **kwargs):
           return obj(*arg, **kwargs)

       return decorated_object

   def f():
       pass

   f = decorator(f)

Class decorator::

   def decorator(cls):
       def __repr__(self):
           return 'Hola'

       cls.__repr__ = __repr__

       return cls

   class C:
       pass

   C = decorator(C)

Decorators can be applied in nested fashion::

   obj = time(log(obj))

Decorator factories
-------------------

Having only one parameter with fixed semantics, decorators have no parametrization.

One could think that the solution is to have a decorator for each case::

   import datetime
   
   def helper(obj, log_start, log_end, *args, **kwargs):
       format = '%Y-%m-%d %M:%H:%S'

       if log_start:
           timestamp = datetime.datetime.today()
           print('{:{}} Start'.format(timestamp, format))
   
       r = obj(*args, **kwargs)
   
       if log_end:
           timestamp = datetime.datetime.today()
           print('{:{}} End'.format(timestamp, format))
   	
       return r
   
   def log_start(obj):
       def decorated_object(*args, **kwargs):
           return helper(obj, log_start=True, log_end=False, *args, **kwargs)
   
       return decorated_object
   
   def log_end(obj):
       def decorated_object(*args, **kwargs):
           return helper(obj, log_start=False, log_end=True, *args, **kwargs)
   
       return decorated_object
   
   def log_start_and_end(obj):
       def decorated_object(*args, **kwargs):
           return helper(obj, log_start=True, log_end=True, *args, **kwargs)
   
       return decorated_object
       
   @log_start
   def sayhi():
       print('Hi')
       
   sayhi()

At this point the code has already gotten very complex, but let's go one step further: what if the timestamp format must be configurable? We can't achieve that with decorators alone without recurring to global variables.

Enter decorator factories.  Decorator factories take arguments, create a decorator, and return it::

   import datetime

   def decorator_factory(log_start, log_end, format='%Y-%m-%d %M:%H:%S'):
      def decorator(obj):
          def decorated_object(*args, **kwargs):
              if log_start:
                  timestamp = datetime.datetime.today()
                  print('{:{}} Start'.format(timestamp, format))

              r = obj(*args, **kwargs)

              if log_end:
                  timestamp = datetime.datetime.today()
                  print('{:{}} End'.format(timestamp, format))

              return r

          return decorated_object

      return decorator
   
   obj = decorator_factory(log_start=True, log_end=True, format='%Y%m%dT%M%H%S')(obj)

Note that decorator factories are not decorators themselves: they create the right decorators for the right scenarios.

Decoration at definition time
-----------------------------

To improve readability, Python provides syntactic sugar for applying decorators at definition time::

   @decorator_expression
   decorated object definition

What follows ``@`` must be an expression that evaluates to a function requiring only one argument.  This is important to highlight: what comes after ``@`` is not necessarily a decorator, but an expression that evalutes to one.

For example, given the decorator::

   def decorator(obj):
       def decorated_object():
           obj()

       return decorated_object

It can be applied at definition time as::

   @decorator
   def obj():
       pass

Multiple decorators can be applied at definition time by putting each one in a new line::

   @time
   @log
   def obj():
       pass

Decorator factories can also be applied at definition time::

   @log(start=True, end=True)
   def obj():
      print('Test')
   
   obj()

Decoration at definition time is not always possible (as when definitions are made by a third party module), but when it is possible, decoration at definition time is much easier to read.

Examples in the standard library
--------------------------------

The standard library provides several decorators and decorator factories that can be studied to see how they work in real life:

=================================   ==========================================
:meth:`contextlib.contextmanager`   function decorator
:meth:`functools.total_ordering`    class decorator
:meth:`unittest.skip`               function decorator factory
:meth:`dataclasses.dataclass`       class decorator factory or class decorator
=================================   ==========================================

See also
--------

.. seealso::

   :pep:`318` - Decorators for Functions and Methods
      A

   :pep:`3129` - Class Decorators
      A
