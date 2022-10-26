# Filter conditions

Conditions used in the mapping must be implemented for every configuration tool and be located in
**{ configuration tool name }_plugins/plugins/modules** in clouni client repository:

## Ansible conditions

Here the *fact* is the key-value object, where keys are the parameters of any cloud resource
and the value are the values of that parameters. *Facts* is the list of *fact*.

When using Ansible configuration tool every condition artifact must have '.py' extension and 
be a standard Ansible module exported via ANSIBLE_LIBRARY.

If developer implement condition artifact, he must use variables *input_facts*, *input_args* 
as predefined. The artifact condition module should return only one matched fact or raise an error with a message that no matchable facts were found.

**ATTENTION!** If there are more then one matched facts the last is taken as matched fact.
