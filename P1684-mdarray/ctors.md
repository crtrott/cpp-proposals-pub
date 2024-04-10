# MDArray Ctors

The number of plausible constructors for `mdarray` is large:
- different ways of specifying layouts
  - integrals, extents, mapping
- all the ways to construct the underlying container
  - copying, moving, init value, iterators, `initializer_list`, `ranges`
- container adaptors have all of these also with an allocator

Plausible set leads to 38 ctors


```c++
  // normal set
  MDSPAN_INLINE_FUNCTION_DEFAULTED constexpr mdarray() requires(extents_type::rank_dynamic()!=0) = default;
  MDSPAN_INLINE_FUNCTION_DEFAULTED constexpr mdarray(const mdarray&) = default;
  MDSPAN_INLINE_FUNCTION_DEFAULTED constexpr mdarray(mdarray&&) = default;

  // converting
  template<class OtherElementType, class OtherExtents, class OtherLayoutPolicy, class OtherContainer>
  constexpr mdarray(const mdarray<OtherElementType, OtherExtents, OtherLayoutPolicy, OtherContainer>& other)
  template<class OtherElementType, class OtherExtents, class OtherLayoutPolicy, class OtherAccessor>
  constexpr mdarray(const mdspan<OtherElementType, OtherExtents, OtherLayoutPolicy, OtherAccessor>& other)

  // sizes/extents/mapping -- vector
  template<class... SizeTypes>
  explicit constexpr mdarray(SizeTypes... dynamic_extents);
  mdarray(const extents_type& ext);
  mdarray(const mapping_type& map);

  // extents/mapping + value -- vector
  mdarray(const extents_type& ext, element_type value);
  mdarray(const mapping_type& map, element_type value);

  // extents/mapping + container -- vector, queue, flat_map
  mdarray(const extents_type& ext, const container_type& c);
  mdarray(const mapping_type& map, const container_type& c);
  mdarray(const extents_type& ext, container_type&& c);
  mdarray(const mapping_type& map, container_type&& c);

  // extents/mapping + iterators -- vector, queue, flat_map
  template<class InputIterator>
  mdarray(const extents_type& ext, InputIterator first, InputIterator last);
  template<class InputIterator>
  mdarray(const mapping_type& map, InputIterator first, InputIterator last);
  
  // extents/mapping + range -- vector, queue, flat_map
  template<container-compatible-range<value_type> R>
  mdarray(const extents_type& ext, R&& r);
  template<container-compatible-range<value_type> R>
  mdarray(const mapping_type& map, R&& r);

  // extents/mapping + initializer_list -- vector, flat_map
  mdarray(const extents_type& ext, initializer_list<value_type> il)
  mdarray(const mapping_type& ext, initializer_list<value_type> il)

  // With Allocator
  template<class Allocator>
  constexpr mdarray(const mdarray&);
  template<class Allocator>
  constexpr mdarray(mdarray&&);

  // converting
  template<class OtherElementType, class OtherExtents, class OtherLayoutPolicy, class OtherContainer, class Allocator>
  constexpr mdarray(const mdarray<OtherElementType, OtherExtents, OtherLayoutPolicy, OtherContainer>& other, const Allocator& alloc)
  template<class OtherElementType, class OtherExtents, class OtherLayoutPolicy, class OtherAccessor, class Allocator>
  constexpr mdarray(const mdspan<OtherElementType, OtherExtents, OtherLayoutPolicy, OtherAccessor>& other, const Allocator& alloc)

  // sizes/extents/mapping -- vector
  template<class Allocator>
  mdarray(const extents_type& ext, const Allocator& alloc);
  template<class Allocator>
  mdarray(const mapping_type& map, const Allocator& alloc);

  // extents/mapping + value -- vector
  template<class Allocator>
  mdarray(const extents_type& ext, element_type value, const Allocator& alloc);
  template<class Allocator>
  mdarray(const mapping_type& map, element_type value, const Allocator& alloc);

  // extents/mapping + container -- vector, queue, flat_map
  template<class Allocator>
  mdarray(const extents_type& ext, const container_type& c, const Allocator& alloc);
  template<class Allocator>
  mdarray(const mapping_type& map, const container_type& c, const Allocator& alloc);
  template<class Allocator>
  mdarray(const extents_type& ext, container_type&& c, const Allocator& alloc);
  template<class Allocator>
  mdarray(const mapping_type& map, container_type&& c, const Allocator& alloc);

  // extents/mapping + iterators -- vector, queue, flat_map
  template<class InputIterator, class Allocator>
  mdarray(const extents_type& ext, InputIterator first, InputIterator last, const Allocator& alloc);
  template<class InputIterator, class Allocator>
  mdarray(const mapping_type& map, InputIterator first, InputIterator last, const Allocator& alloc);
  
  // extents/mapping + range -- vector, queue, flat_map
  template<container-compatible-range<value_type> R, class Allocator>
  mdarray(const extents_type& ext, R&& r, const Allocator& alloc);
  template<container-compatible-range<value_type> R, class Allocator>
  mdarray(const mapping_type& map, R&& r, const Allocator& alloc);

  // extents/mapping + initializer_list -- vector, flat_map
  template<class Allocator>
  mdarray(const extents_type& ext, initializer_list<value_type> il, const Allocator& alloc)
  template<class Allocator>
  mdarray(const mapping_type& ext, initializer_list<value_type> il, const Allocator& alloc)
```


