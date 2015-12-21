# GLnexus
**From DNAnexus R&D: a scalable datastore for population genome sequencing, with on-demand joint genotyping.**
(GL, genotype likelihood)

This is an early-stage R&D project we're developing openly. The code doesn't yet do anything useful! There's a [wiki project roadmap](https://github.com/dnanexus-rnd/GLnexus/wiki), which should be read in the spirit of "plans are worthless, but planning is indispensable."

### Build & run tests

<a href="https://travis-ci.org/dnanexus-rnd/GLnexus"><img src="https://travis-ci.org/dnanexus-rnd/GLnexus.svg?branch=master"/></a> [![Coverage Status](https://coveralls.io/repos/dnanexus-rnd/GLnexus/badge.svg?branch=master&service=github)](https://coveralls.io/github/dnanexus-rnd/GLnexus?branch=master)

First [install gcc 4.9](http://askubuntu.com/a/581497), `cmake` `libjemalloc-dev` `libboost-dev` `libzip-dev` `libsnappy-dev` `liblz4-dev` `libbz2-dev`. Then:

```
cmake -Dtest=ON . && make && ./unit_tests
```

Other dependencies (should be set up automatically by CMake):
* [htslib](https://github.com/samtools/htslib)
* [rocksdb](https://github.com/facebook/rocksdb)
* [yaml-cpp](https://github.com/jbeder/yaml-cpp)
* [CTPL](https://github.com/vit-vit/CTPL)
* [fcmm](https://github.com/giacomodrago/fcmm)
* [Catch](https://github.com/philsquared/Catch) test framework

### Developer documentation

Evolving developer documentation can be found on the [project github page](http://dnanexus-rnd.github.io/GLnexus/index.html).

### Coding conventions

* C++14 - take advantage of [the goodies](http://shop.oreilly.com/product/0636920033707.do)
* Use smart pointers to avoid passing resources needing manual deallocation across function/class boundaries
* Prefer references over pointers when they shouldn't be null nor change ever.
* Avoid exceptions; prefer returning a `Status`, defined early in [types.h](https://github.com/dnanexus-rnd/GLnexus/blob/master/include/types.h)
 * nb the frequently-used convenience macro `S()` defined just below `Status`
* Avoid public constructors with nontrivial bodies; prefer static initializer function returning `Status`
* Avoid elaborate templated class hierarchies


### Performance profiling

The code has some hooks for performance profiling using
[`perf`](https://en.wikipedia.org/wiki/Perf_(Linux)) and
[FlameGraph](http://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html). To
make the profile as detailed as possible, the code should be compiled in
`Profile` mode like so:

```
cmake -Dtest=ON -DCMAKE_BUILD_TYPE=Profile . && make
```

To profile performance within the DNAnexus applet, change `CMAKE_BUILD_TYPE`
to `Profile` in `cli/dxapplet/Makefile`, rebuild the applet, and run it as
usual plus `-i enable_perf=true`. This produces an output file
```genotype.stacks``` containing sampling observation counts for common call
stacks. To generate an SVG visualization with FlameGraph:

```
git clone https://github.com/brendangregg/FlameGraph
FlameGraph/flamegraph.pl <(grep glnexus genotype.stacks) > genotype.svg
```
