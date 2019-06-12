# D1673R0: A free function linear algebra interface based on the BLAS

## Authors

* Peter Caday (peter.caday@intel.com) (Intel)
* Mark Hoemmen (mhoemme@sandia.gov) (Sandia National Laboratories)
* David Hollman (dshollm@sandia.gov) (Sandia National Laboratories)
* Nevin Liber (nliber@anl.gov) (Argonne National Laboratory)
* Li-Ta Lo (ollie@lanl.gov) (Los Alamos National Laboratories)
* Graham Lopez (lopezmg@ornl.gov) (Oak Ridge National Laboratories)
* Piotr Luszczek (luszczek@icl.utk.edu) (University of Tennessee)
* Sarah Knepper (sarah.knepper@intel.com) (Intel)
* Christian Trott (crtrott@sandia.gov) (Sandia National Laboratories)

## Contributors

* Timothy Costa (tcosta@nvidia.com) (NVIDIA)
* Chip Freitag (chip.freitag@amd.com) (AMD)
* Bryce Lelbach (blelbach@nvidia.com) (NVIDIA)
* Siva Rajamanickam (srajama@sandia.gov) (Sandia National Laboratories)
* Srinath Vadlamani (Srinath.Vadlamani@arm.com) (ARM)
* Rene Vanoostrum (Rene.Vanoostrum@amd.com) (AMD)

## Purpose of this paper

