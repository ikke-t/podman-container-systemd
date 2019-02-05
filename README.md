podman-container-systemd
========================

Role sets up container(s) to be run on host with help of systemd.
[Podman](https://podman.io/) implements container events but does not control
or keep track of the life-cycle. That's job of external tool as
[Kubernetes](https://kubernetes.io/) in clusters, and
[systemd](https://freedesktop.org/wiki/Software/systemd/) in local installs.

I wrote this role in order to help managing podman containers life-cycle on
my personal server which is not a cluster. Thus I want to use systemd for
keeping them enabled and running over reboots.

Requirements
------------

Requires system which is capable of running podman, and that podman is found
from package repositories. Role installs podman.

Role Variables
--------------

Role uses variables that are required to be passed:

- ```container_name``` - Identify the container in systemd and podman commands.
  Systemd service file be named container_name--container-pod.service.
- ```container_run_args``` - Anything you pass to podman, except for the name.
- ```state``` - container is installed and run if state is ```running```,
  and stopped and systemd file removed if ```absent```

This playbook doesn't have python module to parse parameters for podman command.
Until that you just need to pass all parameters as you would use podman from
command line. See ```man podman``` or
[podman tutorials](https://github.com/containers/libpod/tree/master/docs/tutorials)
for info.


Dependencies
------------

No dependencies.

Example Playbook
----------------

See the tests/main.yml for sample. In short, include role with vars:

```
- name: tests container
  vars:
    container_name: lighttpd
    container_run_args: >
      --rm
      -v /tmp/podman-container-systemd:/var/www/localhost/htdocs:Z
      -p 8080:80
      sebp/lighttpd:latest
    #state: absent
    state: running
  import_role:
    name: podman-container-systemd
```


License
-------

GPLv3

Author Information
------------------

Ilkka Tengvall <ilkka.tengvall@iki.fi>
