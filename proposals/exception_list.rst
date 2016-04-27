===================================================================
P0322r0 : exception_list 
===================================================================

:Project: ISO JTC1/SC22/WG21: Programming Language C++
:Number: P0322r0
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


    *During the execution of a standard parallel algorithm, if the invocation of
    an element access function exits via an uncaught exception, the behavior of
    the program is determined by the type of execution policy used to invoke the
    algorithm:*

      - *If the execution policy object is of type class
        parallel_vector_execution_policy, std::terminate shall be called.*
      - *If the execution policy object is of type sequential_execution_policy
        or parallel_execution_policy, the execution of the algorithm exits via
        an exception. The exception shall be an exception_list containing all
        uncaught exceptions thrown during the invocations of element access
        functions, or optionally the uncaught exception if there was only one.*

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
races with ``exception_list`` which are allievated by the immutable design.

.. To further our goal of picking a design free from thread-safety caveats, we
   have decided to delete the move constructor of ``exception_list``, providing
   only a copy constructor. Although it is outside of the scope of this paper,
   the authors note that ``exception``'s move constructor is not deleted, which
   we believe risks race conditions in catch blocks during multi-threaded
   execution.

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
      typedef /*** unspecified ***/ iterator;
      typedef /*** unspecified ***/ size_type;

      /////////////////////////////////////////////////////////////////////////
      // CONSTRUCTORS

      constexpr exception_list() noexcept = default; 

      exception_list(const exception_list& other);

      exception_list(exception_list&& other) noexcept;

      // "push_back" constructors
      exception_list(exception_ptr e);
      exception_list(const exception_list& other, exception_ptr e);
      exception_list(exception_list&& other, exception_ptr e);

      // iterator-pair "insert" constructors 
      template <class InputIterator>
      exception_list(InputIterator first, InputIterator last);
      template <class InputIterator>
      exception_list(const exception_list& other,
                     InputIterator first, InputIterator last);
      template <class InputIterator>
      exception_list(exception_list&& other,
                     InputIterator first, InputIterator last);

      // initializer-list "insert" constructors 
      exception_list(initializer_list<exception_ptr> list);
      exception_list(const exception_list& other,
                     initializer_list<exception_ptr> list);
      exception_list(exception_list&& other,
                     initializer_list<exception_ptr> list);

      // "splice" constructors
      exception_list(const exception_list& copy_from,
                     exception_list&& transfer_from); 
      exception_list(exception_list&& move_from,
                     exception_list&& transfer_from) noexcept; 

      /////////////////////////////////////////////////////////////////////////
      // QUERY INTERFACE 

      size_type size() const noexcept;

      iterator begin() const noexcept;
      iterator cbegin() const noexcept;

      iterator end() const noexcept;
      iterator cend() const noexcept;

      /////////////////////////////////////////////////////////////////////////

      const char* what() const noexcept override;
  };

  }

..

FIXME: Exception gurantees for the constructors that can throw.

The class ``exception_list`` owns a sequence of ``exception_ptr`` objects.

The type ``exception_list::iterator`` shall fulfill the requirements of
``ForwardIterator``.

The type ``exception_list::size_type`` shall be an unsigned integral type
large enough to represent the size of the sequence.
      
``constexpr exception_list() noexcept = default;``

  *Effect*: Construct an empty ``exception_list``.

``exception_list(const exception_list& other);``

  *Effect*: Construct a new ``exception_list`` which is a copy of ``other``. 
  as ``other``.

  *Complexity*: Linear time in the size of ``other``.

``exception_list(exception_list&& other) noexcept;``

  *Effect*: Move construct a new ``exception_list`` from ``other``. 

  *Complexity*: Constant time.

``exception_list(exception_ptr e);``

  *Effect*: Construct a new ``exception_list`` which contains a single element,
  ``e``.

  *Complexity*: Constant time.

``exception_list(const exception_list& other, exception_ptr e);``

  *Effect*: Construct a new ``exception_list`` which is a copy of ``other``,
  and append ``e`` to the end of the owned sequence.

  *Complexity*: Linear in the size of ``other`` + 1.

