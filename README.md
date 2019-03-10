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

What role does:

 * installs Podman
 * pulls required images
 * on consecutive runs it pulls image again, and restarts container if image changed
 * creates systemd file for container
 * set's container to be always automatically restarted if container dies.
 * makes container enter run state at system boot
 * adds or removes containers exposed ports to firewall.

Requirements
------------

Requires system which is capable of running podman, and that podman is found
from package repositories. Role installs podman. Role also installs firewalld
if user has defined ```container_firewall_ports``` -variable.

Role Variables
--------------

Role uses variables that are required to be passed:

- ```container_image``` - container image and tag, e.g. nextcloud:latest
- ```container_name``` - Identify the container in systemd and podman commands.
  Systemd service file be named container_name--container-pod.service.
- ```container_run_args``` - Anything you pass to podman, except for the name
  and image.
- ```container_run_as_user``` - Which user should systemd run container as.
  Defaults to root.
- ```container_state``` - container is installed and run if state is
  ```running```, and stopped and systemd file removed if ```absent```
- ```container_firewall_ports``` - list of ports you have exposed from container
  and want to open firewall for. When container_state is absent, firewall ports
  get closed. If you don't want firewalld installed, don't define this.

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
    container_image: sebp/lighttpd:latest
    container_name: lighttpd
    container_run_args: >-
      --rm
      -v /tmp/podman-container-systemd:/var/www/localhost/htdocs:Z
      -p 8080:80
    #container_state: absent
    container_state: running
    container_firewall_ports:
      - 8080/tcp
      - 8443/tcp
  import_role:
    name: podman-container-systemd
```


License
-------

GPLv3

Author Information
------------------

Ilkka Tengvall <ilkka.tengvall@iki.fi>
