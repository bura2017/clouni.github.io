# Filter conditions

Conditions used in the mapping  must be implemented for every configuration tool and be located in
`toscatranslator/providers/<provider>/artifacts` directory

## Ansible conditions

Here the *fact* is the key-value object, where keys are the parameters of any cloud resource
and the value are the values of that parameters. *Facts* is the list of *fact*

When using Ansible configuration tool every condition artifact must have '.yaml' extension and consist
onl Ansible tasks for filtering facts.

If developer implement condition artifact, he must use variables `input_facts`, `input_args` as predefined
and define two variables `matched_object` and `matched_objects` in the end.
Variable `matched_objects` contains list of all matched facts and `matched_object` should contain one of them.

**ATTENTION** If there are more then one matched facts the last is taken as matched fact
