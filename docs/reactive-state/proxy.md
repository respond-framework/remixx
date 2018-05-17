Idea behind reactive state - Proxy

Proxy should
- detect property access and transparently return a value
- return a proxied version for non POD values.
- maintain the same proxies for the same object used in the same positions.


Proposal
- start with https://github.com/theKashey/proxyequal
- modify it to support "masked" access metrics, to be able easily detect the change in access pattern


Proxy should not do anything more that reporting information about some key got observed, or was not observed, while last time was

For the first iteration proxyequal should fit our needs, but later we have to think more about it,
as long Components(rendering POD values, leafs) and components could return or rely on intermediate values(Objects, arrays, edges)
   