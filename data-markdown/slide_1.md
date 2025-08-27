## Test
- Point 1
- Point 2
- Point 3

+++

## Goals
- Point 1 <!-- .element: class="fragment" -->
- Point 2 <!-- .element: class="fragment" -->
- Point 3 <!-- .element: class="fragment" -->

+++

#### Code 1
```python
class ttl:
    def __init__(
        self,
        uri: _TripleSubject,
        *predicate_object_pairs: tuple[
            URIRef,
            _TripleObject
            | list
            | Iterator
            | Self
            | str
            | tuple[_TripleObject | str, ...],
        ],
        graph: Graph | None = None,
    ) -> None:
        self.uri = uri
        self.predicate_object_pairs = predicate_object_pairs
        self.graph = Graph() if graph is None else deepcopy(graph)
        self._iter = iter(self)
		# ...
```

+++

#### Code 2
```python
	# class ttl
    def __iter__(self) -> Iterator[_Triple]:
        """Generate an iterator of tuple-based triple representations."""
        for pred, obj in self.predicate_object_pairs:
            match obj:
                case ttl():
                    yield (self.uri, pred, obj.uri)
                    yield from obj
                case list() | Iterator():
                    _b = BNode()
                    yield (self.uri, pred, _b)
                    yield from ttl(_b, *obj)
                case tuple():
                    _object_list = zip(repeat(pred), obj)
                    yield from ttl(self.uri, *_object_list)
                case obj if isinstance(obj, _TripleObject):
                    yield (self.uri, pred, obj)
                case str():
                    yield (self.uri, pred, Literal(obj))
                case _:
                    raise Exception("This should never happen.")
```
