# brkarc-2023_clus2018
## Usage

Since this is a complex system composes of several parts, all of those parts must be in place and configured to completely reproduce this work.  However, Since this repository used submodules, it has to be checked out recursivley:

```
https://github.com/ismc/devnet-2076_clus2018.git --recursive
```

To build the testbed:

```
ansible-playbook build-testbed.yml
```

To run the DMVPN Integration tests:

* Set the baseline system and interface configuration:

```
    ansible-playbook -i inventory/test network-system.yml
```

* (Optional) Checkpoint the routers after the initial configuration and before the DMVPN deployment.

```
    ansible-playbook -i inventory/test network-checkpoint.yml
```

* Deploy the DMVPN overlay:

```
    ansible-playbook -i inventory/test network-dmvpn.yml
```

* Check to make sure that DMVPN overlay as properly deployed

```
    ansible-playbook -i inventory/test network-dmvpn-check.yml
```

* (Optional) Rollback the router configuration to the state before the DMVPN overlay was deployed.

```
    ansible-playbook -i inventory/test network-rollback.yml
```

To destroy the testbed:

```
ansible-playbook desroy-testbed.yml
```
