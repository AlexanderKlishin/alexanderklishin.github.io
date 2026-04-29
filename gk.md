# gk
* [overview](https://github.com/AltraMayor/gatekeeper/wiki/Overview)
* [deploy](https://github.com/AltraMayor/gatekeeper/wiki/Tips-for-Deployments)
* Functional blocks
    * [cps](https://github.com/AltraMayor/gatekeeper/wiki/Functional-Block:-CPS)
    * [gk](https://github.com/AltraMayor/gatekeeper/wiki/Functional-Block:-GK)

# dpdk
* [guides](https://doc.dpdk.org/guides/)
* [nics](https://doc.dpdk.org/guides/nics/)
    * [ring](https://doc.dpdk.org/guides/nics/ring.html)
    * [virtio](https://doc.dpdk.org/guides/nics/virtio.html)
    * [vhost](https://doc.dpdk.org/guides/nics/vhost.html)
* [prog](https://doc.dpdk.org/guides/prog_guide/index.html)
    * [dev lib](https://doc.dpdk.org/guides/prog_guide/index.html#device-libraries)
        * [vhost lib](https://doc.dpdk.org/guides/prog_guide/vhost_lib.html)
* [howto](https://doc.dpdk.org/guides/howto/index.html)
    * [virtio user container](https://doc.dpdk.org/guides/howto/virtio_user_for_container_networking.html)
* [sample](https://doc.dpdk.org/guides-16.07/sample_app_ug/index.html)
    * [vhost](https://doc.dpdk.org/guides-16.07/sample_app_ug/vhost.html)
* [Getting Started: 9. eal parameters](https://doc.dpdk.org/guides/linux_gsg/linux_eal_parameters.html)
    * run without root
    ```
    sudo dpdk-hugepages.py --reserve 1G
    dpdk-test --in-memory
    dpdk-test --in-memory --vdev=net_ring0 --vdev=net_ring1
    ```

# ovs + dpdk
* [build](https://docs.openvswitch.org/en/stable/intro/install/dpdk/)
```
sudo ovs-vsctl --no-wait set Open_vSwitch . other_config:dpdk-init=true
sudo /usr/local/share/openvswitch/scripts/ovs-ctl start
sudo ovs-vsctl show
sudo ovs-vsctl add-br gk0 -- set bridge gk0 datapath_type=netdev
# sudo ovs-vsctl add-port gk0 p1 -- set Interface p1 type=dpdk options:dpdk-devargs=0000:06:00.1
sudo ovs-vsctl add-port gk0 vhost-user1 -- set Interface vhost-user1 type=dpdkvhostuserclient options:vhost-server-path=/tmp/vhost-user1
```

# i
* [habr virt](https://habr.com/ru/companies/flant/articles/751746/)
* [RH virtio vhost intro](https://www.redhat.com/en/blog/introduction-virtio-networking-and-vhost-net)
* [RH virtio vhost deep dive](https://www.redhat.com/en/blog/deep-dive-virtio-networking-and-vhost-net)
* [vhost-user proto](https://qemu-project.gitlab.io/qemu/interop/vhost-user.html)
* [vdpa](https://www.redhat.com/en/blog/introduction-vdpa-kernel-framework)

# Qs
* make
    * needs many vars
```
# make
error: 'GATEKEEPER_VERSION' undeclared
```
    * move binary check to tests
* move dpdk, lua to external projects, apply patches
* sudo make ????
