================
Decorators HOWTO
================

:Author: Andrés Delfino
:Contact: <adelfino at gmail dot com>

.. Contents::

Abstract
--------

This HOWTO provides

Definition
----------

A decorator is a function that takes a function, method, coroutine, or class and
returns it wrapped in a new object with some logic.

Decorators has a myriad of uses.

Some common built-in decorators are classmethod, staticmethod and property.

And the standard library provides several more:

@contextlib.contextmanager
@functools.singledispatch


Background
----------

Before talking about decorators, it's important to understand a few things that
make working with decorators possible.

Functions are first class citizens in Python: identifiers can point
to them, they can be passed as arguments, and they can also be returned by
functions::

   def return_obj(obj):
       return obj

   newprint = return_obj(print)

   print('Hi')
   newprint('Hi')

Each function has its own namespace, and identifiers are looked in the
namespace of the function they are defined.

What's more, functions can be nested::

   def foo(a, b):
       def sum_numbers(a, b):
           return a + b
       return sum_numbers(a, b)

   foo(10, 20)



Decorator expressions
---------------------

Python provides some syntactic sugar for working with decorators.

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

What follows @ must be an expression that evaluates to a callable requiring
only one argument.

Decorator nesting
-----------------

Decorators can be applied in nested fashion::

   obj = time(log(obj))

Decorator expressions support such nesting by putting each decorator in its
own line::

   @time
   @log
   def obj():
       pass

Decorator factories
-------------------

While an author can write a decorator requiring more arguments than just the
object to be decorated to add some logic to the decorator, as in::

   def decorator(obj, log_start, log_end):
      def decorated_object():
          if log_start:
              print('Start')
          obj()
          if log_end:
              print('End')
      return decorated_object
   
   def obj():
      print('Test')
   
   obj = decorator(obj, log_start=True, log_end=True)
   
   obj()

A function with such signature is not supported as a decorator expression, as
it doesn't evaluate to a callable that only requires one argument.

Enter decorator factories.  Decorator factories takes the arguments used to
configure a decorator and returns a decorator::

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
   
   @decorator_factory(log_start=True, log_end=True)
   def obj():
      print('Test')
   
   obj()

Note that decorator factories are not decorators themselves as they don't take
an object and return it with added logic: they return a decorator instead.

See also
--------

.. seealso::

   :pep:`318` - Decorators for Functions and Methods
      A

   :pep:`3129` - Class Decorators
      A
