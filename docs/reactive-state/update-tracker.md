The core module - bulk update tracker.

Proposal - proxyequal could return a tree of used keys in 2 forms
 - the values which were accessed. Their change could mean nothing.
 - the values which were used. Their change are important
 
 First could be used to get the right "paths" to the seconds, as long "no change"
  in the path means - there is no change in values.
  
  
Idea - track used keys and subscribe to the root store using some "tree" format.
As for me - one of the redux problem - need to touch all mapStateToProps and selectors to generate an update.
As long we `knew` the keys we are expecting, and where - we could form a tree of subscribers, traverse that tree,
which will tool some constant(not bound to Component count) time, and result the list of 
Component to be updated.

The keys difference from `memoize-state` is they way `update` us calculated.

- memoize-state accepts the old and the new states, and the list of keys to compare.
- reactive-state accepts the _"changes"_ and does not have to compare 100500 _used_ values - only 10 _changed_ among used

It will be not easy to track the change - we have to reimpliment `immer`.
As long we had to maintain compatibility with _reducers_ it could be tricky, and more about
deep_equal. I hope immutability itself will help us to track down changed entities on store replacement.

I thing this is a main idea to spike.

Probably this update set is a good fit for Context's changed bits, as long this is somehow similar, and could change set could be converted to a bit mask.
Not sure how 31 bits could be usable, but could. 
 