Single Disk Swift
=================

Run a single-disk, single-replica [Swift](https://docs.openstack.org/swift/latest/)
"cluster" using Docker.

At the moment, this is fairly opinionated: it's only been tested on Fedora (34,
though other releases may work), and it expects to have full control over an
entire disk. Pro-tip: flash is better than SMR.

This should work reasonably well as a dev or test target. The data on the drive
will persist through reboots and upgrades.

You may need a couple extra ansible module collections:

    $ ansible-galaxy collection install ansible.posix ansible.docker

But after that, you just need to edit inventory.yaml-sample to suit your needs
and run

    $ ansible-playbook -i <your inventory> setup.yaml --ask-become-pass

Logging
-------

To view Swift logs, try

    $ docker exec -it docker-swift-all-in-one.service /bin/cat /var/log/s6-uncaught-logs/current

Yes, this is unfortunate and should be fixed upstream.

Tear Down
---------

To destroy all data on the disk, try

    $ ansible -i <your inventory> all -m include_role -a name=nuke-disk --ask-become-pass

Future Work
-----------

Get logging sorted. `journalctl -u docker-swift-all-in-one.service` needs
to be something that's actually useful.

Parameterize the systemd service name to accomodate running multiple
containers each with their own disk on the same host.

Parameterize the tag to pull. Should make upgrade testing a bit easier.

Get mount_check support, to play well with nuke-disk.

Test against more platforms -- at least Ubuntu 20.04.

Get on py3. Unfortunately, the published upstream py3 images are not currently
functional.

Accept an operator-provided `/etc/swift/proxy-server.conf` so they can
customize available middleware (notably: auth, encryption).

Accept operator-provided `swift_hash_path_prefix`/`swift_hash_path_suffix`
values for `/etc/swift/swift.conf`.

Find some reasonable workflow to use this in developing custom middlewares.

Ensure auditors are running to maintain data integrity. Find some reasonable
workflow to use this in developing audit watchers.
