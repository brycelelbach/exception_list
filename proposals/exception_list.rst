===================================================================
D####r# : exception_list 
===================================================================

:Project: ISO JTC1/SC22/WG21: Programming Language C++
:Number: D####r#
:Date: 2016-04-24
:Reply-to: balelbach@lbl.gov
:Author: Bryce Lelbach 
:Contact: balelbach@lbl.gov
:Author: Alisdair Meredith
:Contact: alisdairm@me.com 
:Author: Jared Hoberock 
:Contact: jhoberock@nvidia.com 
:Audience: Library Evolution Working Group (LEWG)
:Audience: Study Group 1 - Concurrency (SG1) for concurrency concerns and unity with the Parallelism TS 
:URL: https://github.com/brycelelbach/exception_list/blob/master/proposals/exception_list.rst

.. sectnum::

******************************************************************
Motivation
******************************************************************

The Parallelism TS specifies ``exception_list``, a class which owns a sequence
of ``exception_ptr`` objects. ``exception_list`` is used to report exceptions
that are thrown during the execution of a standard parallel algorithm. As
specified in [parallel.exceptions.behavior] (N4507 3.1.2):

.. code-block:: none

    During the execution of a standard parallel algorithm, if the invocation of
    an element access function exits via an uncaught exception, the behavior of
    the program is determined by the type of execution policy used to invoke the
    algorithm:
      - If the execution policy object is of type class
        parallel_vector_execution_policy, std::terminate shall be called.
      - If the execution policy object is of type sequential_execution_policy
        or parallel_execution_policy, the execution of the algorithm exits via
        an exception. The exception shall be an exception_list containing all
        uncaught exceptions thrown during the invocations of element access
        functions, or optionally the uncaught exception if there was only one.

..

``exception_list`` in the Parallelism TS specifies a query interface, but has
no interface for constructing and populating the object.

At Jacksonville, there was interest in seeing the ``exception_list`` class from
the Parallelism TS be elaborated into a more general-purpose and usable type.
In particular, we want ``exception_list`` to have interfaces for construction
which make it possible for standard library users to utilize exception_list in
their own code.

******************************************************************
Design
******************************************************************

While exploring the design of ``exception_list``, our major question was should 
``exception_list`` be mutable or immutable after construction. The pros of both
options are:

* Immutable
  * Existing exception types are immutable.
  * An immutable design negates many of our concerns regarding the use of
    ``exception_list`` in a multi-threaded context.

* Mutable
  * Existing standard library containers are mutable.
  * The standard library doesn't currently have a design for immutable
    containers and we will not have sufficient time before C++17 to full explore
    this design space.
  * A simple, non-concurrent mutable ``exception_list`` has decreased space and
    time overhead when compared to an immutable ``exception_list``.

We decided upon an immutable design. The precedence for immutability in existing
exception types was the major deciding factor. We did not wish to introduce a 
new standard exception type which had substantially different semantics from
existing exception types.

Additionally, some of the authors had strong concerns about potential data
races with ``exception_list`` which are allievated by the immutable design.  To
further our goal of picking a design free from thread-safety caveats, we have
decided to delete the move constructor of ``exception_list``, providing only a
copy constructor. Although it is outside of the scope of this paper, the authors
note that ``exception``'s move constructor is not deleted, which we believe risks
race conditions in catch blocks during multi-threaded execution.

If an immutable ``exception_list`` is shipped, we do not believe it will be
possible to switch to a mutable design in the future. Such a switch would break
code (at runtime) that was written assuming that the type was immutable.

******************************************************************
Specification
******************************************************************

.. code-block:: c++

  namespace std {

  class exception_list : public exception
  {
    public:
      typedef /* unspecified */ iterator;
      typedef /* unspecified */ size_type;

      size_type size() const noexcept;

      iterator begin() const noexcept;
      iterator end() const noexcept;

      char const* what() const noexcept override;
  };

  }

..

1.) The class ``exception_list`` owns a sequence of ``exception_ptr`` objects.

2.) The type ``exception_list::iterator`` shall fulfill the requirements of
``ForwardIterator``.

3.) ``size_type size() const noexcept;``

  4.) *Returns*: The number of ``exception_ptr`` objects contained within the
  ``exception_list``.

  5.) *Complexity*: Constant time.

6.) ``iterator begin() const noexcept;``

  7.) *Returns*: An iterator referring to the first ``exception_ptr`` object
  contained within the ``exception_list``.

8.) ``iterator end() const noexcept;``

  9.) *Returns*: An iterator that is past the end of the owned sequence.

10.) ``char const* what() const noexcept override;``

  11.) *Returns*: An implementation-defined NTBS.

******************************************************************
References
******************************************************************

