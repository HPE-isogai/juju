## View / Operate
| command | description |
| --- | --- |
| juju status | |
| watch -c juju status --color | Execute `juju status` recursively |
| juju scp | |
| juju ssh | |
| juju switch <controller-name> | |
| juju controllers | |
| juju models | |
| juju debug-log | Displays log messages, filters logs by `--include` or `--exclude` options |


## Create
| command | description |
| --- | --- |
| juju add-cloud | Add Cloud provider |
| juju bootstrap | Create controller |
| juju deploy <bundle-file>  | |
| juju add-machine | |
| juju add-unit | |


## Modify / Execute
| command | description |
| --- | --- |
| juju run-action | |

## Delete
| command | description |
| --- | --- |
| juju destroy-model <model name> | |
| juju destroy-controller <controller name> | |
| juju remove-unit <unit-name> | |


- Remove juju environment
```
sudo rm -rf /var/lib/juju
sudo rm -rf /lib/systemd/system/juju*
sudo rm -rf /run/systemd/units/invocation:juju*
sudo rm -rf /etc/systemd/system/juju*
```

