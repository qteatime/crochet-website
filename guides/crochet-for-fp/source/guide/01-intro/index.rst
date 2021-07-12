An Introduction to Crochet
--------------------------

Crochet is a programming *system* for building applications safely, even
when you include untrusted, arbitrary code inside of it. The system is
made out of many different programming *languages*, and the main language
in the system is, incidentally, also called Crochet.

We will focus on both Crochet the language and Crochet the system in this
guide.


What is Crochet for?
''''''''''''''''''''

You may have heard many programming languages label themselves as
"general purpose". There is no such thing as "general purpose", however,
and Crochet does not pretend to be that. Instead, there are very specific
domains that Crochet has been designed for:

* Interactive fiction (IF, visual novels, RPGs, etc)
* Software verification (stochastic and bounded model checking)
* Cross-platform automation (including connecting online services/apps)
* Procedural generation (roguelikes, Twitter bots, etc)
* Language tools (parsers, compilers, IDEs, REPLs, etc)

While you *may* be able to use Crochet for a domain that is not included
on the above list, you will be fighting an uphill battle, trying to mold
Crochet to fit a new domain. It's possible, but it requires significant
effort, and will have no initial community support.


Before you choose Crochet
'''''''''''''''''''''''''

Even if Crochet does happen to be designed for a domain you have in mind,
it's important to remember that Crochet is still in active development.
There is no big community around it, the tools and language are not
entirely stable yet, and many of its security guarantees have not been
proven yet---even though they are built on solid theory.

This means that choosing Crochet still requires a significant investment,
and is unlikely to be a good choice for making products for the time being.


