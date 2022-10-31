# Templates
***Will be updated!***
## Implementations operations and the order of their execution
Currently Clouni supports the following operations:
```yaml
tosca.interfaces.node.lifecycle.Standard:
  derived_from: tosca.interfaces.Root
  create:
    description: Standard lifecycle create operation.
  configure:
    description: Standard lifecycle configure operation.
  start:
    description: Standard lifecycle start operation.
  stop:
    description: Standard lifecycle stop operation.
  delete:
    description: Standard lifecycle delete operation.

tosca.interfaces.relationship.Configure:
  derived_from: tosca.interfaces.Root
  pre_configure_source:
    description: Operation to pre-configure the source endpoint.
  pre_configure_target:
    description: Operation to pre-configure the target endpoint.
  post_configure_source:
    description: Operation to post-configure the source endpoint.
  post_configure_target:
    description: Operation to post-configure the target endpoint.
  add_target: # not implemented
    description: Operation to notify the source node of a target node being added via a relationship.
  add_source:
    description: Operation to notify the target node of a source node which is now available via a relationship.
  target_changed: # not implemented
    description: Operation to notify source some property or attribute of the target changed
  remove_target: # not implemented
    description: Operation to remove a target node. 
```

The order of execution is as follows:
1) create (an operation that is always there, even if it is not explicitly specified in implementations) 
2) pre_configure_source и pre_configure_target
3) configure
4) post_configure_source и post_configure_target
5) start
6) stop
7) delete (yes, you can specify deletion operation when creating)

If the node is of type **SoftwareComponent** or derived from **SoftwareComponent** then the operation will be executed on a remote node, otherwise locally.
For relationships operations, it matters what type SOURCE and TARGET have.

When executing the **clouni delete** command, all create operations are replaced with delete operations,
and the rest of the operations are not performed (since delete has the lowest priority).