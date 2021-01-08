# Ansible Role: seaweedfs

Provision installation of [seaweedfs](https://github.com/chrislusf/seaweedfs) a distributed object storage.
This role installs the components master, volume and filer.

This role was based on the role [egeneralov.seaweedfs](https://github.com/egeneralov/seaweedfs)

## Inventory

When used in a play targetting group `all` the role will use all nodes and deploy in such a way

* a master server on the first node of the group
* a volume server on each node
* a filer server on the first node of the group

You can also define groups `weed_master`, `weed_volume` and `weed_filer`, to choose
where to deploy the server master and filer accordingly.
In such case, the volume servers will be configured correctly to contact the master server(s).

## Requirements

None

## Role Defaults Variables

    weed.version: '2.18'
    # large_disk is a specific build of seaweedfs to handle volume size bigger than 30 GB.
    weed.large_disk: False
    weed.bind: 0.0.0.0
    weed.ip: "{{ ansible_default_ipv4.address }}"
    weed.location: /usr/local/sbin # where the binary weed is installed
    weed.user.name: seaweed
    weed.defaultReplication: "000"
    weed.master.port: 9333
    weed.master.dir: "/opt/seaweedfs/{{ domain }}/master"
    weed.volume.port: 8080
    weed.volume.dir: "/opt/seaweedfs/{{ domain }}/volume"
    weed.volume.dataCenter: DefaultDataCenter
    weed.volume.rack: DefaultRack
    weed.volume.max_volumes: 16
    weed.filer.port: 8889
    weed.filer.dir: "/opt/seaweedfs/{{ domain }}/filer"
    weed.filer.encryptData: false

## License

MIT

## Authors Information

* Eduard Generalov <eduard@generalov.net>
* Baptiste Mille-Mathias <baptiste.millemathias@gmail.com>