This paper proposes a C++ Standard Library dense linear algebra
interface based on the dense Basic Linear Algebra Subroutines (BLAS).
This corresponds to most of Chapter 2 and some of Chapter 4 in the
[BLAS
Standard](http://www.netlib.org/blas/blast-forum/blas-report.pdf).
The BLAS supports the following classes of operations on matrices and
vectors:

* Elementwise vector sums
* Multiplying all elements of a vector or matrix by a scalar
* 2-norms, 1-norms, and infinity-norms of vectors
* Vector-vector, matrix-vector, and matrix-matrix products
  (contractions)
* Low-rank updates of a matrix
* Triangular solves with one or more "right-hand side" vectors
* Generating and applying plane (Givens) rotations

The BLAS works with the following matrix storage formats:

* "General" dense matrices, in column-major or row-major format
* Dense banded matrices, stored either as general dense matrices,
  or in a "packed" format
* Symmetric or Hermitian (for complex numbers only) dense matrices,
  stored either as general dense matrices, or in a packed format
* Dense triangular matrices, stored either as general dense matrices
  or in a packed format
* Dense banded triangular matrices, stored in a packed format

We initially propose to implement all these features.  Our proposal
also has the following distinctive characteristics:

* It uses free functions, not arithmetic operator overloading.

* It uses the multidimensional array data structures
  [`mdspan`](wg21.link/p0009) and `mdarray` (D1684R0) to represent
  matrices and vectors.  In the future, it could support other
  proposals' matrix and vector data structures.

* The interface permits optimizations for matrices and vectors with
  small compile-time dimensions; the standard BLAS interface does not.

* Each of our proposed operations supports all element types for which
  that operation makes sense, unlike the BLAS, which only supports
  four element types.

* Our operations permit "mixed-precision" computation with matrices
  and vectors that have different element types.  This subsumes the
  functionality of the Mixed-Precision BLAS specification (see
  [Chapter 4 of the BLAS
  Standard](http://www.netlib.org/blas/blast-forum/blas-report.pdf)).

* Like the C++ Standard Library's algorithms, our operations take an
  optional execution policy argument.  This is a hook to support
  parallel execution and hierarchical parallelism.  Our proposal also
  constrains the underlying implementation to prevent conflicts over
  hardware resources (an issue in historical thread-parallel BLAS
  implementations).

* Unlike the BLAS, our proposal has the option to support "batched"
  operations (see [P1417](wg21.link/p1417)) with almost no interface
  differences.  This will support machine learning and other
  applications that need to do many small matrix or vector operations
  at once.

## Interoperable with other linear algebra proposals

We do not intend to compete with [P1385](wg21.link/p1385), a proposal
for a linear algebra library presented at the 2019 Kona WG21 meeting
that introduces matrix and vector classes and overloaded arithmetic
operators.  In fact, we think that our proposal would make a natural
foundation for a library like what P1385 proposes.  We have had
discussions with the P1385 authors, and they agree on points such as
the use of `mdspan` for storing matrices and vectors.  They also agree
that it would be natural to build P1385 on a free-function interface.

## Why include dense linear algebra in the C++ Standard Library?

1. C++ applications in "important application areas" (see
   [P0939](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0939r0.pdf))
   have depended on linear algebra for a long time.
2. Linear algebra is like `sort`: obvious algorithms are slow, and the
   fastest implementations call for hardware-specific tuning.
3. Dense linear algebra is core functionality of most linear algebra
   operations.

Linear algebra has had wide use in C++ applications for nearly three
decades (see e.g., [P1417](wg21.link/p1417), and talks at the 2019
Kona WG21 meeting).  For much of that time, many third-party C++
libraries for linear algebra have been available.  Many different
subject areas depend on linear algebra, including machine learning,
data mining, web search, statistics, computer graphics, medical
imaging, geolocation and mapping, and physics-based simulations.

["Directions for ISO C++"
(P0939)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0939r0.pdf)
offers the following in support of adding linear algebra to the C++
Standard Library:

* P0939 calls out "Support for demanding applications in important
  application areas, such as medical, finance, automotive, and games
  (e.g., key libraries...)" as an area of general concern that "we
  should not ignore."  All of these areas depend on linear algebra.

* "Is my proposal essential for some important application domain?"
  Large and small private companies, national laboratories, and
  academics all depend on linear algebra for multiple application
  domains.

* "We need better support for modern hardware": Modern hardware spends
  many of its cycles in linear algebra.  For decades, hardware
  vendors, some represented at WG21 meetings, have provided and
  continue to provide features specifically to accelerate linear
  algebra operations.  For example, SIMD (single instruction multiple
  data) is a feature added to processors to speed up matrix and vector
  operations.  [P0214](wg21.link/p0214), a C++ SIMD library, was voted
  into the C++20 draft.

Obvious algorithms for some linear algebra operations like dense
matrix-matrix multiply are asymptotically slower than less-obvious
algorithms.  (Please refer to a survey one of us coauthored,
["Communication lower bounds and optimal algorithms for numerical
linear algebra."](https://doi.org/10.1017/S0962492914000038))
Furthermore, writing the fastest dense matrix-matrix multiply depends
on details of a specific computer architecture.  This makes such
operations comparable to `sort` in the C++ Standard Library: worth
standardizing, so that Standard Library implementers can get them
right and hardware vendors can optimize them.

Dense linear algebra is the core component of most algorithms and
applications that use linear algebra, and the component that is most
widely shared over different application areas.  For example, tensor
computations end up spending most of their time in optimized dense
linear algebra functions.  Sparse matrix computations get best
performance when they spend as much time as possible in dense linear
algebra.

## Related papers

* "Historical lessons for C++ linear algebra library standardization"
  [(P1417)](wg21.link/p1417)
* "Evolving a Standard C++ Linear Algebra Library from the BLAS"
  (D1674R0)
* `mdspan` [(P0009)](wg21.link/p0009)
* `mdarray` (D1684R0)

## Notation and conventions

### The BLAS uses Fortran terms

The BLAS' "native" language is Fortran.  It has a C binding as well,
but the BLAS Standard and documentation use Fortran terms.  This paper
will call out relevant Fortran terms and highlight possibly confusing
differences with corresponding C++ ideas.

### We call "subroutines" functions

Like Fortran, the BLAS distinguishes between functions that return a
value, and subroutines that do not return a value.  In what follows,
we will refer to both as "BLAS functions" or "functions."

### Element types and BLAS function name prefix

The BLAS implements functionality for four different matrix, vector,
or scalar element types:

* `REAL` (`float` in C++ terms)
* `DOUBLE PRECISION` (`double` in C++ terms)
* `COMPLEX` (`complex<float>` in C++ terms)
* `DOUBLE COMPLEX` (`complex<double>` in C++ terms)

The BLAS' Fortran 77 binding uses a function name prefix to
distinguish functions based on element type:

* `S` for `REAL` ("single")
* `D` for `DOUBLE PRECISION`
* `C` for `COMPLEX`
* `Z` for `DOUBLE COMPLEX`

For example, the four BLAS functions `SAXPY`, `DAXPY`, `CAXPY`, and
`ZAXPY` all perform the vector update `Y = Y + ALPHA*X` for vectors
`X` and `Y` and scalar `ALPHA`, but for different vector and scalar
element types.

The convention is to refer to all of these functions together as
`xAXPY`.  In general, a lower-case `x` is a placeholder for all data
type prefixes that the BLAS provides.  For most functions, the `x` is
a prefix, but for a few functions like `IxAMAX`, the data type
"prefix" is not the first letter of the function name.  (`IxAMAX` is a
Fortran function that returns `INTEGER`, and therefore follows the old
Fortran implicit naming rule that integers start with `I`, `J`, etc.)

Not all BLAS functions exist for all four data types.  These come in
three categories:

1. The BLAS provides only real-arithmetic (`S` and `D`) versions of
   the function, since it only makes mathematical sense in real
   arithmetic.

2. The complex-arithmetic versions perform a slightly different
   mathematical operation than the real-arithmetic versions, so
   they have a different base name.

3. The complex-arithmetic versions offer a choice between
   non-conjugated or conjugated operations.

As an example of the second category, the BLAS functions `SASUM` and
`DASUM` compute the sums of absolute values of a vector's elements.
Their complex counterparts `CSASUM` and `DZASUM` compute the sums of
absolute values of real and imaginary components of a vector `v`, that
is, the sum of `abs(real(v(i))) + abs(imag(v(i)))` for all `i` in the
domain of `v`.  The latter operation is still useful as a vector norm,
but it requires fewer arithmetic operations.

Examples of the third category include the following:

* non-conjugated dot product `xDOTU` and conjugated dot product
  `xDOTC`;
* rank-1 symmetric (`xGERU`) vs. Hermitian (`xGERC`) matrix update

The conjugate transpose and the (non-conjugated) transpose are the
same operation in real arithmetic (if one considers real arithmetic
embedded in complex arithmetic), but differ in complex arithmetic.
Different applications have different reasons to want either.  The C++
Standard includes complex numbers, so a Standard linear algebra
library needs to respect the mathematical structures that go along
with complex numbers.

## What we exclude from the design

### Functions not in the Reference BLAS

The BLAS Standard includes functionality that appears neither in the
[Reference
BLAS](http://www.netlib.org/lapack/explore-html/d1/df9/group__blas.html)
library, nor in the classic BLAS "level" 1, 2, and 3 papers.  (For
history of the BLAS "levels" and a bibliography, see
[P1417](wg21.link/p1417).)  For example, the BLAS Standard has

* several new dense functions, like a fused vector update and dot
  product;
* sparse linear algebra functions, like sparse matrix-vector multiply
  and an interface for constructing sparse matrices; and
* extended- and mixed-precision dense functions (however, see section
  below).

Our proposal only includes core Reference BLAS functionality, for the
following reasons:

1. Vendors who implement a new component of the C++ Standard Library
   will want to see and test against an existing reference
   implementation.

2. Sparse linear algebra is not core functionality.  Many applications
   that use sparse linear algebra also use dense, but not vice versa.

3. The Sparse BLAS interface is a stateful interface that is not
   consistent with the rest of the BLAS, and would need more extensive
   redesign to translate into a modern C++ idiom.  See discussion in
   [P1417](wg21.link/p1417).

4. Our proposal subsumes some dense mixed-precision functionality (see
   below).

### LAPACK or related functionality

The [LAPACK](http://www.netlib.org/lapack/) Fortran library implements
solvers for the following classes of mathematical problems:

* Linear systems
* Linear least-squares problems
* Eigenvalue and singular value problems

It also provides matrix factorizations and related linear algebra
operations.  LAPACK deliberately relies on the BLAS for good
performance; in fact, LAPACK and the BLAS were designed together.  See
history presented in [P1417](wg21.link/p1417).

Several C++ libraries provide slices of LAPACK functionality.  Here is
a brief, noninclusive list, in alphabetical order, of some libraries
actively being maintained:

* [Armadillo](http://arma.sourceforge.net/)
* [Boost.uBLAS](https://github.com/boostorg/ublas)
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page)
* [Matrix Template Library](http://www.simunova.com/de/mtl4/)
* [Trilinos](https://github.com/trilinos/Trilinos/)

[P1417](wg21.link/p1417) gives some history of C++ linear algebra
libraries.  The authors of this proposal have
[written](https://github.com/kokkos/kokkos-kernels) and
[maintained](https://github.com/trilinos/Trilinos/tree/master/packages/teuchos/numerics/src)
LAPACK wrappers in C++.  One of the authors was a student of [an
LAPACK founder](https://people.eecs.berkeley.edu/~demmel/).
Nevertheless, we have excluded LAPACK-like functionality from this
proposal, for the following reasons:

1. LAPACK is a Fortran library, unlike the BLAS, which is a
   multilanguage standard.

2. We intend to support more general element types, like short floats,
   fixed-point types, and integers.  It's much more straightforward to
   make a C++ BLAS work for general element types, than to make LAPACK
   algorithms work generically.

First, unlike the BLAS, LAPACK is a Fortran library, not a standard.
LAPACK was developed concurrently with the "level 3" BLAS functions,
and the two projects share contributors.  Nevertheless, only the BLAS
and not LAPACK got standardized.  Some vendors supply LAPACK
implementations with some optimized functions, but most
implementations likely depend heavily on "reference" LAPACK.  There
have been a few efforts by LAPACK contributors to develop C++ LAPACK
bindings, from [Lapack++](https://math.nist.gov/lapack++/) in
pre-templates C++ circa 1993, to the recent ["C++ API for BLAS and
LAPACK"](https://www.icl.utk.edu/files/publications/2017/icl-utk-1031-2017.pdf).
(The latter shares coauthors and contributors with this paper.)
However, these are still just C++ bindings to a Fortran library.  This
means that if vendors had to supply C++ functionality equivalent to
LAPACK, they would either need to start with a Fortran compiler, or
would need to invest a lot of effort in a C++ reimplementation.
Mechanical translation from Fortran to C++ introduces risk, because
many LAPACK functions depend critically on details of floating-point
arithmetic behavior.

Second, we intend to permit use of matrix or vector element types
other than just the four types that the BLAS and LAPACK support.  This
is easier to do for BLAS-like operations than for the much more
complicated numerical algorithms in LAPACK.  LAPACK strives for a
"generic" design (see Jack Dongarra interview summary in
[P1417](wg21.link/p1417)), but only supports two real floating-point
types and two complex floating-point types.  Directly translating
LAPACK source code into a "generic" version could lead to pitfalls.
Many LAPACK algorithms only make sense for number systems that aim to
approximate real numbers (or their complex extentions).  Some LAPACK
functions output error bounds that rely on properties of
floating-point arithmetic.

For these reasons, we have left LAPACK-like functionality for future
work.  It would be natural for a future LAPACK-like C++ library to
build on our proposal.

### Extended-precision BLAS

Our interface does subsume the functionality of the Mixed-Precision
BLAS specification (see Ch. 4 of the BLAS Standard).  For example,
users may multiply two 16-bit floating-point matrices (assuming that a
16-bit floating-point type exists) and accumulate into a 32-bit
floating-point matrix, just by providing a 32-bit floating-point
matrix as output.  Users may specify the precision of a dot product
result.  If it is greater than the input vectors' element type
precisions (e.g., `double` vs. `float`), then this effectively
performs accumulation in higher precision.

However, we do not include the "Extended-Precision BLAS" in this
proposal.  The BLAS Standard lets callers decide at run time whether
to use extended precision floating-point arithmetic for internal
evaluations.  We could support this feature at a later time.  One way
we could do so is by annotating the execution policy argument using
the [Properties mechanism in P1393](wg21.link/p1393), to `prefer` or
`require` extended-precision evaluation, if it makes sense for the
input and output types.  Implementations of our interface will also
have the freedom to use more accurate evaluation methods than typical
BLAS implementations.

### Arithmetic operators and associated expression templates

Our proposal omits arithmetic operators on matrices and vectors.
We do so for the following reasons:

1. We propose a low-level, minimal interface.

2. `operator*` could have multiple meanings for matrices and vectors.
   We prefer to let a higher-level library decide this.

3. Arithmetic operators require defining the element type of the
   vector or matrix returned by an expression.  Functions let users
   specify this explicitly, and even let users use different output
   types for the same input types in different expressions.

4. Arithmetic operators may require allocation of temporary matrix or
   vector storage.

5. Arithmetic operators strongly suggest expression templates.  These
   introduce problems such as dangling references and aliasing.

Our goal is to propose a low-level interface.  Other libraries, such
as that proposed by [P1385](wg21.link/p1385), could use our interface
to implement overloaded arithmetic for matrices and vectors.
[P0939](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0939r0.pdf)
advocates using "an incremental approach to design to benefit from
actual experience."  A constrained, function-based, BLAS-like
interface builds incrementally on the many years of BLAS experience.

Arithmetic operators on matrices and vectors would require the
library, not necessarily the user, to specify the element type of an
expression's result.  This gets tricky if the terms have mixed element
types.  For example, what should the element type of the result of the
vector sum `x + y` be, if `x` has element type `complex<float>` and
`y` has element type `double`?  It's tempting to use `common_type_t`,
but `common_type_t<complex<float>, double>` is `complex<float>`.  This
loses precision.  Some users may want `complex<double>`; others may
want `complex<long double>` or something else, and others may want to
choose different types in the same program.

[P1385](wg21.link/p1385) lets users customize the return type of such
arithmetic expressions.  However, different algorithms may call for
the same expression with the same inputs to have different output
types.  For example, iterative refinement of linear systems `Ax=b` can
work either with an extended-precision intermediate residual vector `r
= b - A*x`, or with a residual vector that has the same precision as
the input linear system.  Each choice produces a different algorithm
with different convergence characteristics.  Thus, our library lets
users specify the result element type of linear algebra operations
explicitly, by calling a named function that takes an output argument
explicitly, rather than an arithmetic operator.

Arithmetic operators on matrices or vectors may also need to allocate
temporary storage.  Users may not want that.  `mdspan` is a view of
existing memory; it does not come with an allocator.  When LAPACK's
developers switched from Fortran 77 to a subset of Fortran 90, their
users rejected the option of letting LAPACK functions allocate
temporary storage on their own.  Users wanted to control memory
allocation.

Arithmetic expressions on matrices or vectors strongly suggest
expression templates, as a way to avoid allocation of temporaries and
to fuse computational kernels.  They do not *require* expression
templates; for example, `valarray` offers overloaded operators for
vector arithmetic, but the Standard lets implementers decide whether
to use expression templates.  However, all of the current C++ linear
algebra libraries that we mentioned above have some form of expression
templates for overloaded arithmetic operators, so users will expect
this and rely on it for good performance.  This was, indeed, one of
the major complaints about initial implementations of `valarray`: its
lack of mandate for expression templates meant that initial
implementations were slow, and thus users did not want to rely on it.

Expression templates work well, but have issues.  Our papers
[P1417](wg21.link/p1417) and "Evolving a Standard C++ Linear Algebra
Library from the BLAS" (D1674R0) give more detail on these issues.  A
particularly troublesome one is that modern C++ `auto` makes it easy
for users to capture expressions before their evaluation and writing
into an output array.  For matrices and vectors with container
semantics, this makes it easy to create dangling references.  Users
might not realize that they need to assign expressions to named types
before actual work and storage happen.  [Eigen's
documentation](https://eigen.tuxfamily.org/dox/TopicPitfalls.html)
describes this common problem.

Our `scaled_view`, `conjugate_view`, and `transpose_view` functions
make use of one aspect of expression templates, namely modifying the
array access operator.  However, we intend these functions for use
only as in-place modifications of arguments of a function call.  Also,
when modifying `mdspan`, these functions merely view the same data
that their input `mdspan` views.  They introduce no more potential for
dangling references than `mdspan` itself.  The use of views like
`mdspan` is self-documenting; it tells users that they need to take
responsibility for scope of the viewed data.

### Tensors

We exclude tensors from this proposal, for the following reasons.
First, tensor libraries naturally build on optimized dense linear
algebra libraries like the BLAS, so a linear algebra library is a good
first step.  Second, `mdspan` and `mdarray` have natural use as a
low-level representation of dense tensors, so we are already partway
there.  Third, even simple tensor operations that naturally generalize
the BLAS have infintely many more cases than linear algebra.  It's not
clear to us which to optimize.  Fourth, even though linear algebra is
a special case of tensor algebra, users of linear algebra have
different interface expectations than users of tensor algebra.  Thus,
it makes sense to have two separate interfaces.

## Design justification

We take a step-wise approach.  We begin with core BLAS dense linear
algebra functionality.  We then deviate from that only as much as
necessary to get algorithms that behave as much as reasonable like the
existing C++ Standard Library algorithms.  Future work or
collaboration with other proposals could implement a higher-level
interface.  We also offer an option for an extension to "batched BLAS"
in order to support machine learning and other use cases.

We propose to build the interface on top of `basic_mdspan`, as well as
a new `basic_mdarray` variant of `basic_mdspan` with container
semantics.  We explain the value of these two classes below.

Please refer to our papers "Evolving a Standard C++ Linear Algebra
Library from the BLAS" [CITE] and "Historical lessons for C++ linear
algebra library standardization" [(P1417)](wg21.link/p1417).  They
will give details and references for many of the points that we
summarize here.

### Why base a C++ linear algebra library on the BLAS?

1. The BLAS is a standard that codifies decades of existing practice.
2. The BLAS separates out "performance primitives" for hardware
   experts to tune, from mathematical operations that rely on those
   primitives for good performance.
3. Benchmarks reward hardware and system vendors for providing an
   optimized BLAS implementations.
4. Writing a fast BLAS implementation for common element types is
   nontrivial, but well understood.
5. Optimized third-party BLAS implementations with liberal software
   licenses exist.
6. Building a C++ interface on top of the BLAS is a straightforward
   exercise, but has pitfalls for unaware developers.

Linear algebra has had a cross-language standard, the Basic Linear
Algebra Subroutines (BLAS), since 2002.  The Standard came out of a
[standardization process] (http://www.netlib.org/blas/blast-forum/)
that started in 1995 and held meetings three times a year until 1999.
Participants in the process came from industry, academia, and
government research laboratories.  The dense linear algebra subset of
the BLAS codifies forty years of evolving practice, and existed in
recognizable form since 1990 (see e.g., [P1417](wg21.link/p1417)).

The BLAS interface was specifically designed as the distillation of
the "computer science" / performance-oriented parts of linear algebra
algorithms.  It cleanly separates operations most critical for
performance, from operations whose implementation takes expertise in
mathematics and rounding-error analysis.  This gives vendors
opportunities to add value, without asking for expertise outside the
typical required skill set of a Standard Library implementer.

Well-established benchmarks such as the [LINPACK
benchmark](https://www.top500.org/project/linpack/) reward computer
hardware vendors for optimizing their BLAS implementations.  Thus,
many vendors provide an optimized BLAS library for their computer
architectures.  Writing fast BLAS-like operations is not trivial, and
depends on computer architecture.  However, it is not black magic; it
is a well-understood problem whose solutions could be parameterized
for a variety of computer architectures.  See, for example, the 2008
paper ["Anatomy of high-performance matrix
multiplication,"](https://doi.org/10.1145/1356052.1356053) by K. Goto
and R. A. van de Geijn.  There are optimized third-party BLAS
implementations for common architectures, like
[ATLAS](http://math-atlas.sourceforge.net/) and
[GotoBLAS](https://www.tacc.utexas.edu/research-development/tacc-software/gotoblas2).
A (slow but correct) [reference implementation of the
BLAS](http://www.netlib.org/blas/#_reference_blas_version_3_8_0)
exists and it has a liberal software license for easy reuse.

We have experience in the exercise of wrapping a C or Fortran BLAS
implementation for use in portable C++ libraries.  We describe this
exercise in detail in our paper "Evolving a Standard C++ Linear
Algebra Library from the BLAS" (D1674R0).  It is straightforward for
vendors, but has pitfalls for developers.  For example, Fortran's
application binary interface (ABI) differs across platforms in ways
that can cause run-time errors (even incorrect results, not just
crashing).  Historical examples of vendors' C BLAS implementations
have also had ABI issues that required work-arounds.  This dependence
on ABI details makes availability in a standard library valuable.

### We do not require using the BLAS library

Our proposal is based on the BLAS interface, and it would be natural
for implementers to use an existing C or Fortran BLAS library.
However, we do not require an underlying BLAS C interface.  Vendors
should have the freedom to decide whether they want to rely on an
existing BLAS library.  They may also want to write a "pure" C++
implementation that does not depend on an external library.  They
will, in any case, need a "generic" C++ implementation for arbitrary
matrix and vector element types.

### Why use `mdspan` and `mdarray`?

* C++ does not currently have a data structure for representing
  multidimensional arrays.

* The BLAS' C interface takes a large number of pointer and integer
  arguments that represent matrices and vectors.  Using
  multidimensional array data structures in the C++ interface reduces
  the number of arguments and avoids common errors.

* `mdspan` and `mdarray` support row-major, column-major, and strided
  layouts out of the box, and have `Layout` as an extension point.
  This lets our interface support layouts beyond what the BLAS
  Standard supports (column major for the Fortran interface, row- and
  column-major for the less widely available C interface).

* They can exploit any dimensions or strides known at compile time.

* They have built-in "slicing" capabilities via `subspan`.

* Their `Layout` and `Accessor` policies will let us simplify our
  interfaces even further, by encapsulating transpose, conjugate, and
  scalar arguments.  See below for details.

* `mdspan` and `mdarray` are low level; they impose no mathematical
  meaning on multidimensional arrays.  This gives users the freedom to
  develop mathematical libraries with the semantics they want.  (Some
  users object to calling something a "matrix" or "tensor" if it
  doesn't have the right mathematical properties.  The word `vector`
  is already taken.)

* `mdspan` and `mdarray` offer a hook for future expansion to support
  heterogenous memory spaces.

* Their encapsulation of matrix indexing makes C++ implementations of
  BLAS-like operations much less error prone and easier to read.

* They make it easier to support an efficient "batched" interface.

### Why optionally include batched linear algebra?

* Batched interfaces expose more parallelism for many small linear
  algebra operations.

* Batched linear algebra operations are useful for many different
  fields, including machine learning.

* Hardware vendors offer both hardware features and optimized
  software libraries to support batched linear algebra.

* There is an ongoing interface standardization effort.

## Function argument aliasing and zero scalar multipliers

Summary:

1. The BLAS Standard forbids aliasing any input (read-only) argument
   with any output (write-only or read-and-write) argument.

2. The BLAS uses `INTENT(INOUT)` (read-and-write) arguments to express
   "updates" to a vector or matrix.  By contrast, C++ Standard
   algorithms like `transform` take input and output iterator ranges
   as different parameters, but may let input and output ranges be the
   same.

3. The BLAS uses the values of scalar multiplier arguments ("alpha" or
   "beta") of vectors or matrices at run time, to decide whether to
   treat the vectors or matrices as write only.  This matters both for
   performance and semantically, assuming IEEE floating-point
   arithmetic.

4. We decide separately, based on the category of BLAS function, how
   to translate `INTENT(INOUT)` arguments into a C++ idiom:

   a. For in-place triangular solve or triangular multiply, we
      translate the function to take separate input and output
      arguments that shall not alias each other.

   b. Else, if the BLAS function unconditionally updates (like
      `xGER`), we retain read-and-write behavior for that argument.

   c. Else, if the BLAS function uses a scalar `beta` argument to
      decide whether to read the output argument as well as write to
      it (like `xGEMM`), we provide two versions: a write-only version
      (as if `beta` is zero), and a read-and-write version (as if
      `beta` is nonzero).

For a detailed analysis, see "Evolving a Standard C++ Linear Algebra
Library from the BLAS" (D1674R0).

## Support for different matrix layouts

The dense BLAS supports several different dense matrix "types"
(storage formats).  Our paper "Evolving a Standard C++ Linear Algebra
Library from the BLAS" (D1674R0) lists the different matrix types.  In
this proposal, we represent all of these matrix type options as
different layouts, for the following reasons:

* Algorithms can specialize on layout type.  This reduces the number
  of distinct function names.

* Layouts can permit explicit access to implicitly stored values, like
  the "other triangle" in symmetric, Hermitian, or triangular matrix
  types.

## Data structures and utilities borrowed from other proposals

### `basic_mdspan`

[P0009](wg21.link/p0009) is a proposal for adding multidimensional
arrays to the C++ Standard Library.  `basic_mdspan` is the main class
in this proposal.  It is a "view" (in the sense of `span`) of a
multidimensional array.  The rank (number of dimensions) is fixed at
compile time.  Users may specify some dimensions at run time and
others at compile time; the type of the `basic_mdspan` expresses this.
`basic_mdspan` also has two customization points:

  * `Layout` expresses the array's memory layout: e.g., row-major (C++
    style), column-major (Fortran style), or strided.  We use a custom
    `Layout` later in this paper to implement a "transpose view" of an
    existing `basic_mdspan`.

  * `Accessor` defines the storage handle (i.e., `pointer`) stored in
    the `mdspan`, as well as the reference type returned by its access
    operator.  This is an extension point for modifying how access
    happens, for example by using `atomic_ref` to get atomic access to
    every element.  We use custom `Accessor`s later in this paper to
    implement "scaled views" and "conjugated views" of an existing
    `basic_mdspan`.

The `basic_mdspan` class has an alias `mdspan` that uses the default
`Layout` and `Accessor`.  In this paper, when we refer to `mdspan`
without other qualifiers, we mean the most general `basic_mdspan`.

### `basic_mdarray`

`basic_mdspan` views an existing memory allocation.  It does not give
users a way to allocate a new array, even if the array has all
compile-time dimensions.  Furthermore, `basic_mdspan` always stores a
pointer.  For very small matrices or vectors, this is not a
zero-overhead abstraction.  For these reasons, our paper (D1684R0)
proposes a new class `basic_mdarray`.

`basic_mdarray` has the same extension points as `basic_mdspan`, and
also has the ability to use any *contiguous container* (see
**[container.requirements.general]**) for storage.  Contiguity matters
because `basic_mdspan` views a subset of a contiguous pointer range.
`basic_mdarray` also behaves like a container with respect to
construction and assignment.  For example, copy construction and copy
assignment do a "deep copy" of all the entries from the input to the
output.  This means that `basic_mdarray` with compile-time dimensions
need only store the container; it does not need to store a pointer.
When the container is `array`, this makes `basic_mdarray` a
zero-overhead abstraction.  `basic_mdarray` will come with support for
two different underlying containers: `array` (allowed if all
dimensions are known at compile time) and `vector`.

A `subspan` (see [P0009](wg21.link/p0009)) of a `basic_mdarray` will
return a `basic_mdspan` with the appropriate `Layout` and matching
`Accessor`.  Users must guard against dangling pointers, just as they
currently must do when using `span` to view a subset of a `vector`.

The `basic_mdarray` class has an alias `mdarray` that uses the default
`Layout` and `Accessor`.  In this paper, when we refer to `mdarray`
without other qualifiers, we mean the most general `basic_mdarray`.

## Data structures and utilities

### Layouts

Our proposal uses the `Layout` policy of `mdspan` and `mdarray` in
order to represent different matrix and vector data layouts.  Layouts
as described by P0009 come in three different categories:

* Unique
* Contiguous
* Strided

P0009 includes three different layouts -- `layout_left`,
`layout_right`, and `layout_stride` -- all of which are unique,
contiguous, and strided.

This proposal includes the following additional layouts:

* `layout_blas_general`
* `layout_blas_symmetric`
* `layout_blas_symmetric_packed`
* `layout_blas_triangular`
* `layout_blas_triangular_packed`

These layouts have "tag" template parameters that control their
behavior; see below.

These layouts would thus be the first additions to the layouts in
P0009 that are not unique, contiguous, and strided.  We've discussed
above how algorithms cannot be written generically if they have output
arguments with nonunique layouts.  Furthermore, the unpacked
triangular layouts introduce a new idea to P0009, namely that some
indices in the Cartesian product of the object's extents are not
actually valid indices for writing to the object.  The unpacked
triangular layouts are still unique, but the extents do not suffice to
determine the set of validly writeable indices.  This also means that
algorithms cannot be written generically, even for supposedly unique
layouts.  (See the "Options and votes" section below.)

Thus, we impose the following rules:

* Unless otherwise specified, our functions accept input objects that
  are not also output objects, as long as they have any of the
  following layouts:

  * any layout defined in P0009 or this proposal, or

  * any user-defined layout for which any multi-index in the Cartesian
    product of its extents is a valid multi-index for accessing the
    object.

* Unless otherwise specified, none of our functions of our functions
  accept output arguments with nonunique or triangular layouts.

Some functions explicitly require outputs with specific nonunique
layouts.  This includes low-rank updates to symmetric or Hermitian
matrices, and matrix-matrix multiplication with symmetric or Hermitian
matrices.

#### Tag classes for layouts

The number of possible BLAS layouts is combinatorial in the following
options:

* "Base" layout (e.g., column major or row major)
* Symmetric or triangular
* Access only the upper triangle, or access only the lower triangle
* Implicit unit diagonal or explicitly stored diagonal (only for
  triangular)
* Packed or not

Instead of introducing a large number of layout names, we parameterize
a small number of layouts with tags.  Layouts take tag types as
template arguments, and callers of "view" functions use the
corresponding `constexpr` instances of tag types.  The "packed" option
cannot be a tag, because packed layouts have different template
parameters, as we will see below.

##### Storage order tags

```c++
struct column_major_t {};
constexpr column_major_t column_major = column_major_t ();

struct row_major_t {};
constexpr row_major_t row_major = row_major_t ();
```

`column_major_t` indicates a column-major order, and `row_major_t`
indicates a row-major order.  The interpretation of these orders
depends on the specific layout that uses the tag.  See
`layout_blas_general`, `layout_blas_symmetric_packed`, and
`layout_blas_triangular_packed`.

##### Triangle tags

```c++
struct upper_triangle_t {};
constexpr upper_triangle_t upper_triangle = upper_triangle_t ();

struct lower_triangle_t {};
constexpr lower_triangle_t lower_triangle = lower_triangle_t ();
```

These tag classes specify whether algorithms and other users of a
matrix (represented as a `basic_mdspan` or `basic_mdarray`) should
access the upper triangle or lower triangle of the matrix.

The tag `upper_triangle_t` indicates that algorithms or other users of
the view with that tag ("viewer") will only access the upper triangle
of the matrix being viewed ("viewee") by the viewer.  That is, if the
viewee is `A`, then the viewee will only access the element `A(i,j)`
if `i >= j`.  This is also subject to the restrictions of
`implicit_unit_diagonal_t` if that tag is also applied; see below.

The tag `lower_triangle_t` indicates that algorithms or other users of
the view with that tag ("viewer") will only access the lower triangle
of the matrix being viewed ("viewee") by the viewer.  That is, if the
viewee is `A`, then the viewee will only access the element `A(i,j)`
if `i <= j`.  This is also subject to the restrictions of
`implicit_unit_diagonal_t` if that tag is also applied; see below.

##### Diagonal tags

```c++
struct implicit_unit_diagonal_t {};
constexpr implicit_unit_diagonal_t implicit_unit_diagonal =
  implicit_unit_diagonal_t ();

struct explicit_diagonal_t {};
constexpr explicit_diagonal_t explicit_diagonal =
  explicit_diagonal_t ();
```

These tag classes specify what algorithms and other users of a matrix
(represented as a `basic_mdspan` or `basic_mdarray`) should assume
about the diagonal entries of the matrix, and whether algorithms and
users of the matrix should access those diagonal entries explicitly.

The `implicit_unit_diagonal_t` tag indicates that algorithms or other
users of the view with that tag ("viewer") will never refer to the
`i,i` element of the matrix being viewed ("viewee").  Algorithms and
other users of the view will assume that the matrix has a diagonal of
ones (a unit diagonal).  *[Note:* Typical BLAS practice is that the
algorithm never actually needs to form an explicit `1.0`, so there is
no need to impose a constraint that `1` or `1.0` is convertible to
`element_type`. --*end note]*

The tag `explicit_diagonal_t` indicates that algorithms and other
users of the viewer may access the viewee's diagonal entries directly,
by referring to the `i,i` element (if `i,i` is in the viewee's
domain).

##### Packed storage tags

```c++
struct unpacked_storage_t {};
constexpr unpacked_storage_t unpacked_storage =
  unpacked_storage_t ();

struct packed_storage_t {};
constexpr packed_storage_t packed_storage =
  packed_storage_t ();
```

These tag classes specify whether the matrix uses a "packed" storage
format.

The `unpacked_storage_t` tag indicates that the matrix to which the
tag is applied has any of the unpacked storage formats, that is, any
format other than the packed format described in this section.

The `packed_storage_t` tag indicates that the matrix to which the tag
is applied has a packed storage format, which we describe in this
section.

The actual "packed" format depends on

* whether the format represents the upper or lower triangle of the
  matrix (see "Triangle tags" above), and

* whether the format uses column-major or row-major ordering to store
  its entries (see "Storage order tags" above).

#### New "General" layouts

```c++
template<class StorageOrder>
class layout_blas_general;
```

* *Constraints:* `StorageOrder` is either `column_major_t` or
  `row_major_t`.

These new layouts represent exactly the two General (GE) matrix
layouts that the BLAS' C interface supports.  Both of these are more
general than P0009's `layout_left` and `layout_right`, because they
permit a stride between columns resp. rows that is greater than the
corresponding extent.  This is why BLAS functions take an "LDA"
(leading dimension of the matrix A) argument separate from the
dimenions (extents, in `mdspan` terms) of A.  However, these layouts
are slightly *less* general than P0009's `layout_stride`, because they
assume contiguous storage of columns resp. rows.

We could omit these layouts and use `layout_stride` without removing
functionality.  However, the advantage of these layouts is that
subspans taken by many matrix algorithms preserve the layout type (if
the stride is a run-time value).  Many matrix algorithms work on
"submatrices" that are rank-2 subspans of contiguous rows and columns
of a "parent" matrix.  If the parent matrix is `layout_left`, then in
general, the submatrix is `layout_stride`, not `layout_left`.
However, if the parent matrix is `layout_blas_general<column_major_t>`
or `layout_blas_general<row_major_t>`, such submatrices always have
the same layout as their parent matrix.  Algorithms on submatrices may
thus always assume contiguous access along one dimension.

These new layouts have natural generalizations to ranks higher than 2.
The definition of each of these layouts would look and work like
`layout_stride`, except for the following differences:

* `layout_blas_general::mapping` would be templated on two `extent`
  types.  The first would express the `mdspan`'s dimensions, just like
  with `layout_left`, `layout_right`, or `layout_stride`.  The second
  would express the `mdspan`'s strides.  The second `extent` would
  have rank `rank()-1`.

* `layout_blas_general::mapping`'s constructor would take an instance
  of each of these two `extent` specializations.

These new layouts differ intentionally from `layout_stride`, which
takes strides all as run-time elements in an `array`.  (P0009 may
later change this to be a second `extents` object.)  We want users to
be able to express strides as an arbitrary mix of compile-time and
run-time values, just as they can express dimensions.

#### New unpacked symmetric and triangular layouts

These layouts make sense for rank-2 and rank-3 `mdspan` and `mdarray`.
In the rank-3 case, they would represent a batch of matrices, where
the leftmost dimension indicates which matrix.

##### Requirements

Throughout this Clause, where the template parameters are not
constrained, the names of template parameters are used to express type
requirements.

  * `Triangle` is either `upper_triangle_t` or `lower_triangle_t`.

  * `DiagonalStorage` is either `implicit_unit_diagonal_t` or
    `explicit_diagonal_t`.

  * `BaseLayout` is a unique, contiguous, and strided layout.

##### Unpacked symmetric layout

```c++
template<class Triangle,
         class BaseLayout>
class layout_blas_symmetric {
public:
  using triangle_type = Triangle;
  using base_layout_type = BaseLayout;

  template<class Extents>
  class mapping {
  public:
    using base_mapping_type = base_layout_type::template mapping<Extents>;

    constexpr mapping() noexcept;
    constexpr mapping(const mapping&) noexcept;
    constexpr mapping(mapping&&) noexcept;

    constexpr mapping(const base_mapping_type& base_mapping) noexcept :
      base_mapping_ (base_mapping) {}

    // ... TODO more details here; see layout_left ...

    const base_mapping_type base_mapping() const {
      return base_mapping_;
    }

  private:
    base_mapping_type base_mapping_; // <i>exposition only</i>
  };
};
```

If `Triangle` is `upper_triangle_t`, then this layout's mapping for
`i,j` invokes the `BaseLayout`'s `i,j` mapping when `i` is greater
than or equal to `j`, and invokes the `BaseLayout`'s `j,i` mapping
when `i` is less than `j`.

If `Triangle` is `lower_triangle_t`, then this layout's mapping for
`i,j` invokes the `BaseLayout`'s `i,j` mapping when `i` is less than
or equal to `j`, and invokes the `BaseLayout`'s `j,i` mapping when `i`
is greater than `j`.

*[Note:* Algorithms may avoid the overhead of checking and swapping
indices by accessing the base mapping. --*end note]*

##### Unpacked triangular layout

```c++
template<class Triangle,
         class DiagonalStorage,
         class BaseLayout>
class layout_blas_triangular;
```


#### New packed symmetric and triangular layouts

Throughout this Clause, where the template parameters are not
constrained, the names of template parameters are used to express type
requirements.

  * `Triangle` is either `upper_triangle_t` or `lower_triangle_t`.

  * `DiagonalStorage` is either `implicit_unit_diagonal_t` or
    `explicit_diagonal_t`.

  * `StorageOrder` is either `column_major_t` or `row_major_t`.

##### Packed layout mapping

Both symmetric and triangular packed formats store the represented
entries of the matrix contiguously, and only represent one triangle of
the matrix (either upper or lower).  They start at the top left side
of the matrix.  For column-major ordering, they pack entries starting
with the leftmost (least column index) column (or the second leftmost
column, if the diagonal is stored implicitly), and proceeding column
by column, from the top entry (least row index).  For row-major
ordering, they pack entries starting with the topmost (least row
index) row, and proceeding row by row, from the leftmost (least column
index) entry.

Let a packed matrix have N rows.

* For the upper triangular, column-major format, index pair i,j maps
  to i + (1 + 2 + ... + j).

* For the lower triangular, column-major format, index pair i,j maps
  to i + (1 + 2 + ... + N-j-1).

* For the upper triangular, row-major format, index pair i,j maps
  to j + (1 + 2 + ... + i).

* For the lower triangular, row-major format, index pair i,j maps
  to j + (1 + 2 + ... + N-i-1).

*[Note:* Whether or not the storage format has an implicit unit
 diagonal (see the `implicit_unit_diagonal_t` tag above) does not
 change the mapping.  This means that packed matrix storage "wastes"
 the unit diagonal, if present.  This follows BLAS convention; see
 Section 2.2.4 of the BLAS Standard.  It also has the advantage that
 every index pair `i,j` in the Cartesian product of the extents maps
 to a valid (though wrong) codomain index.  This is why we declare the
 packed layout mappings as "nonunique." --*end note]

##### Layouts

```c++
template<class Triangle,
         class StorageOrder>
class layout_blas_symmetric_packed;

template<class Triangle,
         class DiagonalStorage,
         class StorageOrder>
class layout_blas_triangular_packed;
```

### Layout views

BLAS functions often take an existing matrix or vector data structure,
and "view" a subset of its elements as another data structure.  For
example, the triangular solve functions take a square matrix in
General (GE) storage format, and view either its upper or lower
triangle (TR format).  Matrix factorizations exploit this.  For
instance, LU (Gaussian elimination) computes the lower triangular
factor L with an implicitly stored diagonal of ones ("unit diagonal"),
so that it can overwrite the original matrix A with both L and the
upper triangular factor U.

One of our design goals is to reduce the number of BLAS function
parameters.  We can eliminate the `UPLO` (upper or lower triangle) and
`DIAG` (implicit unit diagonal or explicitly stored diagonal)
parameters by letting callers annotate an existing General-format
matrix argument with a new layout that encapsulates both parameters.

#### Requirements

Throughout this Clause, where the template parameters are not
constrained, the names of template parameters are used to express type
requirements.

  * `Triangle` is either `upper_triangle_t` or `lower_triangle_t`.

  * `DiagonalStorage` is either `implicit_unit_diagonal_t` or
    `explicit_diagonal_t`.

  * `StorageOrder` is either `column_major_t` or `row_major_t`.

  * `Packing` is either `packed_storage_t` or `unpacked_storage_t`.

#### Create an unpacked triangular view of an existing rank-2 object

```c++
// Upper triangular, explicit diagonal

template<class EltType, class Extents, class Layout,
         class Accessor>
constexpr basic_mdspan<EltType, Extents,
  layout_blas_triangular<
    upper_triangle_t,
    explicit_diagonal_t,
    Layout>,
  Accessor>
triangular_view(
  basic_mdspan<EltType, Extents, Layout, Accessor> m,
  upper_triangle_t,
  explicit_diagonal_t d = explicit_diagonal);

template<class EltType, class Extents, class Layout,
         class Accessor>
constexpr basic_mdspan<EltType, Extents,
  layout_blas_triangular<
    upper_triangle_t,
    explicit_diagonal_t,
    Layout>,
  Accessor>
triangular_view(
  basic_mdarray<EltType, Extents, Layout, Accessor>& m,
  upper_triangle_t,
  explicit_diagonal_t d = explicit_diagonal);

template<class EltType, class Extents, class Layout,
         class Accessor>
constexpr basic_mdspan<const EltType, Extents,
  layout_blas_triangular<
    upper_triangle_t,
    explicit_diagonal_t,
    Layout>,
  Accessor>
triangular_view(
  const basic_mdarray<EltType, Extents, Layout, Accessor>& m,
  upper_triangle_t,
  explicit_diagonal_t d = explicit_diagonal);

// Upper triangular, implicit unit diagonal

template<class EltType, class Extents, class Layout,
         class Accessor>
constexpr basic_mdspan<EltType, Extents,
  layout_blas_triangular<
    upper_triangle_t,
    implicit_unit_diagonal_t,
    Layout>,
  Accessor>
triangular_view(
  basic_mdspan<EltType, Extents, Layout, Accessor> m,
  upper_triangle_t,
  implicit_unit_diagonal_t);

template<class EltType, class Extents, class Layout,
         class Accessor>
constexpr basic_mdspan<EltType, Extents,
  layout_blas_triangular<
    upper_triangle_t,
    implicit_unit_diagonal_t,
    Layout>,
  Accessor>
triangular_view(
  basic_mdarray<EltType, Extents, Layout, Accessor>& m,
  upper_triangle_t,
  implicit_unit_diagonal_t);

template<class EltType, class Extents, class Layout,
         class Accessor>
constexpr basic_mdspan<const EltType, Extents,
  layout_blas_triangular<
    upper_triangle_t,
    implicit_unit_diagonal_t,
    Layout>,
  Accessor>
triangular_view(
  const basic_mdarray<EltType, Extents, Layout, Accessor>& m,
  upper_triangle_t,
  implicit_unit_diagonal_t);

// Lower triangular, implicit unit diagonal

template<class EltType, class Extents, class Layout,
         class Accessor>
constexpr basic_mdspan<EltType, Extents,
  layout_blas_triangular<
    lower_triangle_t,
    implicit_unit_diagonal_t,
    Layout>,
  Accessor>
triangular_view(
  basic_mdspan<EltType, Extents, Layout, Accessor> m,
  lower_triangle_t,
  implicit_unit_diagonal_t d = implicit_unit_diagonal);

template<class EltType, class Extents, class Layout,
         class Accessor>
constexpr basic_mdspan<EltType, Extents,
  layout_blas_triangular<
    lower_triangle_t,
    implicit_unit_diagonal_t,
    Layout>,
  Accessor>
triangular_view(
  basic_mdarray<EltType, Extents, Layout, Accessor>& m,
  lower_triangle_t,
  implicit_unit_diagonal_t d = implicit_unit_diagonal);

template<class EltType, class Extents, class Layout,
         class Accessor>
constexpr basic_mdspan<const EltType, Extents,
  layout_blas_triangular<
    lower_triangle_t,
    implicit_unit_diagonal_t,
    Layout>,
  Accessor>
triangular_view(
  const basic_mdarray<EltType, Extents, Layout, Accessor>& m,
  lower_triangle_t,
  implicit_unit_diagonal_t d = implicit_unit_diagonal);

// Lower triangular, explicit diagonal

template<class EltType, class Extents, class Layout,
         class Accessor>
constexpr basic_mdspan<EltType, Extents,
  layout_blas_triangular<
    lower_triangle_t,
    explicit_diagonal_t,
    Layout>,
  Accessor>
triangular_view(
  basic_mdspan<EltType, Extents, Layout, Accessor> m,
  lower_triangle_t,
  explicit_diagonal_t);

template<class EltType, class Extents, class Layout,
         class Accessor>
constexpr basic_mdspan<EltType, Extents,
  layout_blas_triangular<
    lower_triangle_t,
    explicit_diagonal_t,
    Layout>,
  Accessor>
triangular_view(
  basic_mdarray<EltType, Extents, Layout, Accessor>& m,
  lower_triangle_t,
  explicit_diagonal_t);

template<class EltType, class Extents, class Layout,
         class Accessor>
constexpr basic_mdspan<const EltType, Extents,
  layout_blas_triangular<
    lower_triangle_t,
    explicit_diagonal_t,
    Layout>,
  Accessor>
triangular_view(
  const basic_mdarray<EltType, Extents, Layout, Accessor>& m,
  lower_triangle_t,
  explicit_diagonal_t);
```

* *Constraints:*

  * `m.rank()` is at least 2.

  * `Layout` is a unique, contiguous, and strided layout.

#### Create a packed triangular view of an existing rank-1 object

```c++
template<class EltType,
         class Extents,
         class Layout,
         class Accessor,
         class Triangle,
         class DiagonalStorage,
         class StorageOrder>
constexpr basic_mdspan<EltType,
  <i>rank-two-extents-see-returns-below</i>,
  layout_blas_triangular_packed<
    Triangle,
    DiagonalStorage,
    StorageOrder>,
  Accessor>
packed_triangular_view(
  basic_mdspan<EltType, Extents, Layout, Accessor>& m,
  typename basic_mdarray<EltType, Extents, Layout, Accessor>::index_type num_rows,
  Triangle,
  DiagonalStorage,
  StorageOrder);

template<class EltType,
         class Extents,
         class Layout,
         class Accessor,
         class Triangle,
         class DiagonalStorage,
         class StorageOrder>
constexpr basic_mdspan<const EltType,
  <i>rank-two-extents-see-returns-below</i>,
  layout_blas_triangular_packed<
    Triangle,
    DiagonalStorage,
    StorageOrder>,
  Accessor>
packed_triangular_view(
  const basic_mdarray<EltType, Extents, Layout, Accessor>& m,
  typename basic_mdarray<EltType, Extents, Layout, Accessor>::index_type num_rows,
  Triangle,
  DiagonalStorage,
  StorageOrder);

template<class EltType,
         class Extents,
         class Layout,
         class Accessor,
         class Triangle,
         class DiagonalStorage,
         class StorageOrder>
constexpr basic_mdspan<EltType,
  <i>rank-two-extents-see-returns-below</i>,
  layout_blas_triangular_packed<
    Triangle,
    DiagonalStorage,
    StorageOrder>,
  Accessor>
packed_triangular_view(
  basic_mdarray<EltType, Extents, Layout, Accessor>& m,
  typename basic_mdarray<EltType, Extents, Layout, Accessor>::index_type num_rows,
  Triangle,
  DiagonalStorage,
  StorageOrder);
```

* *Requires:*

  * If `num_rows` is nonzero and if `DiagonalStorage` is
    `implicit_unit_diagonal_t`, then `m.extent(0)` is at least
    `num_rows` * (`num_rows` - 1) / 2.

  * If `num_rows` is nonzero and if `DiagonalStorage` is
    `explicit_diagonal_t`, then `m.extent(0)` is at least (`num_rows`
    + 1) * `num_rows` / 2.

* *Constraints:*

  * `Extents::rank()` is one.

  * `m.is_always_contiguous()` and `m.is_always_unique()` are both true.

* *Effects:* Views the given rank-1 contiguous unique `basic_mdspan`
   or `basic_mdarray` as a packed triangular matrix with the given
   `Triangle`, `DiagonalStorage`, and `StorageOrder`.

* *Returns:* A rank-2 `basic_mdspan` `r` with packed triangular
   layout.  If `E_r` is the type of `r.extents()`, then `E_r` has at
   least as many `StaticExtents` (the number of `extents`'s
   `ptrdiff_t` template arguments) as the type of `m.extents()` has.

### Scaled view of an object

Most BLAS functions that take scalar arguments use those arguments as
a transient scaling of another vector or matrix argument.  For
example, `xAXPY` computes `y = alpha*x + y`, where `x` and `y` are
vectors, and `alpha` is a scalar.  Scalar arguments help make the BLAS
more efficient by combining related operations and avoiding temporary
vectors or matrices.  In this `xAXPY` example, users would otherwise
need a temporary vector `z = alpha*x` (`xSCAL`), and would need to
make two passes over the input vector `x` (once for the scale, and
another for the vector add).  However, scalar arguments complicate the
interface.

We can solve all these issues in C++ by introducing a "scaled view" of
an existing vector or matrix, via a changed `basic_mdspan` `Accessor`.
For example, users could imitate what `xAXPY` does by using our
`linalg_add` function (see below) as follows:

```c++
mdspan<double, extents<dynamic_extent>> y = ...;
mdspan<double, extents<dynamic_extent>> x = ...;
double alpha = ...;

linalg_add(scaled_view(alpha, x), y, y);
```

The resulting operation would only need to iterate over the entries of
the input vector `x` and the input / output vector `y` once.  An
implementation could dispatch to the BLAS by noticing that the first
argument has an `accessor_scaled` (see below) `Accessor` type,
extracting the scalar value `alpha`, and calling the corresponding
`xAXPY` function (assuming that `alpha != 0`; see discussion above).

The same `linalg_add` interface would then support the operation `w :=
alpha*x + beta*y`:

```c++
linalg_add(scaled_view(alpha, x), scaled_view(beta, y), w);
```

Note that this operation could not dispatch to an existing BLAS
library, unless the library implements the `xWAXPBY` function
specified in the BLAS Standard.  However, implementations could
specialize on the result of a `scaled_view`, in order to transform the
user's arguments into something suitable for a BLAS library call.  For
example, if the user calls `matrix_product` (see below) with `A` and
`B` both results of `scaled_view`, then the implementation could
combine both scalars into a single "alpha" and call `xGEMM` with it.

#### `scaled_scalar`

`scaled_scalar` expresses a scaled version of an existing scalar.
This must be read only, in order to avoid unpleasantly surprising
results.

```c++
template<class T, class S>
class scaled_scalar {
public:
  scaled_scalar(const T& v, const S& s) :
    val(v), scale(s) {}

  template<class T2>
  T operator* (const T2 upd) const {
    return val*scale * upd;
  }
  template<class T2>
  T operator+ (const T2 upd) const {
    return val*scale + upd;
  }
  //...

private:
  const T& val;
  const S scale;
};
```
#### `accessor_scaled`

Accessor to make `basic_mdspan` return a `scaled_scalar`.
```c++
template<class Accessor, class S>
class accessor_scaled {
public:
  using element_type  = Accessor::element_type;
  using pointer       = Accessor::pointer;
  using reference     = scaled_scalar<Accessor::reference,S>;
  using offset_policy = accessor_scaled<Accessor,S>;

  accessor_scaled(Accessor a, S sval) :
    acc(a), scale_factor(sval) {}

  scaled_scalar<T,S> access(pointer& p, ptrdiff_t i) {
    return scaled_scalar<T,S>(acc.access(p,i), scale_factor);
  }

private:
  Accessor acc;
  S scale_factor;
};
```

#### `scaled_view`

Return a scaled view using a new accessor.
```c++
template<class T, class Extents, class Layout,
         class Accessor, class S>
basic_mdspan<T, Extents, Layout, accessor_scaled<Accessor, S>>
scaled_view(S s, basic_mdspan<T, Extents, Layout, Accessor> a);

template<class T, class Extents, class Layout,
         class Accessor, class S>
basic_mdspan<T,Extents,Layout,accessor_scaled<Accessor::offset_policy,S>>
scaled_view(S s, const basic_mdarray<T, Extents, Layout, Accessor>& a);
```

*Example:*

```c++
void test_scaled_view(basic_mdspan<double,extents<10>> a)
{
  auto a_scaled = scaled_view(5.0, a);
  for(int i = 0; i < a.extent(0); ++i)
    assert(a_scaled(i) == 5.0 * a(i));
}
```

### Conjugated view of an object

Some BLAS functions of matrices also take an argument that specifies
whether to view the transpose or conjugate transpose of the matrix.
The BLAS uses this argument to modify a read-only input transiently.
This means that users can let the BLAS work with the data in place,
without needing to compute the transpose or conjugate transpose
explicitly.  However, it complicates the BLAS interface.

Just as we did above with "scaled views" of an object, we can apply
the complex conjugate operation to each element of an object using a
special accessor.

What does the complex conjugate mean for non-complex numbers?  One
convention, which the [Trilinos](https://github.com/trilinos/Trilinos)
library (among others) uses, is that the "complex conjugate" of a
non-complex number is just the number.  This makes sense
mathematically, if we embed a field (of real numbers) in the
corresponding set of complex numbers over that field, as all complex
numbers with zero imaginary part.  However, as we will show below,
this does not work with the C++ Standard Library's definition of
`conj`.  Thus, the least invasive option for us is to permit creating
a conjugated view of an object, only if the object's element type is
complex.

#### `conjugated_scalar`

`conjugated_scalar` expresses a conjugated version of an existing
scalar.  This must be read only, in order to avoid unpleasantly
surprising results.  Otherwise, `conjugated_scalar` behaves a bit like
`atomic_ref`, in that it is a special `reference` type for the
Accessor `accessor_conjugate` (see below), and that it provides
overloaded arithmetic operators that implement a limited form of
expression templates.  The arithmetic operators are templated on their
input type, so that they only need to compile if an algorithm actually
uses them.

The C++ Standard imposes the following requirements on `complex<T>`
numbers:

1. `T` may only be `float`, `double`, or `long double`.

2. Overloads of `conj(const T&)` exist for `T=float`, `double`, `long
   double`, or any built-in integer type, but all these overloads have
   return type `complex<U>` with `U` either `float`, `double`, or
   `long double`.  (See **[cmplx.over]**.)

We need the return type of `conjugated_scalar`'s arithmetic operators
to be the same as the type of the scalar that it wraps.  This means
that `conjugated_scalar` only works for `complex<T>` scalar types.
Users cannot define custom types that are complex numbers.  (The
alternative would be to permit users to specialize
`conjugated_scalar`, but we didn't want to add a *customization point*
in the sense of **[namespace.std]**.  Our definition of
`conjugated_scalar` is compatible with any future expansion of the C++
Standard to permit `complex<T>` for other `T`.)

```c++
template<class T>
class conjugated_scalar {
public:
  using value_type = T;

  conjugated_scalar(const T& v) : val(v) {}

  template<class T2>
  T operator* (const T2 upd) const {
    return conj (val) * upd;
  }

  template<class T2>
  T operator+ (const T2 upd) const {
    return conj (val) + upd;
  }

  // ...

private:
  const T& val;
};
```

#### `accessor_conjugate`

The `accessor_conjugate` Accessor makes `basic_mdspan` return a
`conjugated_scalar`.

```c++
template<class Accessor, class S>
class accessor_conjugate {
public:
  using element_type  = Accessor::element_type;
  using pointer       = Accessor::pointer;
  using reference     = conjugated_scalar<Accessor::reference,S>;
  using offset_policy = accessor_conjugate<Accessor,S>;

  conjugated_scaled(Accessor a) : acc(a) {}

  conjugated_scalar<T,S> access(pointer& p, ptrdiff_t i) {
    return conjugated_scalar<T,S>(acc.access(p,i),scale_factor);
  }

private:
  Accessor acc;
};
```

#### `conjugate_view`

The `conjugate_view` function returns a conjugated view using a new
accessor.

```c++
template<class T, class Extents, class Layout, class Accessor>
basic_mdspan<T, Extents, Layout,
             accessor_conjugate<Accessor>>
conjugate_view(basic_mdspan<T,Extents,Layout,Accessor> a);

template<class T, class Extents, class Layout, class Accessor>
basic_mdspan<T, Extents, Layout,
             accessor_conjugate<Accessor::offset_policy>>
conjugate_view(const basic_mdarray<T, Extents, Layout, Accessor>& a);
```

*Example:*

```c++
void test_conjugate_view(basic_mdspan<complex<double>, extents<10>>)
{
  auto a_conj = conjugate_view(a);
  for(int i = 0; i < a.extent(0); ++i)
    assert(a_conj(i) == conj(a(i));
}
```

### Transpose view of an object

Many BLAS functions of matrices take an argument that specifies
whether to view the transpose or conjugate transpose of the matrix.
The BLAS uses this argument to modify a read-only input transiently.
This means that users can let the BLAS work with the data in place,
without needing to compute the transpose or conjugate transpose
explicitly.  However, it complicates the BLAS interface.

Just as we did above with a "scaled view" of an object, we can
construct a "transposed view" or "conjugate transpose" view of an
object.  This lets us simplify the interface.

An implementation could dispatch to the BLAS by noticing that the
first argument has an `layout_transpose` (see below) `Layout` type
(in both transposed and conjugate transposed cases), and/or an
`accessor_conjugate` (see below) `Accessor` type (in the conjugate
transposed case).  It could use this information to extract the
appropriate run-time BLAS parameters.

#### `layout_transpose`

Layout to wrap other layouts and swap indices. Only defines operators
for 2D and 3D. Will swap last two indices.

```c++
template<class Layout>
class layout_transpose {
  struct mapping {
    Layout::mapping nested_mapping;

    mapping(Layout::mapping map):nested_mapping(map) {}

    ptrdiff_t operator() (ptrdiff_t i, ptrdiff_t j) const {
      return nested_mapping(j,i);
    }
    ptrdiff_t operator() (ptrdiff_t k, ptrdiff_t i, ptrdiff_t j) const {
      return nested_mapping(k,j,i);
    }
  };
};
```

#### `transpose_view`

The `transpose_view` function returns a transposed view of an object.
For rank-2 objects `mdspan` and `mdarray`, the transposed view swaps
the row and column indices.  For batched BLAS (rank-3) `mdspan` and
`mdarray`, the transposed view swaps the last two indices.  (The first
index indicates which matrix in the batch is being accessed.)

Note that `transpose_view` always returns an `mdspan` with the
`layout_transpose` argument.  This gives a type-based indication of
the transpose operation.  However, functions' implementations may
convert the `layout_transpose` object to an object with a different
but equivalent layout.  For example, functions can view the transpose
of a `layout_blas<column_major_t>` matrix as a
`layout_blas<row_major_t>` matrix.  (This is a classic technique for
supporting row-major matrices using the Fortran BLAS interface.)

```c++
template<class T, class Extents, class Layout, class Accessor>
basic_mdspan<T, Extents, layout_transpose<Layout>, Accessor>>
transpose_view(basic_mdspan<T, Extents, Layout, Accessor> a);

template<class T, class Extents, class Layout, class Accessor>
basic_mdspan<T, Extents, layout_transpose<Layout>,
             Accessor::offset_policy>>
transpose_view(const basic_mdarray<T, Extents, Layout, Accessor>& a);
```

#### Conjugate transpose view

The `conjugate_transpose_view` function returns a conjugate transpose
view of an object.  This combines the effects of `transpose_view` and
`conjugate_view`.

```c++
template<class T, class Extents, class Layout, class Accessor>
basic_mdspan<T, Extents, layout_transpose<Layout>,
             accessor_conjugate<Accessor>>>
conjugate_transpose_view(basic_mdspan<T,Extents,Layout,Accessor> a);

template<class T, class Extents, class Layout, class Accessor>
basic_mdspan<T, Extents, layout_transpose<Layout>,
             accessor_conjugate<Accessor::offset_policy>>>
conjugate_transpose_view(const basic_mdarray<T, Extents, Layout, Accessor>& a)
```

## Algorithms

### Requirements

Throughout this Clause, where the template parameters are not
constrained, the names of template parameters are used to express type
requirements.

* Algorithms that have a template parameter named `ExecutionPolicy`
  are parallel algorithms **[algorithms.parallel.defns]**.

* `Scalar` is generally a "numeric" value type.

* `Real` is any of the following types: `float`, `double`, or `long
  double`.

* `in_vector_*_t` is a rank-1 `basic_mdspan` or `basic_mdarray` with a
  `const` element type.  If the algorithm accesses the object, it will
  do so in read-only fashion.

* `inout_vector_*_t` is a rank-1 `basic_mdspan` or `basic_mdarray`
  with a non-`const` element type.

* `out_vector_*_t` is a rank-1 `basic_mdspan` or `basic_mdarray` with
  a non-`const` element type.  If the algorithm accesses the object,
  it will do so in write-only fashion.

* `in_matrix_*_t` is a rank-2 `basic_mdspan` or `basic_mdarray` with a
  `const` element type.  If the algorithm accesses the object, it will
  do so in read-only fashion.

* `inout_matrix_*_t` is a rank-2 `basic_mdspan` or `basic_mdarray`
  with a non-`const` element type.

* `out_matrix_*_t` is a rank-2 `basic_mdspan` or `basic_mdarray` with
  a non-`const` element type.  If the algorithm accesses the object,
  it will do so in write-only fashion.

* `in_object_*_t` is a rank-1 or rank-2 `basic_mdspan` or
  `basic_mdarray` with a `const` element type.  If the algorithm
  accesses the object, it will do so in read-only fashion.

* `inout_object_*_t` is a rank-1 or rank-2 `basic_mdspan` or
  `basic_mdarray` of a non-`const` element type.

* `out_object_*_t` is a rank-1 or rank-2 `basic_mdspan` or
  `basic_mdarray` of a non-`const` element type.

All functions take "input" (read-only) object parameters by const
reference (e.g., `const basic_mdspan<...>&` or `const
basic_mdarray<...>&`).  All such functions have overloads that take
the same parameter by rvalue (`&&`) reference.

All functions take "input/output" or "output" object parameters as
follows:

* by nonconst reference if they are `basic_mdarray` (i.e.,
  `basic_mdarray<T, ...>&` for nonconst `T`); or,

* by const reference if they are `basic_mdspan` (i.e., `const
  basic_mdspan<T, ...>&` for nonconst `T`).

### BLAS 1 functions

#### Givens rotations

##### Compute Givens rotations

```c++
template<class Real>
void givens_rotation_setup(const Real a,
                           const Real b,
                           Real& c,
                           Real& s);

template<class Real>
void givens_rotation_setup(const complex<Real>& a,
                           const complex<Real>& b,
                           Real& c,
                           complex<Real>& s);
```

This function computes the plane (Givens) rotation represented by the
two values `c` and `s` such that the mathematical expression

```
[c        s]   [a]   [r]
[          ] * [ ] = [ ]
[-conj(s) c]   [b]   [0]
```

holds, where `conj` indicates the mathematical conjugate of `s`, `c`
is always a real scalar, and `c*c + abs(s)*abs(s)` equals one.  That
is, `c` and `s` represent a 2 x 2 matrix, that when multiplied by the
right by the input vector whose components are `a` and `b`, produces a
result vector whose first component `r` is the Euclidean norm of the
input vector, and whose second component as zero.  *[Note:* The C++
Standard Library `conj` function always returns `complex<T>` for some
`T`, even though overloads exist for non-complex input.  The above
exprssion uses `conj` as mathematical notation, not as code.  --*end
note]*

*[Note:* This function corresponds to the BLAS function `xROTG`.  It
has an overload for complex numbers, because the output argument `c`
(cosine) is a signed magnitude. --*end note]*

* *Constraints:* `Real` is `float`, `double`, or `long double`.

* *Effects:* Assigns to `c` and `s` the plane (Givens) rotation
  corresponding to the input `a` and `b`.

* *Throws:* Nothing.

##### Apply a computed Givens rotation to vectors

```c++
template<class ExecutionPolicy,
         class inout_vector_1_t,
         class inout_vector_2_t,
         class Real>
void givens_rotation(ExecutionPolicy&& exec,
                     inout_vector_1_t v1,
                     inout_vector_2_t v2,
                     const Real c,
                     const Real s);

template<class inout_vector_1_t,
         class inout_vector_2_t,
         class Real>
void givens_rotation(inout_vector_1_t v1,
                     inout_vector_2_t v2,
                     const Real c,
                     const Real s);

template<class ExecutionPolicy,
         class inout_vector_1_t,
         class inout_vector_2_t,
         class Real>
void givens_rotation(ExecutionPolicy&& exec,
                     inout_vector_1_t v1,
                     inout_vector_2_t v2,
                     const Real c,
                     const complex<Real> s);

template<class inout_vector_1_t,
         class inout_vector_2_t,
         class Real>
void givens_rotation(inout_vector_1_t v1,
                     inout_vector_2_t v2,
                     const Real c,
                     const complex<Real> s);
```

*[Note:* The `givens_rotation` functions correspond to the BLAS
function `xROT`. --*end note]*

* *Requires:*

  * `c` and `s` form a plane (Givens) rotation.  *[Note:* Users
    normally would compute `c` and `s` using `givens_rotation_setup`,
    but they are not required to do this. --*end note]*

  * `v1` and `v2` have the same domain.

* *Constraints:*

  * `Real` is `float`, `double`, or `long double`.

  * `v1.rank()` and `v2.rank()` are both one.

  * For the overloads that take the last argument `s` as `Real`, for
    `i` in the domain of `v1` and `j` in the domain of `v2`, the
    expressions `v1(i) = c*v1(i) + s*v2(j)` and `v2(j) = -s*v1(i) +
    c*v2(j)` are well formed.

  * For the overloads that take the last argument `s` as `const
    complex<Real>`, for `i` in the domain of `v1` and `j` in the
    domain of `v2`, the expressions `v1(i) = c*v1(i) + s*v2(j)` and
    `v2(j) = -conj(s)*v1(i) + c*v2(j)` are well formed.

* *Effects:* Applies the plane (Givens) rotation specified by `c` and
  `s` to the input vectors `v1` and `v2`, as if the rotation were a 2
  x 2 matrix and the input vectors were successive rows of a matrix
  with two rows.

#### Swap matrix or vector elements

```c++
template<class inout_object_1_t,
         class inout_object_2_t>
void linalg_swap(inout_object_1_t v1,
                 inout_object_2_t v2);

template<class ExecutionPolicy,
         class inout_object_1_t,
         class inout_object_2_t>
void linalg_swap(ExecutionPolicy&& exec,
                 inout_object_1_t v1,
                 inout_object_2_t v2);
```

*[Note:* These functions correspond to the BLAS function `xSWAP`.
--*end note]*

* *Requires:* `v1` and `v2` have the same domain.

* *Constraints:*

  * `v1.rank()` equals `v2.rank()`.

  * For `i...` in the domain of `v2` and `v1`, the
    expression `v2(i...) = v1(i...)` is well formed.

* *Effects:* Swap all corresponding elements of the vectors
  `v1` and `v2`.

#### Multiply the elements of an object in place by a scalar

```c++
template<class Scalar,
         class inout_object_t>
void scale(const Scalar alpha,
           inout_object_t obj);

template<class ExecutionPolicy,
         class Scalar,
         class inout_object_t>
void scale(ExecutionPolicy&& exec,
           const Scalar alpha,
           inout_object_t obj);
```

*[Note:* These functions correspond to the BLAS function `xSCAL`.
--*end note]*

* *Constraints:* For `i...` in the domain of `obj`, the expression
  `obj(i...) *= alpha` is well formed.

* *Effects*: Multiply each element of `obj` in place by `alpha`.

#### Copy elements of one matrix or vector into another

```c++
template<class in_object_t,
         class out_object_t>
void linalg_copy(in_object_t x,
                 out_object_t y);

template<class ExecutionPolicy,
         class in_object_t,
         class out_object_t>
void linalg_copy(ExecutionPolicy&& exec,
                 in_object_t x,
                 out_object_t y);
```

*[Note:* These functions correspond to the BLAS function `xCOPY`.
--*end note]*

* *Constraints:*

  * `x.rank()` equals `y.rank()`.

  * `x.rank()` is less than 3.

  * For all `i...` in the domain of `x` and `y`, the expression
    `y(i...) = x(i...)` is well formed.

* *Requires:* The domain of `y` equals the domain of `x`.

* *Effects:* Overwrite each element of `y` with the corresponding
  element of `x`.

#### Add vectors or matrices elementwise

```c++
template<class in_object_1_t,
         class in_object_2_t,
         class out_object_t>
void linalg_add(in_object_1_t x,
                in_object_2_t y,
                out_object_t z);

template<class ExecutionPolicy,
         class in_object_1_t,
         class in_object_2_t,
         class out_object_t>
void linalg_add(ExecutionPolicy&& exec,
                in_object_1_t x,
                in_object_2_t y,
                out_object_t z);
```

*[Note:* These functions correspond to the BLAS function `xAXPY`.
--*end note]*

* *Requires:* The domain of `z` equals the domains of `x` and `y`.

* *Constraints:*

  * `x.rank()`, `y.rank()`, and `z.rank()` are all the same.

  * `x.rank()` is less than 3.

  * For `i...` in the domain of `x`, `y`, and `z`, the expression
    `z(i...) = x(i...) + y(i...)` is well formed.

* *Effects*: Compute the elementwise sum z = x + y.

#### Non-conjugated inner product of two vectors

```c++
template<class in_vector_1_t,
         class in_vector_2_t,
         class Scalar>
void dotu(in_vector_1_t v1,
          in_vector_2_t v2,
          Scalar& result);
template<class ExecutionPolicy,
         class in_vector_1_t,
         class in_vector_2_t,
         class Scalar>
void dotu(ExecutionPolicy&& exec,
          in_vector_1_t v1,
          in_vector_2_t v2,
          Scalar& result);
```

*[Note:* These functions correspond to the BLAS functions `xDOT` (for
real element types) and `xDOTU` (for complex element types).  --*end
note]*

* *Constraints:* For all `i` in the domain of `v1` and `v2`,
  the expression `result += v1(i)*v2(i)` is well formed.

* *Requires:* `v1` and `v2` have the same domain.

* *Effects:* Assigns to `result` the sum of the products of
  corresponding entries of `v1` and `v2`.

* *Remarks:* If `in_vector_t::element_type` and `Scalar` are both
  floating-point types or complex versions thereof, and if `Scalar`
  has higher precision than `in_vector_type::element_type`, then
  implementations will use `Scalar`'s precision or greater for
  intermediate terms in the sum.

#### Conjugated inner product of two vectors

```c++
template<class in_vector_1_t,
         class in_vector_2_t,
         class Scalar>
void dotc(in_vector_1_t v1,
          in_vector_2_t v2,
          Scalar& result);
template<class ExecutionPolicy,
         class in_vector_1_t,
         class in_vector_2_t,
         class Scalar>
void dotc(ExecutionPolicy&& exec,
          in_vector_1_t v1,
          in_vector_2_t v2,
          Scalar& result);

template<class in_vector_1_t,
         class in_vector_2_t,
         class Scalar>
void dot(in_vector_1_t v1,
         in_vector_2_t v2,
         Scalar& result);
template<class ExecutionPolicy,
         class in_vector_1_t,
         class in_vector_2_t,
         class Scalar>
void dot(ExecutionPolicy&& exec,
         in_vector_1_t v1,
         in_vector_2_t v2,
         Scalar& result);
```

*[Note:* These functions correspond to the BLAS functions `xDOT` (for
real element types) and `xDOTC` (for complex element types).  The
`dot` functions do the same thing as their `dotc` counterparts, and
exist so that users who attempt to write a generic code will get
reasonable default behavior for complex element types. --*end note]*

* *Constraints:*

  * If `in_vector_1_t::element_type` is `complex<T>` for some `T`,
    then for all `i` in the domain of `v1` and `v2`, the expression
    `result += conj(v1(i))*v2(i)` is well formed.

  * Otherwise, for all `i` in the domain of `v1`, the expression
    `result += v1(i)*v2(i)` is well formed.

* *Requires:* `v1` and `v2` have the same domain.

* *Effects:*

  * If `in_vector_1_t::element_type` is `complex<T>` for some `T`,
    then assigns to `result` the sum of the products of corresponding
    entries of `conjugate_view(v1)` and `v2`.

  * Otherwise, assigns to `result` the sum of the products of
    corresponding entries of `v1` and `v2`.

* *Remarks:* If `in_vector_t::element_type` and `Scalar` are both
  floating-point types or complex versions thereof, and if `Scalar`
  has higher precision than `in_vector_type::element_type`, then
  implementations will use `Scalar`'s precision or greater for
  intermediate terms in the sum.

#### Euclidean (2) norm of a vector

```c++
template<class in_vector_t,
         class Scalar>
void vector_norm2(in_vector_t v,
                  Scalar& result);

template<class ExecutionPolicy,
         class in_vector_t,
         class Scalar>
void vector_norm2(ExecutionPolicy&& exec,
                  in_vector_t v,
                  Scalar& result);
```

*[Note:* These functions correspond to the BLAS function `xNRM2`.
--*end note]*

* *Constraints:* For all `i` in the domain of `v1` and `v2`, the
  expressions `result += abs(v(i))*abs(v(i))` and `sqrt(result)` are
  well formed.  *[Note:* This does not suggest a recommended
  implementation for floating-point types.  See *Remarks*
  below. --*end note]*

* *Effects:* Assigns to `result` the Euclidean (2) norm of the
  vector `v`.

* *Remarks:*

  1. If `in_vector_t::element_type` and `Scalar` are both
     floating-point types or complex versions thereof, and if `Scalar`
     has higher precision than `in_vector_type::element_type`, then
     implementations will use `Scalar`'s precision or greater for
     intermediate terms in the sum.

  2. Let `E` be `in_vector_t::element_type`.  If

     * `E` is `float`, `double`, `long double`, `complex<float>`,
       `complex<double>`, or `complex<long double>`;

     * `Scalar` is `E` or larger in the above list of types; and

     * `numeric_limits<E>::is_iec559` is `true`;

     then implementations compute without undue overflow or
     underflow at intermediate stages of the computation.

*[Note:* The intent of the second point of *Remarks* is that
implementations generalize the guarantees of `hypot` regarding
overflow and underflow.  This excludes naive implementations.
--*end note]*

#### Sum of absolute values

```c++
template<class in_vector_t, class Scalar>
void abs_sum(in_vector_t v,
             Scalar& result);

template<class ExecutionPolicy, class in_vector_t, class Scalar>
void abs_sum(ExecutionPolicy&& exec,
             in_vector_t v,
             Scalar& result);
```

*[Note:* This function corresponds to the BLAS functions `SASUM`,
`DASUM`, `CSASUM`, and `DZASUM`. --*end note]*

* *Constraints:*

  * If `in_vector_t::element_type` is `complex<T>` for some `T`, then
    for all `i` in the domain of `v`, the expression `result +=
    real(v(i)) + imag(v(i))` is well formed.

  * Else, for all `i` in the domain of `v`, the expression
    `result += abs(v(i))` is well formed.

* *Effects:*

  * If `in_vector_t::element_type` is `complex<T>` for some `T`, then
    assigns to `result` the sum of absolute values of the real and
    imaginary components of the elements of the vector `v`.

  * Else, assigns to `result` the sum of absolute values of the
    elements of the vector `v`.

* *Remarks:*

  * If `in_vector_t::element_type` and `Scalar` are both
    floating-point types or complex versions thereof, and if `Scalar`
    has higher precision than `in_vector_type::element_type`, then
    implementations will use `Scalar`'s precision or greater for
    intermediate terms in the sum.

#### Index of maximum absolute value of vector elements

```c++
template<class in_vector_t>
void idx_abs_max(in_vector_t v,
                 ptrdiff_t& result);

// Index of maximum absolute value [iamax]
template<class ExecutionPolicy,
         class in_vector_t>
void idx_abs_max(ExecutionPolicy&& exec,
                 in_vector_t v,
                 ptrdiff_t& result);
```

*[Note:* These functions correspond to the BLAS function `IxAMAX`.
--*end note]*

* *Constraints:* For `i` and `j` in the domain of `v`, the expression
  `abs(v(i)) < abs(v(j))` is well formed.

* *Effects:* Assigns to `result` the index (in the domain of `v`) of
  the first element of `v` having largest absolute value.  If `v` has
  zero elements, then assigns `-1` to `result`.

## BLAS 2 functions

### Matrix-vector product

#### Overwriting matrix-vector product

```c++
template<class ExecutionPolicy, class in_vector_t,
         class in_matrix_t, class out_vector_t>
void matrix_vector_product(in_matrix_t A,
                           in_vector_t x,
                           out_vector_t y);

template<class ExecutionPolicy, class in_vector_t,
         class in_matrix_t, class out_vector_t>
void matrix_vector_product(ExecutionPolicy&& exec,
                           in_matrix_t A,
                           in_vector_t x,
                           out_vector_t y);
```

*[Note:* These functions correspond to the BLAS functions `xGEMV`,
`xGBMV`, `xHEMV`, `xHBMV`, `xHPMV`, `xSYMV`, `xSBMV`, `xSPMV`, and
`xTRMV`. --*end note]*

* *Requires:* If `i,j` is in the domain of `A`, then
  `i` is in the domain of `y` and `j` is in the domain of `x`.
  *[Note:* The converse need not be true. --*end note]*

* *Constraints:*

  * `A.rank()` equals 2, `x.rank()` equals 1, and
    `y.rank()` equals 1.

  * For `i,j` in the domain of `A`, the expression
    `y(i) += A(i,j)*x(j)` is well formed.

* *Effects:* Assign to the elements of `y` the
  product of the matrix `A` with the vector `x`.

#### Updating matrix-vector product

```c++
template<class ExecutionPolicy, class in_vector_1_t,
         class in_matrix_t, class in_vector_2_t,
         class out_vector_t>
void matrix_vector_product(in_matrix_t A,
                           in_vector_1_t x,
                           in_vector_2_t y,
                           out_vector_t z);

template<class ExecutionPolicy, class in_vector_1_t,
         class in_matrix_t, class in_vector_2_t,
         class out_vector_t>
void matrix_vector_product(ExecutionPolicy&& exec,
                           in_matrix_t A,
                           in_vector_1_t x,
                           in_vector_2_t y,
                           out_vector_t z);
```

*[Note:* These functions correspond to the BLAS functions `xGEMV`,
`xGBMV`, `xHEMV`, `xHBMV`, `xHPMV`, `xSYMV`, `xSBMV`, `xSPMV`, and
`xTRMV`. --*end note]*

* *Requires:*

  * If `i,j` is in the domain of `A`, then `i` is in the
    domain of `y`, and `j` is in the domain of `x`.

  * `y` and `z` have the same domain.

* *Constraints:*

  * `A.rank()` equals 2, `x.rank()` equals 1, and
    `y.rank()` equals 1.

  * For `i,j` in the domain of `A`, the expression
    `z(i) = y(i) + A(i,j)*x(j)` is well formed.

* *Effects:* Assigns to the elements of `z` the elementwise sum of
  `y`, with the vector resulting from product of the matrix `A` with
  the vector `x`.

### Solve a triangular linear system

```c++
template<class in_matrix_t,
         class in_object_t,
         class out_object_t>
void matrix_triangular_solve(in_matrix_t A,
                             in_object_t b,
                             out_object_t x);

template<class ExecutionPolicy,
         class in_matrix_t,
         class in_object_t,
         class out_object_t>
void matrix_triangular_solve(ExecutionPolicy&& exec,
                             in_matrix_t A,
                             in_object_t b,
                             out_object_t x);
```

*[Note:* These functions correspond to the BLAS functions `xTRSV`,
`xTBSV`, `xTPSV`, and `xTRSM`. --*end note]*

* *Requires:*

  * If `b.rank()` and `x.rank()` equal 1, then if `i,j` is in the
    domain of `A`, then `i` is in the domain of `x` and `j` is
    in the domain of `b`.

  * If `b.rank()` and `x.rank()` equal 2, then `x.extent(1)` equals
    `b.extent(1)`.

  * If `b.rank()` and `x.rank()` equal 2 and `x.extent(1) != 0`, then
    if `i,j` is in the domain of `A`, then there exists `k`
    such that `i,k` is in the domain of `x` and `j,k` is in the domain
    of `b`.

* *Constraints:*

  * `A.rank()` equals 2.

  * `b.rank()` equals 1 or 2.

  * `x.rank()` equals `b.rank()`.

  * `A` has triangular layout.

  * If `b.rank()` equals 1:

    * If `r` is in the domain of `x` and `b`, then the expression
      `x(r) = y(r)` is well formed.

    * If `A` has implicit unit diagonal layout and `r` is in the
      domain of `x`, then the expression `x(r) -= A(r,c)*x(c)` is well
      formed.

  * If `b.rank()` equals 2:

    * If `r,j` is in the domain of `x` and `b`, then the expression
      `x(r,j) = y(r,j)` is well formed.

    * If `A` has implicit unit diagonal layout and `r,j` and `c,j` are
      in the domain of `x`, then the expression `x(r,j) -=
      A(r,c)*x(c,j)` is well formed.

* *Effects:* Assigns to the elements of `x` the result of solving the
  triangular linear system(s) Ax=b.

### Rank-1 update of a matrix

#### Nonsymmetric non-conjugated rank-1 update

(FIXME (mfh 09 Jun 2019) Consider replacing this with
`matrix_product`, where users apply `transpose_view` or
`conjugate_transpose_view` to `y`.)

```c++
template<class in_vector_1_t,
         class in_vector_2_t,
         class inout_matrix_t>
void rank_1_update_u(in_vector_1_t x,
                     in_vector_2_t y,
                     inout_matrix_t A);

template<class ExecutionPolicy,
         class in_vector_1_t,
         class in_vector_2_t,
         class inout_matrix_t>
void rank_1_update_u(ExecutionPolicy&& exec,
                     in_vector_1_t x,
                     in_vector_2_t y,
                     inout_matrix_t A);
```

*[Note:* This function corresponds to the BLAS functions `xGER` and
`xGERU`. --*end note]*

* *Requires:* If `i,j` is in the domain of `A`, then
  `i` is in the domain of `x` and `j` is in the domain of `y`.
  *[Note:* The converse need not be true. --*end note]*

* *Constraints:*

  * `A.rank()` equals 2, `x.rank()` equals 1, and
    `y.rank()` equals 1.

  * For `i,j` in the domain of `A`, the expression
    `A(i,j) += x(i)*y(j)` is well formed.

  * `A` does not have a symmetric or Hermitian layout.

* *Effects:* Assigns to `A` on output the sum of `A` on input, and the
  (outer) product of `x` and the (non-conjugated) transpose of `y`.

#### Nonsymmetric conjugated rank-1 update

```c++
template<class in_vector_1_t,
         class in_vector_2_t,
         class inout_matrix_t>
void rank_1_update_c(in_vector_1_t x,
                     in_vector_2_t y,
                     inout_matrix_t A);

template<class ExecutionPolicy,
         class in_vector_1_t,
         class in_vector_2_t,
         class inout_matrix_t>
void rank_1_update_c(ExecutionPolicy&& exec,
                     in_vector_1_t x,
                     in_vector_2_t y,
                     inout_matrix_t A);
```

*[Note:* This function corresponds to the BLAS functions `xGER` and
`xGERC`. --*end note]*

* *Requires:* If `i,j` is in the domain of `A`, then
  `i` is in the domain of `x` and `j` is in the domain of `y`.
  *[Note:* The converse need not be true. --*end note]*

* *Constraints:*

  * `A.rank()` equals 2, `x.rank()` equals 1, and
    `y.rank()` equals 1.

  * If `in_vector_2_t::element_type` is `complex<T>` for some `T`,
    then for `i,j` in the domain of `A`, the expression `A(i,j) +=
    x(i)*conj(y(j))` is well formed.

  * Otherwise, for `i,j` in the domain of `A`, the expression `A(i,j)
    += x(i)*y(j)` is well formed.

  * `A` does not have a symmetric or Hermitian layout.

* *Effects:* Assigns to `A` on output the sum of `A` on input, and the
  (outer) product of `x` and the conjugate transpose of `y`.

#### Rank-1 update of a Symmetric or Hermitian matrix

```c++
template<class in_vector_1_t,
         class in_vector_2_t,
         class inout_matrix_t>
void rank_1_update(in_vector_1_t x,
                   inout_matrix_t A);

template<class ExecutionPolicy,
         class in_vector_1_t,
         class in_vector_2_t,
         class out_matrix_t>
void rank_1_update(ExecutionPolicy&& exec,
                   in_vector_1_t x,
                   inout_matrix_t A);
```

*[Note:* These functions correspond to the BLAS functions `xHER`,
`xHPR`, `xSYR`, and `xSPR`. --*end note]*

* *Requires:* If `i,j` is in the domain of `A`, then `i` and
  `j` are in the domain of `x`.  *[Note:* The converse need not be
  true. --*end note]*

* *Constraints:*

  * `A.rank()` equals 2 and `x.rank()` equals 1.

  * `A` has a symmetric or Hermitian layout.

  * If `A` has a symmetric layout, then for `i,j` in the domain of
    `A`, the expression `A(i,j) += x(i)*x(j)` is well formed.

  * If `A` has a Hermitian layout, then for `i,j` in the domain of
    `A`, the expression `A(i,j) += x(i)*conj(x(j))` is well formed.

* *Effects:*

  * If `A` has a symmetric layout, then assigns to `A` on output the
    sum of `A` on input, and the (outer) product of `x` and the
    (non-conjugated) transpose of `x`.

  * Else, if `A` has a Hermitian layout, then assigns to `A` on output
    the sum of `A` on input, and the (outer) product of `x` and the
    conjugate transpose of `x`.

*[Note:* The layout of `A` determines whether the outer product uses
the conjugate transpose or the (non-conjugated) transpose.  Thus,
unlike the `xDOTU` and `xDOTC` case, we do not need separate
functions. --*end note]*

### Rank-2 update of a Symmetric or Hermitian matrix

```c++
template<class in_vector_1_t,
         class in_vector_2_t,
         class inout_matrix_t>
void rank_2_update(in_vector_1_t x,
                   in_vector_2_t y,
                   inout_matrix_t A);

template<class ExecutionPolicy,
         class in_vector_1_t,
         class in_vector_2_t,
         class inout_matrix_t>
void rank_2_update(ExecutionPolicy&& exec,
                   in_vector_1_t x,
                   in_vector_2_t y,
                   inout_matrix_t A);
```

*[Note:* These functions correspond to the BLAS functions `xHER2`,
`xHPR2`, `xSYR2`, and `xSPR2`. --*end note]*

* *Requires:* If `i,j` is in the domain of `A`, then
  `i` and `j` are in the domain of `x` and `y`.
  *[Note:* The converse need not be true. --*end note]*

* *Constraints:*

  * `A.rank()` equals 2, `x.rank()` equals 1, and
    `y.rank()` equals 1.

  * `A` has a symmetric or Hermitian layout.

  * If `A` has a symmetric layout, then for `i,j` in the domain of
    `A`, the expression `A(i,j) += x(i)*y(j) + y(i)*x(j)` is well
    formed.

  * If `A` has a Hermitian layout, then for `i,j` in the domain of
    `A`, the expression `A(i,j) += x(i)*conj(y(j)) + y(i)*conj(x(j))`
    is well formed.

* *Effects:*

  * If `A` has a symmetric layout, then assigns to `A` on output the
    sum of `A` on input, the (outer) product of `x` and the
    (non-conjugated) transpose of `y`, and the (outer) product of `y`
    and the (non-conjugated) transpose of `x`.

  * Else, if `A` has a Hermitian layout, then assigns to `A` on output
    the sum of `A` on input, the (outer) product of `x` and the
    conjugate transpose of `y`, and the (outer) product of `y` and the
    conjugate transpose of `x`.

*[Note:* The layout of `A` determines whether the outer product uses
the conjugate transpose or the (non-conjugated) transpose.  Thus,
unlike the `xDOTU` and `xDOTC` case, we do not need separate
functions. --*end note]*

## BLAS 3 functions

### Matrix-matrix product

#### Overwriting matrix-matrix product

```c++
template<class in_matrix_1_t,
         class in_matrix_2_t,
         class out_matrix_t>
void matrix_product(in_matrix_1_t A,
                    in_matrix_2_t B,
                    out_matrix_t C);

template<class ExecutionPolicy,
         class in_matrix_1_t,
         class in_matrix_2_t,
         class out_matrix_t>
void matrix_product(ExecutionPolicy&& exec,
                    in_matrix_1_t A,
                    in_matrix_2_t B,
                    out_matrix_t C);
```

*[Note:* These functions correspond to the BLAS functions `xGEMM`,
`xSYMM`, `xHEMM`, and `xTRMM`. --*end note]*

* *Requires:*

  * If `i,j` is in the domain of `C`, then there exists `k`
    such that `i,k` is in the domain of `A`, and `k,j` is in the
    domain of `B`.

* *Constraints:*

  * `A.rank()` equals 2, `B.rank()` equals 2, and
    `C.rank()` equals 2.

  * For `i,j` in the domain of `C`, `i,k` in the domain of `A`, and
    `k,j` in the domain of `B`, the expression `C(i,j) +=
    A(i,k)*B(k,j)` is well formed.

* *Effects:* Assigns to the elements of the matrix `C` the product of
  the matrices `A` and `B`.

*[Note:* Implementers may wish to give users control over whether the
matrix product uses a Strassen-type algorithm.  One way they could do
so is by letting users annotate the execution policy argument, for
example, using the [Properties mechanism in P1393](wg21.link/p1393).
--*end note]*

#### Updating matrix-matrix product

```c++
// matrix product update: C = A * B + E [gemm, symm, hemm, trmm]

template<class in_matrix_1_t,
         class in_matrix_2_t,
         class in_matrix_3_t,
         class out_matrix_t>
void matrix_product(in_matrix_1_t A,
                    in_matrix_2_t B,
                    in_matrix_3_t E,
                    out_matrix_t C);

template<class ExecutionPolicy,
         class in_matrix_1_t,
         class in_matrix_2_t,
         class in_matrix_3_t,
         class out_matrix_t>
void matrix_product(ExecutionPolicy&& exec,
                    in_matrix_1_t A,
                    in_matrix_2_t B,
                    in_matrix_3_t E,
                    out_matrix_t C);
```

*[Note:* These functions correspond to the BLAS functions `xGEMM`,
`xSYMM`, `xHEMM`, and `xTRMM`. --*end note]*

* *Requires:*

  * If `i,j` is in the domain of `C`, then there exists `k`
    such that `i,k` is in the domain of `A`, and `k,j` is in the
    domain of `B`.

  * `C` and `E` have the same domain.

* *Constraints:*

  * `A.rank()` equals 2, `B.rank()` equals 2,
    `C.rank()` equals 2, and `E.rank()` equals 2.

  * For `i,j` in the domain of `C`, `i,k` in the domain of `A`, and
    `k,j` in the domain of `B`, the expression `C(i,j) += E(i,j) +
    A(i,k)*B(k,j)` is well formed.

* *Effects:* Assigns to the elements of the matrix `C` on output, the
  elementwise sum of `E` and product of the matrices `A` and `B`.

* *Remarks:* `C` and `E` may refer to the same matrix.  If so, then
  they must have the same layout.

*[Note:* Implementers may wish to give users control over whether the
matrix product uses a Strassen-type algorithm.  One way they could do
so is by letting users annotate the execution policy argument, for
example, using the [Properties mechanism in P1393](wg21.link/p1393).
--*end note]*

### Rank-k update of a Symmetric or Hermitian matrix

```c++
template<class in_matrix_t,
         class inout_matrix_t>
void rank_k_update(in_matrix_t A,
                   inout_matrix_t C);

template<class ExecutionPolicy,
         class in_matrix_t,
         class inout_matrix_t>
void rank_k_update(ExecutionPolicy&& exec,
                   in_matrix_t A,
                   inout_matrix_t C);
```

*[Note:* These functions correspond to the BLAS functions `xHERK`
and `xSYRK`. --*end note]*

* *Requires:* If `i,j` is in the domain of `C`, then there
  exists `k` such that `i,k` and `j,k` are in the domain of `A`.

* *Constraints:*

  * `A.rank()` equals 2 and `C.rank()` equals 2.

  * `C` has a symmetric or Hermitian layout.

  * If `C` has a symmetric layout, then for `i,j` in the domain of `C`
    and `i,k` and `j,k` in the domain of `A`, the expression `C(i,j)
    += A(i,k)*A(j,k)` is well formed.

  * If `C` has a Hermitian layout, then for `i,j` in the domain of `C`
    and `i,k` and `j,k` in the domain of `A`, the expression `C(i,j)
    += A(i,k)*conj(A(j,k))` is well formed.

* *Effects*:

  * If `C` has a symmetric layout, then updates `C` with the matrix
    product of A and the non-conjugated transpose of A.

  * If `C` has a Hermitian layout, then updates `C` with the matrix
    product of A and the conjugate transpose of A.

*[Note:* Users who want to perform a rank-k update of a nonsymmetric
matrix `C` may use `matrix_product`. --*end note]*

### Rank-2k update of a Symmetric or Hermitian matrix

```c++
template<class in_matrix_1_t,
         class in_matrix_2_t,
         class inout_matrix_t>
void rank_2k_update(in_matrix_1_t A,
                    in_matrix_2_t B,
                    out_matrix_t C);

template<class ExecutionPolicy,
         class in_matrix_1_t,
         class in_matrix_2_t,
         class inout_matrix_t>
void rank_2k_update(ExecutionPolicy&& exec,
                    in_matrix_1_t A,
                    in_matrix_2_t B,
                    out_matrix_t C);
```

*[Note:* These functions correspond to the BLAS functions `xHER2K`
and `xSYR2K`. --*end note]*

* *Requires:* If `i,j` is in the domain of `C`, then there exists `k`
  such that `i,k` and `j,k` are in the domain of `A`, and `j,k` and
  `i,k` are in the domain of `B`.

* *Constraints:*

  * `A.rank()` equals 2, `B.rank()` equals 2, and
    `C.rank()` equals 2.

  * `C` has a symmetric or Hermitian layout.

  * If `C` has a symmetric layout, then for `i,j` in the domain of
    `C`, `i,k` and `k,i` in the domain of `A`, and `j,k` and `k,j` in
    the domain of `B`, the expression `C(i,j) += A(i,k)*B(j,k) +
    B(i,k)*A(j,k)` is well formed.

  * If `C` has a Hermitian layout, then for `i,j` in the domain of
    `C`, `i,k` and `k,i` in the domain of `A`, and `j,k` and `k,j` in
    the domain of `B`, the expression `C(i,j) += A(i,k)*conj(B(j,k)) +
    B(i,k)*conj(A(j,k))` is well formed.

* *Effects*:

  * If `C` has a symmetric layout, then updates `C` with the sum of
    (the matrix product of A and the non-conjugated transpose of B),
    and (the matrix product of B and the non-conjugated transpose of
    A.)

  * If `C` has a Hermitian layout, then updates `C` with the sum of
    (the matrix product of A and the conjugate transpose of B), and
    (the matrix product of B and the conjugate transpose of A.)

*[Note:* Users can achieve the effect of the `TRANS` argument of these
BLAS functions, by making `C` a `transpose_view`. --*end note]*

*[Note:* The BLAS "quick reference" has a typo; the "ALPHA" argument
of `CSYR2K` and `ZSYR2K` should not be conjugated. --*end note]*

## Examples

```c++
using vector_t = basic_mdspan<double, extents<dynamic_extent>>;
using dy_ext2_t = extents<dynamic_extent, dynamic_extent>;
using matrix_t = basic_mdspan<double, dy_ext2_t>;

// Create vectors
vector_t x = ...;
vector_t y = ...;
vector_t z = ...;

// Create matrices
matrix_t A = ...;
matrix_t B = ...;
matrix_t C = ...;

// z = 2.0 * x + y;
linalg_add(par, scaled_view(2.0, x), y, z);
// y = 2.0 * y + z;
linalg_add(par, z, scaled_view(2.0, y), y);

// y = 3.0 * A * x;
matrix_vector_product(par, scaled_view(3.0, A), x, y);
// y = 3.0 * A * x + 2.0 * y;
matrix_vector_product(par, scaled_view(3.0, A), x,
                      scaled_view(2.0, y), y);

// y = transpose(A) * x;
matrix_vector_product(par, transpose_view(A), x, y);

// Create matrix
matrix_t D = ...;

// View D's upper triangle as a representation of a
// symmetric matrix, and perform a matrix-vector product
// with the resulting symmetric matrix.
matrix_vector_product(par, symmetric_view(A, upper_triangle),
                      x, y);
```

## Batched BLAS

Batched BLAS will be handled by simply taking arguments of higher rank
than specified in BLAS. A nonunique broadcast layout can be used to
use the same lower rank object in the operation for each of the
batched operations. The first dimension of the objects is the number
of operations performed. `extent(0)` of each argument must be equal.

## Options and votes

This is a preliminary proposal.  Besides the usual bikeshedding, we
also want to present more broad options for voting.  Here is a list;
we will explain each option below.

1. Omit vector-vector operations in favor of existing C++ Standard algorithms?
2. Retain "view" functions (modest expression templates)?
3. `triangular_view` redefines `mdspan` "domain"; what to do?
4. Combine functions that differ only by rank of arguments?
5. Prefer overloads to different function names?
6. Retain existing BLAS behavior for scalar multipliers?

### Omit vector-vector operations in favor of existing C++ Standard algorithms?

Annex C of the BLAS Standard offers a "Thin BLAS" option for Fortran
95, where the language itself could replace many BLAS operations.
Fortran 95 comes with dot products and other vector operations built
in, so the "Thin BLAS" only retains four "BLAS 1" functions: `SWAP`,
`ROT`, `NRM2`, and `ROTG`.  By analogy with the "Thin BLAS," we could
offer a much more limited set of functions, by relying on
functionality either already in C++, or likely to enter C++ soon.  For
example, if we defined iterators for rank-1 `mdspan` and `mdarray`, we
could rely on `transform` and `transform_reduce` for most of the
vector-vector operations.

Matrix-vector ("BLAS 2") and matrix-matrix ("BLAS 3") operations
require iteration over two or three dimensions, and are thus less
natural to implement using `transform` or `transform_reduce`.  They
are also more likely to get performance benefits from specialized
implementations.

Here are arguments for this approach:

1. It reduces the number of new functions.

2. It focuses on "performance primitives" most likely to benefit from
   vendor optimization.

3. If a hypothetical "parallel Ranges" enters the Standard, it could
   cover many of the use cases for parallel vector-vector operations.

Here are arguments against this approach:

1. Some of our "vector-vector" operations are actually "object-object"
   operations that work for matrices too.  Replacing those with
   existing Standard algorithms would call for iterators on matrices.

2. It takes some effort to implement correct and accurate vector
   norms.  Compare to [POSIX requirements for
   `hypot`](http://pubs.opengroup.org/onlinepubs/9699919799/functions/hypot.html).
   If `hypot` is in the Standard, then perhaps norms should also be.

3. It's easier to apply hardware-specific optimizations to
   vector-vector operations if they are exposed as such.

4. Exposing a full linear algebra interface would give implementers
   the option to use extended-precision or even reproducible
   floating-point arithmetic for all linear algebra operations.  This
   can be useful for debugging complicated algorithms.  Compare to
   "checked iterator" debug options for the C++ Standard Library.

5. It helps to have linear algebra names for linear algebra
   operations.  For example, `string` still exists, even though much
   of its functionality is covered by `vector<char>`.

### Retain "view" functions (modest expression templates)?

The four functions `scaled_view`, `conjugate_view`, `transpose_view`,
and `conjugate_transpose_view` use `mdspan` accessors to implement a
modest form of expression templates.  We say "modest" because they
mitigate several known issues with expression templates:

1. They do not introduce ADL-accessible arithmetic operators.

2. If used with `mdspan`, then they would not introduce any more
   dangling references than `span` (**[views.span]**) would introduce.

3. Their intended use case, as temporary "decorators" for function
   arguments, discourages capture as `auto` (which may result in
   unexpectedly dangling references).

The functions have the following other advantages:

1. They reduce the number of linear algebra function arguments.

2. They only create read-only views.  This avoids potential confusion
   with what it means to write to, e.g., a "scaled object" (does it
   multiply the input, ignore it, or divide by it?).

3. They simplify native C++ implementations, especially for BLAS 1 and
   BLAS 2 functions that do not need complicated optimizations in
   order to get reasonable performance.

However, the functions have the following disadvantages:

1. They would be the first instance of required expression templates
   in the C++ Standard Library.  (`valarray` in **[valarray.syn]**
   permits but does not require expression templates.)

2. When applied to an `mdarray`, the functions could introduce
   dangling references.  Compare to `gslice_array` for `valarray`.

3. When applied to an `mdarray`, the functions turn the `mdarray` into
   an `mdspan`.  This could introduce (modest) overhead for very small
   vectors or matrices.

Here are the options:

1. Keep "view" functions.

2. Drop "view" functions.  Retain tags and `constexpr` tag values; use
   them as function arguments to algorithms.

### `triangular_view` redefines `mdspan` "domain"; what to do?

The `triangular_view` function returns an `basic_mdspan`.  However,
its "domain" -- the set of its valid index pairs `i,j` -- does not, in
general, include all pairs in the Cartesian product of its extents.
This differs from the definition of *domain* in
[P0009](wg21.link/p0009).  In what follows, we will call this an
*incomplete domain*.

It may help to compare this with sparse matrix data structures,
especially the class of sparse matrix data structures that forbid
"structure changes."  The sparse matrix comes with a set of
multi-indices -- the "populated elements" P -- that is a subset of the
Cartesian product of extents E.  At all multi-indices in E but not in
P, the matrix elements are zero.  Users may read from them, but not
write to them.

We could design the unpacked layout `layout_blas_triangular` in the
same way, by having its mapping and accessor return zero values for
index pairs not in the triangle.  The mapping would map index pairs
not in the triangle to some flag value that the accessor would
interpret as "return a read-only zero value."  This would take extra
machinery and overhead in the accessor (a different `reference_type`
with a branch inside), but it's possible.  Just like with
`layout_blas_symmetric`, algorithms could avoid this overhead by
extracting the "base layout" but respecting the triangular
assumptions.

The packed layouts do not have this issue.  All index pairs in the
Cartesian product of the extents map (nonuniquely) to a valid index in
the codomain.  For index pairs outside the triangle, the codomain
index is wrong, but it's still valid.

Here are options for addressing this issue.  We order them by our
decreasing preference.

1. Add a new "incomplete" category of layout mapping policy to the
   categories in P0009.  Forbid users of `layout_blas_triangular` (and
   possibly also `layout_blas_symmetric`) layouts from accessing index
   pairs outside the layout's triangle.  The layouts will not remap
   forbidden indices.

2. Add a new "incomplete" category of layout mapping policy to the
   categories in P0009.  Make `layout_blas_triangular` return
   unwriteable zero (whatever that means) for index pairs outside the
   triangle.  Algorithms may get the "base layout" to avoid index
   remapping and accessor overhead.

3. Change the algorithms to take "triangle" and "diagonal storage"
   information as separate arguments.  Do not redefine *domain*.

We do not prefer (3), because (1) and (2) open the door to future
expansion of matrix storage formats.

### Combine functions that differ only by rank of arguments?

This relates to the "thin BLAS" proposal mentioned above.  Another
part of that proposal was the elimination of separate function names,
when (the Fortran 95 equivalent of) overloads could express the same
idea.  There are two parts to this:

1. Combine functions that differ only by rank of arguments.
2. Combine functions that differ only by layout of output arguments.

We only recommend the first part.  The second part has pitfalls that
we will describe below.

As an example of the first part, the BLAS functions `xSYRK` and
`xSYR1` differ only by rank of their input arguments.  Both perform a
symmetric outer-product update of their input/output matrix argument
`C`.  `xSYRK` could implement `xSYR1` by taking an input "matrix" `A`
with a single column.  This is not necessarily the fastest way to
implement `xSYR1`.  However, since the rank of an `mdspan` or
`mdarray` is known at compile time, implementations could dispatch to
the appropriate low-level computational kernel with no run-time
overhead.  (Existing BLAS implementations do not always optimize for
the case where a "matrix" argument to a BLAS 3 function like `xGEMM`
has only one column, and thus the function could dispatch to a BLAS 2
function like `xGEMV`.)

Here are arguments for this approach:

1. It reduces the number of new functions.

2. Implementations could identify all special cases at compile time.

Here are arguments against this approach:

1. It adds special cases to implementations.

As an example of the second part, the BLAS functions `xGEMM` and
`xSYRK` differ mostly just by layout of output arguments.  `xGEMM`
computes the matrix-matrix product update `C := alpha * A * B + beta *
C`.  `xSYRK` computes the symmetric matrix-matrix product update `C :=
alpha * A * A^T + beta * C`, where `C` is assumed to be symmetric and
the algorithm only accesses either the upper or lower triangle of `C`.
We could express both algorithms in a single function `gemm`, where
`C` on input and output has the appropriate symmetric layout (see
`layout_blas_symmetric` above).  However, this approach easily leads
to unexpected behavior.  For example, what if `C` has a symmetric
layout and `A * B` is nonsymmetric, but users request to compute `C :=
A * B`?  The result `C` would be mathematically incorrect, even though
it would retain symmetry.  For this reason, we recommend against
combining functions in this way.

### Retain existing BLAS behavior for scalar multipliers?

The BLAS Standard treats zero values of `alpha` or `beta` scalar
multipliers as special short-circuiting cases.  For example, the
matrix-matrix multiply update `C := alpha * A * B + beta * C` does not
compute `A*B` if `alpha` is zero, and treats `C` as write only if
`beta` is zero.  We propose to change this behavior by always
performing the requested operation, regardless of the values of any
scalar multipliers.  This has the following advantages:

1. It removes special cases.

2. It avoids branches, which could affect performance for small
   problems.

3. It does not privilege floating-point element types.

However, it has the following disadvantages:

1. Implementations based on an existing BLAS library must
   "double-check" scaling factors.  If any is zero, the implementation
   cannot call the BLAS and must perform the scaling manually.  This
   will likely reduce performance for a case that users intend to be
   fast, unless the implementation has access to internal BLAS details
   that can skip the special cases.

2. Users may expect BLAS semantics in a library that imitates BLAS
   functionality.  These users will get unpleasantly surprising
   results (like `Inf` or `NaN` instead of zero).

We mitigate these disadvantages by offering both write-only and
read-and-write versions of algorithms like `matrix_product`, whose
BLAS versions take a `beta` argument.  In our experience using the
BLAS, users are more likely to expect that setting `beta=0` causes
write-only behavior.  Thus, if the interface suggests write-only
behavior, users are less likely to be unpleasantly surprised.