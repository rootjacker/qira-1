## 9/8/2014 -- The IDA Proxy ##
  * <strike>Support drawing CFGs of functions</strike>
  * <strike>Support renaming and adding comments</strike>
  * <strike>The structured instruction project, registers should be colored and names should work</strike>
    * We still need a better way to do this
  * <strike>Add flat mode support.</strike>
    * Not the best quality
  * Support full proxying of the IDA server, and loading/saving of idb. (so make 'c' work)

## 9/13/2014 -- Release v1.1 ##
  * <strike>A release has to happen soon, it's been a while</strike>
    * Static stuff can be disabled, but names/comments, reg highlighting, etc must work.
  * <strike>Fix the name caching problem?</strike>
    * Rethink the tags cache in general, how large is it?
  * <strike>Come up with a testing plan</strike>, and consider setting up a build/packaging server on js.

## 9/15/2014 -- Making the static view good ##
  * <strike>Fix the graph scroller thing, push this to history or like the function position cache</strike>
  * Fix the perf issues with redrawing the graph every time
  * Get the data and strings from IDA.
  * Stack frame analysis?
  * Show the DWARF data

## 9/22/2014 -- Moving away from IDA ##
  * Consider the xref problem, I think staticxrefs and dynamicxrefs should be different
  * Add a static view of the memory that isn't hacks.
    * Sadly, this involves loaders, and I don't see a way around that. Find libraries.
  * Get the BAP semantics in a smarter way, spawning a BAP process for each is unacceptable.
  * Add a recursive descent parser using BAP semantics. Extract the start of functions from dynamics.
  * Add string xref detection with a simple heuristic. Support searching over the memory space given a changenumber, even if it's slow.

# QIRA Projects #
  * MIPS support in the frontend -- 1-3 hours
  * Having the qiradb load from BAP traces. -- 5-10 hours
  * Program slicing, over a bap trace, need semantics, with a nice view -- 10-50 hours
    * Forward slicing from syscalls,