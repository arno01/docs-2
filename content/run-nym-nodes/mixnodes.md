---
title: "Mixnodes"
weight: 30
description: "Mixnodes accept Sphinx packets, shuffle packets together, and forward them onwards, providing strong anonymity for internet users."
---

### Mixnodes

{{% notice info %}}
The Nym Mixnodes were built in the [Quickstart](../../quickstart). If you haven't yet built Nym and want to run the code, go there first.
{{% /notice %}}

You can run the mixnode from the `nym` top-level directory like this:

`target/release/nym-mixnode`

You should get a welcome message:

```shell
nym$ target/release/nym-mixnode


      _ __  _   _ _ __ ___
     | '_ \| | | | '_ \ _ \
     | | | | |_| | | | | | |
     |_| |_|\__, |_| |_| |_|
            |___/

             (mixnode - version 0.8.0)

    usage: --help to see available options.
```

Mixnodes accept Sphinx packets, shuffle packets together, and forward them onwards.

On receipt of a packet, the mixnode unwraps a layer of Sphinx encryption and assigns a delay timer based on information inside the unwrapped packet header. When the timer expires, the node forwards the packet payload (which is another Sphinx packet) to its next destination based on the routing information in the packet header. The process repeats, with layer-encrypted Sphinx packets being progressively unwrapped and forwarded, until the packet gets to its destination. The shuffling together of lots of packets within the nodes is what provides privacy to users.

Routing and delay information is chosen by the client, rather than by mixnodes.

#### Seeing available options

`target/release/nym-mixnode --help` provides a list of available commands. You can always see help info for a given subcommand by doing `target/release/nym-mixnode <commandname> --help`

#### Init the mixnode config

The `init` command saves a configuration file to disk. You **must** supply 3 parameters: 

1. `--id` a name for this mixnode (determines where the config file will be saved, keep it to one word)
1. `--host` needs to be an IPv4 or IPv6 address. If you're planning to join the testnet, you'll need to make sure this address is routable from the open internet. Hostnames also work but they need to be resolvable at the time you `init`. 
1. `--layer` needs to be an integer (1, 2, or 3) to assign the mixnode to a layer in the network topology. This option will go away in a few weeks, as we're working on having the system assign layer automatically according to need.

If you'd like, you can add in: 

1. `--location` an OPTIONAL parameter that tells us where your node is located. Helpful for tracking distances between nodes, which can tell us a bit about latency and help with debugging.

Example: 

`target/release/nym-mixnode init --id winston-smithnode --host 91.236.6.149 --layer 1 --location London`

Results in:

```
nym$ target/release/nym-mixnode init --id winston-smithnode --host 91.236.6.149 --layer 1 --location London


      _ __  _   _ _ __ ___
     | '_ \| | | | '_ \ _ \
     | | | | |_| | | | | | |
     |_| |_|\__, |_| |_| |_|
            |___/

             (mixnode - version 0.8.0)

    
Initialising mixnode winston-smithnode...
 2020-03-20T11:19:41.721 INFO  pemstore::pemstore > Written private key to "~/.nym/mixnodes/winston-smithnode/data/private_sphinx.pem"
 2020-03-20T11:19:41.721 INFO  pemstore::pemstore > Written public key to "~/.nym/mixnodes/winston-smithnode/data/public_sphinx.pem"
Saved mixnet sphinx keypair
Saved configuration file to "~/.nym/mixnodes/winston-smithnode/config/config.toml"
Mixnode configuration completed.
```

Have a look at the saved configuration files to see more configuration options.

{{% notice info %}}
You'll see a startup warning whenever you bind to your loopback address, because you won't be routable for clients out on the big internet.
{{% /notice %}}

However, if you are attempting to join the Nym testnet, your `--host` parameter *must* contain a publicly routable internet address.

#### Running a single mixnode

The `target/release/nym-mixnode run` command runs a mixnode.

Example: 

`target/release/nym-mixnode run --id winston-smithnode`

```shell
nym$  ./target/release/nym-mixnode run --id winston-smithnode

      _ __  _   _ _ __ ___
     | '_ \| | | | '_ \ _ \
     | | | | |_| | | | | | |
     |_| |_|\__, |_| |_| |_|
            |___/

             (mixnode - version 0.8.0)


Starting mixnode...

Public key: rbl74MfQ-xVqtBIOq2coPGWxNztlqNDqP0R0kLB9p2g=
Directory server: https://directory.nymtech.net
Listening for incoming packets on 91.236.6.149:1789
Announcing the following socket address: 91.236.6.149:1789
```

`./target/release/nym-mixnode help run` shows available options.


#### Virtual IPs, Google, AWS, and all that

On some services (e.g. AWS, Google), the machine's available bind address is not the same as the public IP address. In this case, bind `--host` to the local machine address returned by `ifconfig`, but also specify `--announce-host` with the public IP. Please make sure that you pass the correct, routable `--announce-host`.

For example, on a Google machine, you may see the following output from the `ifconfig` command:

```
ens4: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1460
        inet 10.126.5.7  netmask 255.255.255.255  broadcast 0.0.0.0
        ...
```

The `ens4` interface has the IP `10.126.5.7`. But this isn't the public IP of the machine, it's the IP of the machine on Google's internal network. Google uses virtual routing, so the public IP of this machine is something else, maybe `36.68.243.18`.

`nym-mixnode init --host 10.126.5.7 --layer 1`, starts the mixnode, but no packets will be routed because `10.126.5.7` is not on the public internet.

Trying `nym-mixnode init --host 36.68.243.18 --layer 1`, you'll get back a startup error saying `AddrNotAvailable`. This is because the mixnode doesn't know how to bind to a host that's not in the output of `ifconfig`.

The right thing to do in this situation is `nym-mixnode init --host 10.126.5.7 --announce-host 36.68.243.18 --layer 1`.

This will bind the mixnode to the available host `10.126.5.7`, but announce the mixnode's public IP to the directory server as `36.68.243.18`. It's up to you as a node operator to ensure that your public and private IPs match up properly.

#### What layer should I use?

We are running a mixnet that's 3 layers deep for our testnet. Have a look at the [Nym testnet dashboard](https://dashboard.nymtech.net) and slot yourself into layer 1, 2, or 3. 

We are currently working to eliminate the need for you to choose a layer - the system itself will soon do the job automatically.

#### IPv4 vs IPv6

If you're joining the testnet, your mixnode **must speak both IPv4 and IPv6**. You will need to cooperate with other nodes in order to route traffic.

#### Check the dashboard

Once you've started your mixnode and it connects to the testnet directory server at https://directory.nymtech.net, your node will automatically show up in the [Nym testnet dashboard](https://dashboard.nymtech.net).