``exception_list(exception_list&& other, exception_ptr e);``

  *Effect*: Move construct a new ``exception_list`` from ``other``, and append
  ``e`` to the end of the owned sequence.

  *Complexity*: Linear in the size of ``other`` + 1.
  
``template<class InputIterator>``

``exception_list(InputIterator first, InputIterator last);``

  *Effect*: Construct a new ``exception_list`` which contains
  ``distance(first, last)`` elements from the range ``[first, last)``.

  *Complexity*: Linear in ``distance(first, last)``.

  *Remarks*: This constructor shall not participate in overload resolution if
  ``is_convertible_v<InputIterator::value_type, exception_ptr> == false``.

``template<class InputIterator>``

``exception_list(const exception_list& other, InputIterator first, InputIterator last);``

  *Effect*: Construct a new ``exception_list`` which is a copy of ``other``,
  and append the range ``[first, last)`` to the end of the owned sequence.

  *Complexity*: Linear in the size of ``other`` + ``distance(first, last)``.

  *Remarks*: This constructor shall not participate in overload resolution if
  ``is_convertible_v<InputIterator::value_type, exception_ptr> == false``.

``template<class InputIterator>``

``exception_list(exception_list&& other, InputIterator first, InputIterator last);``

  *Effect*: Move construct a new ``exception_list`` from ``other``, and append 
  the range ``[first, last)`` to the end of the owned sequence.

  *Complexity*: Linear in the size of ``other`` + ``distance(first, last)``.

  *Remarks*: This constructor shall not participate in overload resolution if
  ``is_convertible_v<InputIterator::value_type, exception_ptr> == false``.

``exception_list(initializer_list<exception_ptr> list);``

  *Effect*: Construct a new ``exception_list`` which contains ``list.size()``
  elements from ``list``. 

  *Complexity*: Linear in the size of ``list``.

``exception_list(const exception_list& other, initializer_list<exception_ptr> list);``

  *Effect*: Construct a new ``exception_list`` which is a copy of ``other``,
  and append ``list`` to the end of the owned sequence.

  *Complexity*: Linear in the size of ``other`` + the size of ``list``.

``exception_list(exception_list&& other, initializer_list<exception_ptr> list);``

  *Effect*: Move construct a new ``exception_list`` from ``other``, and append
  ``list`` to the end of the owned sequence.

  *Complexity*: Linear in the size of ``other`` + the size of ``list``.

``exception_list(const exception_list& copy_from, exception_list&& transfer_from);``

  *Effect*: Construct a new ``exception_list`` which is a copy of ``copy_from``,
  and transfer all elements from ``transfer_from`` to the end of the owned
  sequence. ``transfer_from`` is left in an empty state.

  *Complexity*: Linear in the size of ``copy_from``.

  *Remarks*: The behavior is undefined if ``this == &transfer_from``.

``exception_list(exception_list&& move_from, exception_list&& transfer_from) noexcept;``

  *Effect*: Move construct a new ``exception_list`` from ``copy_from``, and
  transfer all elements from ``transfer_from`` to the end of the owned
  sequence. ``transfer_from`` is left in an empty state.

  *Complexity*: Constant time. 

  *Remarks*: The behavior is undefined if ``this == &transfer_from``.

``size_type size() const noexcept;``

  *Returns*: The number of ``exception_ptr`` objects contained within the
  ``exception_list``.

  *Complexity*: Constant time.

``iterator begin() const noexcept;``

``iterator cbegin() const noexcept;``

  *Returns*: An iterator referring to the first ``exception_ptr`` object
  contained within the ``exception_list``.

``iterator end() const noexcept;``

``iterator cend() const noexcept;``

  *Returns*: An iterator that is past the end of the owned sequence.

``const char* what() const noexcept override;``

  *Returns*: An implementation-defined NTBS.

******************************************************************
Examples
******************************************************************

******************************************************************
References
******************************************************************