Setting up your environment
'''''''''''''''''''''''''''

To follow this guide, we'll assume that you have some environment to
execute Crochet code. Well, if you're following this guide, you *do*
have one---the guide itself. Click on the tab-bar at the bottom of
the screen to bring up the Interactive Playground, where you can
execute and debug Crochet programs.

Once you're ready to play with fancier (and more dangerous) things with
Crochet, you can also :doc:`install it on your computer </guide/a-installation/index>`.


A taste of Crochet
''''''''''''''''''

If you're still set on using Crochet (or just playing around with it),
let's start our journey!

Crochet (the language) is very influenced by ideas in functional programming,
object-oriented programming, and relational logic programming. As well as many
ideas in the programming languages and security community.

The language is divided into "Core Crochet", what is actually executed by
the Crochet virtual machine, and "Surface Crochet", what programmers are
expected to write. Because this is not really a guide on the internals,
we'll focus on Surface Crochet for the rest of the guide, and will call it
just "Crochet" from now on.

So, what does Crochet look like? Well, imagine that you wanted to know how
many seconds there are in 4 hours and 35 minutes. Sure you could use a
calculator. You could even do the computation yourself with pen-and-paper.
But if you decided to use Crochet to find the answer, you would open the
interactive playground and write a program that looks like this:

.. code-block:: crochet
   :linenos:

   open crochet.time;

   (4 hours + 35 minutes) to-seconds

Upon executing it, the interactive playground would tell you ``16,500``. That
is, there are 16,500 seconds in 4 hours and 35 minutes. But what exactly is
going on here?

In the first line, ``open crochet.time`` allows the program to refer to any
types and commands from the ``crochet.time`` package. Functions in Crochet
are called commands. And ``hours``, ``minutes``, and ``to-seconds`` are all
commands. Slightly different from most mainstream programming languages out
there, unary commands in Crochet are written in **post-fix** form. Indeed,
to highlight this fact, the *actual name* of these commands is rather
``_ hours``, ``_ minutes``, and ``_ to-seconds``. The ``_`` indicates where
arguments should go---and we'll use this notation for the rest of the
document, as we'll soon see that commands are not *always* written in
post-fix form.

For example, let's look at what ``+`` means here. Why, that's another command!
The ``_ + _`` command, to be more exact. This is a binary command that takes
one argument on each side, and does something with them. Here we're applying
this command to the result of ``4 hours`` and ``35 minutes``.

Before we go into the meaning of ``_ + _``, let's take a moment to learn what
``4 hours`` and ``35 minutes`` even are, however. If you try typing and
evaluating each of these expressions in the interactive playground it will
tell you:

.. code-block:: crochet

   4 hours
   // => (duration from crochet.time) [days -> (***), hours -> (***), ...]

   35 minutes
   // => (duration from crochet.time) [days -> (***), hours -> (***), ...]

So, each of these expressions evaluates to some "duration" value that can
be found in the ``crochet.time`` package. A duration is a kind of value that
captures some amount of time accurately, and the package provides commands for
creating, combining, and converting durations. The ``_ to-seconds`` is one such
conversion function, which takes an accurate duration and produces an
*approximation* of it in seconds. A ``_ to-minutes`` would do the same, and
produce an approximation in minutes.

But what's up with ``_ + _``? Isn't ``+`` an operation on numbers? How does it
know what to do with these dura--what's-their-face? Well, let's move on to the next
important concept in Crochet: commands, again.


Commands can do that?!
^^^^^^^^^^^^^^^^^^^^^^

In Crochet, a command is a function. But not just *any* function. No. A
command is really a *collection* of functions, and in order to evaluate one
we pick whatever from that collection best fits the arguments provided.

So if we evaluate ``1 + 3`` that will do integral addition, and output ``4``.
If we evaluate ``1.2 + 3.2`` that will do floating point addition, and output ``4.4``.
But if we evaluate ``4 days + 35 minutes`` that will do a duration combination,
and output a new duration that contains "4 days and 35 minutes" (accurately).

This choice is performed by looking at the value itself, and therefore decided
at runtime [#f1]_. You may have heard of a similar concept before called a
"Multi-method". Crochet's commands are pretty much like the multi-methods
you'd see in a language like Julia, but with some caveats that we'll get
to in due time---both due to security and interactive programming.

Anyway! The ``_ + _`` is a command, and the instance of this command that
the ``crochet.time`` package provides looks like this:

.. code-block:: crochet

   command (This is duration) + (That is duration) =
     new duration(This days + That days, This hours + That hours, ...)

It says that the command with the signature on the left
(``(This is duration) + (That is duration)``) evaluates to the thing
on the right (``new duration(...)``). The ``<Name> is <Type>`` guard
denotes where the arguments go---to evaluate it, you write an expression
in the same format, but replacing the guard with the value you want to
put there, so in ``4 hours + 35 minutes``, ``This`` gets the value of
``4 hours``, and ``That`` gets the value of ``35 minutes``.


Wait, what do you mean by "type"?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There are many different uses of ``type`` in programming. But in Crochet,
a ``type`` is a way of categorising values. This is similar to the role
that ``data`` declarations play in Haskell, or class declarations play in
Java (without the method part), or ``struct`` declarations play in C.

If we look at the way the ``duration`` type is declared in the ``crochet.time``
package, it should feel very similar to other data-type declarations that
you've seen before:

.. code-block:: crochet

   type duration(
     days is integer, 
     hours is integer, 
     minutes is integer, 
     seconds is integer, 
     milliseconds is integer,
   );

And, as seen in the previous section, one would construct a value of this
type with the ``new <type>(...)`` form.


That's pretty much all!
^^^^^^^^^^^^^^^^^^^^^^^

Of course, Crochet has several other concepts, but Types and Commands form the
very core of it, and a lot of code in the language relies almost entirely on
just these two ideas.

We'll see them shortly in this guide. So, if this introduction section
has motivated you to explore more of Crochet, let's move on to some
deeper dives into different concepts!


.. rubric:: Footnotes

.. [#f1] Although the "decided at runtime" is the simple way of looking at it,
   this isn't an *accurate* way of looking at it. The choice is done based on
   which tag the value has at runtime, but if the Crochet implementation can
   prove that it knows this information at compile time, the choice will be
   made at compile time.

