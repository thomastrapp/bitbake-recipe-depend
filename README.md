BitBake Recipe Dependencies
===========================

List recipes from Bitbake's `task-depends.dot` and the dependencies
between these recipes.

Note: [openembedded-core](https://github.com/openembedded/openembedded-core) ships the script [oe-depends-dot](https://github.com/openembedded/openembedded-core/blob/0441b53d55a919b5ac42e997f4092053b017b553/scripts/oe-depends-dot), which solves the same problem as this project. But that script does not work anymore, because the output of `bitbake -g` has changed.

Example:

```shell
# generate `task-depends.dot`
bitbake -g foo

# list recipes with at least one task that recipe "curl" depends on directly
bb-recipe-depend task-depends.dot curl

# list recipes that have at least one task that directly depends on a task
# from recipe "curl":
bb-recipe-depend task-depends.dot -r curl

# list recipes that have at least one task that transitively depends on a task
# from recipe "curl":
bb-recipe-depend task-depends.dot -tr curl

# list recipes with at least one task that recipe "curl" depends on, and list
# all their dependencies
bb-recipe-depend task-depends.dot -t curl
```

Options:

```
./bb-recipe-depend - List dependencies between BitBake recipes.

Usage:
  ./bb-recipe-depend [options] <task-depends.dot>
      List all recipes

  ./bb-recipe-depend [options] <task-depends.dot> <recipe_name>
      List dependencies of a specific recipe

Options:
  --task-depends-dot <file> The task-depends.dot file generated by `bitbake -g`
  --recipe <recipe_name>    Select a recipe
  -d [ --depends ]          List dependencies of recipe (default if recipe 
                            given)
  -r [ --rdepends ]         List reverse dependencies of recipe
  -t [ --transitive ]       List all transitive dependencies of the given 
                            recipe
  -h [ --help ]             Print this help message
  -V [ --version ]          Print version
```

Dependencies:

* A compiler with `C++17` support, such as `g++` ≥7 or  `clang++` ≥5
* `cmake` ≥3.10
* `libboost-dev`
* `libboost-graph-dev`
* `libboost-program-options-dev`

Build and install:

```
cd build
cmake ..
make
make install
```

How it works:

* `bitbake -g` generates a file called `task-depends.dot` containing a graph described with the [DOT language](https://en.wikipedia.org/wiki/DOT_(graph_description_language)).
* This graph contains an edge for each dependency between [tasks](https://docs.yoctoproject.org/ref-manual/tasks.html) of the [recipes](https://docs.yoctoproject.org/dev-manual/common-tasks.html#writing-a-new-recipe) contained in a build.
* `bitbake-recipe-depend` [parses](https://github.com/thomastrapp/bitbake-recipe-depend/blob/master/ragel/dot-machine.rl) the `taks-depends.dot` file to build a [graph](https://github.com/thomastrapp/bitbake-recipe-depend/blob/master/bbrd/bbrd/DependencyGraph.h) of the [dependencies](https://github.com/thomastrapp/bitbake-recipe-depend/blob/master/bbrd/bbrd/Dependencies.h) between recipes.
* Transitive dependencies are resolved by using a breadth first search while recording the vertices (i.e. recipes).
* Direct dependencies are resolved by simply recording the adjacent vertices of the directed graph.
* The option `--rdepends` transforms the graph with [boost::reverse\_graph](https://www.boost.org/doc/libs/1_77_0/libs/graph/doc/reverse_graph.html).
* Note that `bitbake-recipe-depend` cannot parse arbitrary DOT. Only the output file of `bitbake -g` is supported.
