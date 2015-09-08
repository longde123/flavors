# flavors

<img src="resources/images/flavors-logo.png" />

LFE Flavors package

Implements the Lisp Machine flavors system in/and for LFE.

This is still very experimental and it includes different versions to
test different implementation methods.

NOTE: in these descriptions we will not describe the Lisp Machine
flavors. Check here for the [Lisp Machine
manual](http://bitsavers.trailing-edge.com/pdf/mit/cadr/chinual_6thEd_Jan84/),
our stuff is in chapter 21.

In this version a flavors instance is modelled with a process which is
more LFEy. The instance variables are kept internally in a map. This
means that a flavor instance more like a global object.

Note that if you try to send a message to yourself then the instance
process will hang, you after all just sending a synchronous message to
yourself. This could be fixed for the special case of sending messages
to yourself but the general problem that sends are synchronous
messages communication remains.

This model has 2 modules for each flavor:

- *flav*-flavor-core which contains functions describing all the
  properties of the flavor *flav*. This module is built at compile
  time.

- *flav*-flavor which contains the access functions for which are
  called when sending messages to an instance of this flavor. It is
  built when the first instance of this flavor is made using
  ``make-instance`` or ``flavors:instantiate-flavor``.

The reason for having 2 modules per flavor is that the access function
module will only be built for flavors which actually have instances of
them, so they won't be built for mixins. It also makes it easier to
modify flavors that are being used.

NOTE: no extra .lfe files are actually created. A
*flav*-flavor-core.beam is created directly from the original file
containing the flavor definition but the *flav*-flavor module is
compiled and directly loaded into the system without being written
into a file. This means that all the flavor-core modules are generated
at compile time and put in the ebin directory when the application is
built.

To access the macros do ``(include-file "include/flavors.lfe")`` or
``(include-lib "flavors/include/flavors.lfe")`` if you have the
flavors application in your search path.

In this simple test the map representing the instance is directly
visible and operations on it are explicitly done. There is no hiding
of the actual implementation. The following macros are available for
defining flavors:

```lisp
(defflavor <flavor-name> (<var1> <var2> ...) (<flav1> <flav2> ...) <opt1> <opt2> ...)
(defmethod (<flavor-name> <operation>) <lambda-list> <form1> <form2>)
(defmethod (<flavor-name> <method-type> <operation>) <lambda-list> <form1> <form2>)
(endflavor <flavor-name>)               ;Must be last after the methods
```

Currently we support the options:

- ``gettable-instance-variables``
- ``settable-instance-variables``
- ``inittable-instance-variables``
- ``required-instance-variables``
- ``required-methods`` (methods given as ``(name arity)``)
- ``required-flavors``
- ``no-vanilla-flavor``
- ``abstract-flavor``

and the standard method types ``before`` and ``after`` for the
daemons.

For using the flavor definitions there is:

```lisp
(make-instance <flavor-name> <opt1> <value1> <opt2> <value2> ... )
(flavors:instantiate-flavor <flavor-name> <init-plist>)

(send <object> <operation> <arg1> ...)
```

The variable ``self`` is automatically bound to the actual instance so
it can be passed around. A primary method must now only return the
actual return value which will be returned from sending the
method. The return value of a ``before`` and ``after`` daemon is
ignored. See the example flavors ``f1`` and ``f2``.

To access the instance variables there are two predefined function
``get/1`` and ``set/2``. Calling ``(get 'foo)`` will return the value
of the variable ``foo`` while ``(set 'bar 42)`` sets the value of the
variable ``bar`` to ``42``.

When defining a flavor the component sequence is built as it should be
for the ``before`` and ``after`` daemons and there is a very (very)
rudimentary ``vanilla-flavor``.

You can only define **ONE** flavor in an LFE file and no other LFE
module. Compiling the flavor definition file results in the
*flav*-flavor-core.beam file being generated. Local functions used by
the flavor methods can be defined in the same file but the **MUST**
come after the ``defflavor`` definition and before the ``endflavor``.

From the LFE repl do ``(c "f1" '(to_exp return))`` to see the
resultant code generated by the ``defflavor``, ``defmethod`` and
``endflavor`` macros. 

In the ``test`` directory there are two example flavors, ``f1`` and
``f2``, where ``f2`` is a component of ``f1``. Here is an example of
compiling and using them:

```lisp
> (c "test/f1")
#(module f1-flavor-core)
> (c "test/f2")
#(module f2-flavor-core)
> (run "include/flavors.lfe")
()
> (set f1 (make-instance 'f1 'x 42))
#(*flavor-instance* f1 f1-flavor <0.50.0>)
> (send f1 'set-y 99)
f2 before set-y #(*flavor-instance* f1 f1-flavor <0.50.0>)
f2 after set-y #(*flavor-instance* f1 f1-flavor <0.50.0>)
f1 after set-y #(*flavor-instance* f1 f1-flavor <0.50.0>)
99
> (send f1 'y)   
99
```

For more examples try compiling the flavors ``foo``, ``foo-base``,
``foo-mixin`` and ``bar-mixin`` and the generating an instance of the
flavor ``foo``.

Yes, this is still an experiment. The *-flavor-core* and *-flavor*
modules code can be printed out even though the LFE files are never
written.

**NOTE**: Using the flavors now requires the latest version of the
compiler which can handle the module not being the same as the file
name. However the .beam still has the same name as the module as it
must.

### v1

This is the original version in which the flavor instance is just a
map which behaves just like any normal map or other data
structure. This means that an instance is not a global object and when
one is updated then a new one is created and the old one is there
unchanged.

This means that this flavors version is like super records/elixir
structs on steroids and is probably not what most people would expect.
