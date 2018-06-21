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

The most important thing is that functions, coroutines and classes are first class citizens, which allows us to:

* Pass them as arguments::

     help(print)

* Return them as values::

     def f():
         return print

     f()('Hello World')

* Bound them to identifiers::

     my_print = print
     my_print('Hello World')

Another import thing is that functions can be nested::

   def f1():
       def f2():
           print('Hello World')

       f2()
       print('This is getting old...')

   f1()

Variables accesed by a nested function but not defined in it are looked up in the nesting function, recursively, until reaching the non-nested function, which looks for the identifier in the global namespace::

   y = 'y'

   def f1():
       x = 'x'

       def f2():
           print(x, y)

       f2()

   f1()

Nested functions code is preserved with the values that free variables had at definition time in what is known as "closures".

Decorators
----------

A decorator is a one-parameter function that takes a function, coroutine, or class.

Usually, decorators that take a function/coroutine return a new function/coroutine, and decorators that take a class return the same object.

Decorator::

   def decorator(obj):
       def decorated_object():
           obj()

       return decorated_object

   def f():
       pass

   f = decorator(f)

Decorators can be applied in nested fashion::

   obj = time(log(obj))

Decoration at definition time
-----------------------------

Python provides syntactic sugar for applying decorators at definition time.  What follows @ must be an expression that evaluates to a callable requiring only one argument.  This is import to highlight: what cames after @ is not a decorator, but an expression to evalutes to a decorator.

For example, given the decorator::

   def decorator(obj):
       def decorated_object():
           obj()

       return decorated_object

This::

   def obj():
       pass

   obj = decorator(obj)

Can be written as::

   @decorator
   def obj():
       pass

What's more, multiple decorators can also be applied at definition time by simply putting each decorator in a new line::

   @time
   @log
   def obj():
       pass

Decorator factories
-------------------

Having only one parameter, with fixed semantics, decorators do not allow parametrization.

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

It's clear how the DRY principle is being violated here.

Fortunately, this isn't needed at all.  Enter decorator factories.  Decorator factories take the arguments needed to create the right decorator and return it.

Given the decorator factory::

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

It can be applied::
   
   obj = decorator_factory(log_start=True, log_end=True)(obj)

Or by using the @ syntax::

   @decorator_factory(log_start=True, log_end=True)
   def obj():
      print('Test')
   
   obj()

Note that decorator factories are not decorators themselves.

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
