# ton-deploy

Ansible playbooks that install a [gton](https://github.com/xssnick/gton) node and, optionally, let it take over the identity of a C++ node installed with mytonctrl.

## Playbooks

| Playbook | What it does |
|---|---|
| `playbooks/gton_node_deploy.yml` | Builds gton from source, creates the `gton` user and `/opt/gton/node`, generates keys and config, installs the systemd unit and ufw rules. Aborts if the node path or unit already exists. |
| `playbooks/gton_node_cpp_config.yml` | Stops the C++ `validator` service, reads its ports, keys and custom overlays, reconfigures an already deployed gton node with them and starts it. |

```sh
ansible-playbook -i inventory playbooks/gton_node_deploy.yml -e validator_enabled=true
ansible-playbook -i inventory playbooks/gton_node_cpp_config.yml
```

Both plays run as root (use `-b` or a root login). Hosts need `git`, `jq` and, for the migration, `xxd`. The controller needs the `community.general` collection for ufw.

## Variables

Defaults live in `playbooks/group_vars/all/gton_vars.yml` and are documented there. Override them from the inventory or with `-e`. Node roles are set per play:

| Variable | deploy | cpp_config |
|---|---|---|
| `validator_enabled` | false | true |
| `liteserver_enabled` | true | true |
| `collator_enabled` | false | false |

Ports and keys left empty are generated, or in the migration play read from the C++ node. Validator and collator nodes need `repository_version` set to a gton branch that has validator support.

## Migration notes

* Migration from cpp node requires installation with mytonctrl
* Mytonctrl shall not be installed as root user
* Run the deploy play first, then the migration play on the same host
