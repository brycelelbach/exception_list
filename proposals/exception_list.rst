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

******************************************************************
Design
******************************************************************

******************************************************************
Concerns
******************************************************************

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

