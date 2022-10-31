# Написание шаблонов
***Будет дополнено!***
## Операции implementations и порядок их выполнения
На текущий момент Clouni поддерживает следующие операции:
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

Порядок выполнения следующий:
1) create (операция, которая есть всегда, даже если явно не задана в implementations) 
2) pre_configure_source и pre_configure_target
3) configure
4) post_configure_source и post_configure_target
5) start
6) stop
7) delete (да, можно задать удаление узла при создании)

Если узел имеет тип **SoftwareComponent** или derived from **SoftwareComponent**, то операции выполняется на удаленном узле, иначе - локально.
Для операций у relationships значение имеет то какой тип имеет SOURCE и TARGET.

При выполнении команды **clouni delete** все операции create, заменяются на delete, 
а остальные операции не выполняются (так как delete имеет самый низкий приоритет).