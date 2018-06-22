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

Free variables values had at the nested function definition time is stored in what is known as "closures".

Decorators
----------

A decorator is a one-parameter function that takes a function, coroutine, or class.

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

   def log_start(obj):
       def decorated_object():
           print('Start')
           obj()

       return decorated_object

   def log_end(obj):
       def decorated_object():
           obj()
           print('End')

       return decorated_object

   def log_start_and_end(obj):
       def decorated_object():
           print('Start')
           obj()
           print('End')

       return decorated_object

At this point it should be clear how the DRY principle is being violated, but let's go one step further: what if we wanted the time format of the logging to be configurable? We can't achieve that with decorators.

Enter decorator factories.  Decorator factories take arguments, create a decorator, and return it::

   def decorator_factory(log_start, log_end):
      def decorator(obj):
          def decorated_object():
              if log_start:
                  print('Start')

              obj()

              if log_end:
                  print('End')

          return decorated_object

      return decorator
   
   obj = decorator_factory(log_start=True, log_end=True)(obj)

Note that decorator factories are not decorators themselves.

Decoration at definition time
-----------------------------

Python provides syntactic sugar for applying decorators at definition time.  What follows @ must be an expression that evaluates to a function requiring only one argument.  This is important to highlight: what comes after @ is not a decorator, but an expression that evalutes to a decorator.

For example, given the decorator::

   def decorator(obj):
       def decorated_object():
           obj()

       return decorated_object

This::

   def obj():
       pass

   obj = decorator(obj)

Can be applied at definition time written as::

   @decorator
   def obj():
       pass

Multiple decorators can be applied at definition time by putting each decorator in a new line::

   @time
   @log
   def obj():
       pass

Decorator factories can also be applied at definition time::

   @decorator_factory(log_start=True, log_end=True)
   def obj():
      print('Test')
   
   obj()

Decorating at definition time is not always possible (as when the definitions are made by a third party module), but when it is, it is much easier to read.

Examples in the standard library
--------------------------------

The standard library provides several decorators that can be read to see how decorators work in real life:

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
