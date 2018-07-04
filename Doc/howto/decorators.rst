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

Before discussing decorators and decorator factories, let's go over the things that make them possible in Python.

The most important thing is that functions, coroutines and classes are first-class citizens in Python, which allows us to:

* Pass them as arguments::

     help(print)

* Return them as values::

     def f():
         return print

     f()('Hello World')

* Bound them to new identifiers::

     my_print = print
     my_print('Hello World')

Also, it's possible to define a function, a coroutine or a class in another definition::

   def nesting_func():
       def nested_func():
           print('Hello World')

       nested_func()
       print('This is getting old...')

   nesting_func()

Variables used by a nested function but not defined in it ("free variables") are looked up in the nesting function, recursively, until reaching the non-nested function, which looks for the variable at the module level::

   y = 'y'

   def nesting_func():
       x = 'x'

       def nested_func():
           print(x, y)

       nested_func()

   nesting_func()

The values that free variables had at the nested function definition time are saved in "closures"::

   import inspect

   y = 'y'

   def nesting_func():
       x = 'x'

       def nested_func():
           print(x, y, z)

       return nested_func

   my_func = nesting_func()
   y = '2'

   print(inspect.getclosurevars(my_func))

Closures allow nested functions to know the values of variables defined in functions whose lifetime has ended.

Now that we have all the necessary building blocks, let's move forward.

Decorators
----------

To put it simple, decorators let you alter a call to a function, coroutine or class.

A decorator can be implemented with a function or a class.  When implemented with a function, the function must require only one argument; when implemented with a class, the class :meth:`__init__` method must require only one argument.  Said argument will reference the object to be decorated.

While it's usually the case for decorators to call the original object, it's not a requirement at all, and decorators can be written to outright ignore the original object.

Decorator functions
^^^^^^^^^^^^^^^^^^^

Decorator functions usually return a new function which at some point calls the original object::

   def decorator(obj):
       def decorated_object(*args, **kwargs):
           return obj(*args, **kwargs)

       return decorated_object

   print = decorator(print)

Decorator classes
^^^^^^^^^^^^^^^^^

Decorator classes return an instance which at some point calls the original object::

   class Decorator:
       def __init__(self, obj):
           self.obj = obj

       def __call__(self, *args, **kwargs):
           return self.obj(*args, **kwargs)

   print = Decorator(print)

Preserving the original object metadata
---------------------------------------

All metadata of the original object is lost when a decorator returns a new object::

   def decorator(obj):
       def decorated_object(*args, **kwargs):
           return obj(*args, **kwargs)

       return decorated_object

   def function(a: int = 1, b: int) -> int:
       '''Returns a + b'''
       return a + b

   function = decorator(function)

   print(function.__qualname__)
   print(function.__doc__)
   print(function.__annotations__)

If the decorator acts as a wrapper instead of replacing the original object behaviour, this might be an inconvenience.

To remediate this, the standard library provides the :meth:`functools.update_wrapper` function which copies the relevant metadata from the original object to the decorated object::

   import functools

   def decorator(obj):
       def decorated_object(*args, **kwargs):
           return obj(*args, **kwargs)

       functools.update_wrapper(decorated_object, obj)

       return decorated_object

   def function(a: int, b: int) -> int:
       '''Returns a + b'''
       return a + b

   function = decorator(function)

   print(function.__qualname__)
   print(function.__doc__)
   print(function.__annotations__)

Decorator factories
-------------------

Requiring only one parameter with fixed semantics, decorators have no parametrization.

Enter decorator factories.  Decorator factories take as many arguments as needed, create a decorator, and return it.

As with decorators, decorator factories can be implemented with functions or classes.

Decorator factory functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Decorator factory functions create a decorator, and make use of closures to provide the decorator its arguments.

Example::

   import datetime

   def decorator_factory(format='%Y-%m-%d %M:%H:%S'):
      def decorator(obj):
          def decorated_object(*args, **kwargs):
              timestamp = datetime.datetime.today()
              print('{:{}} Start'.format(timestamp, format))

              return obj(*args, **kwargs)

          return decorated_object

      return decorator

   def obj():
   	   print('Test')

   obj = decorator_factory(format='%Y%m%dT%M%H%S')(obj)
   obj()

Decorator factory classes
^^^^^^^^^^^^^^^^^^^^^^^^^

In decorator factory classes the :meth:`__init__` method acts as the decorator factory, storing all arguments as class attributes, and the :meth:`__call__` method acts as the decorator.

Example::

   import datetime

   class DecoratorFactory:
       def __init__(self, format='%Y-%m-%d %M:%H:%S'):
           self.format = format

       def __call__(obj, *args, **kwargs):
           def decorated_object(*args, **kwargs):
              timestamp = datetime.datetime.today()
              print('{:{}} Start'.format(timestamp, self.format))
 
               return obj(*args, **kwargs)
 
           return decorated_object

   def obj():
   	   print('Test')
   
   obj = DecoratorFactory(format='%Y%m%dT%M%H%S')(obj)
   obj()

Decoration at definition time
-----------------------------

To improve readability, Python provides syntactic sugar for applying decorators at definition time::

   @decoration
   decorated object definition

Where ``decoration`` is the name of a decorator or a call to a decorator factory.

For example, given the NOP decorator::

   def decorator(obj):
       return obj

It can be applied at definition time as::

   @decorator
   def obj():
       pass

Multiple decorators can be applied at definition time by putting each one in a new line::

   @time
   @log
   def obj():
       pass

When multiple decorators are specified, they are applied bottom to top.

Decoration at definition time is not always possible (as when the objects to be decorated are defined in a third party module), but when it is, it is much easier to read.

Examples in the standard library
--------------------------------

The standard library provides several decorators and decorator factories that can be studied to see how they work in production code:

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
      The proposal that introduced syntax for decoration at definition time.

   :pep:`3129` - Class Decorators
      The proposal that extended :pep:`318` to allow class decoration at definition time.
