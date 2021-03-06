### v1

In this version a flavors instance is modelled with a map (it could
just as well be a tuple). This means that a flavor instance is not a
global object but behaves like a "normal" LFE term. If one is updated
then a new one is created and the old one is left unchanged.

This model has 2 modules for each flavor:

- *flav*-flavor-core which contains functions describing all the
  properties of the flavor *flav*. This module is built at compile
  time.

- *flav*-flavor which contains the access functions for which are
  called when sending messages to an instance of this flavor. It is
  built when the first instance of this flavor is made using
  ``make-instance`` or ``flavors:instantiate-flavor``.

The reason for having 2 modules per flavor is that access function
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
- ``required-methods``
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

In both the primary and daemon methods the instance map is
automatically bound to the variable ``self``.  A primary method
**MUST** return the tuple ``#(<return-value> <updated-instance-map>)``
and a daemon method **MUST** return only the updated instance map. See
the example flavors ``f1`` and ``f2``.

When defining a flavor the component sequence is built as it should be
for the ``before`` and ``after`` daemons and there is a very (very)
rudimentary ``vanilla-flavor``.

You can only define **ONE** flavor in an LFE file and no other LFE
module. Compiling the flavor definition file results in the
*flav*-flavor-core.beam file being generated. Local functions used by
the flavor methods can be defined in the same file but the **MUST**
come after the defflavor definition.

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
#M(*flavor-module* f1-flavor a undefined g undefined m 42 q undefined
   share f1 time #(1439 913945 374362) x 42 y undefined z undefined)
> (send f1 'set-y 99)
f2 before set-y #M(*flavor-module* f1-flavor a undefined g undefined
                   m 42 q undefined share f1
                   time #(1439 913945 374362) x 42 y undefined
                   z undefined)
f2 after set-y #M(*flavor-module* f1-flavor a undefined g undefined
                  m 42 q undefined share f1
                  time #(1439 913945 374362) x 42 y 99 z undefined)
f1 after set-y #M(*flavor-module* f1-flavor a undefined g undefined
                  m 42 q undefined share f1
                  time #(1439 913945 374362) x 42 y 99 z undefined)
#(99
  #M(*flavor-module* f1-flavor a undefined g undefined m 42
     q undefined share f1 time #(1439 913945 374362) x 42 y 99
     z undefined))
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