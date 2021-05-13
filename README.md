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
 * on consecutive runs it pulls image again,
   and restarts container if image changed (not for pod yet)
 * creates systemd file for container or pod
 * set's container or pod to be always automatically restarted if container dies.
 * makes container or pod enter run state at system boot
 * adds or removes containers exposed ports to firewall.
 * It takes parameter for running rootless containers under given user
   (I didn't test this with pod mode yet)

For reference, see these two blogs about the role:
* [Automate Podman Containers with Ansible 1/2](https://redhatnordicssa.github.io/ansible-podman-containers-1)
* [Automate Podman Containers with Ansible 2/2](https://redhatnordicssa.github.io/ansible-podman-containers-2)

Blogs describe how you can single container, or several containers as one pod
using this module.

## Note for running rootless containers:

* You need to have the user created prior running this role.
* The user should have entries in /etc/sub[gu]id files for namespace range.
  If not, this role adds some variables there in order to get something going,
  but preferrably you check them.
* I only tested the single container mode, not the pod mode with several containers.
  Please report back how that part works! :)
* Some control things like memory or other resource limit's won't work as user.
* You want to increase ```systemd_TimeoutStartSec``` heavily, as we can not
  prefetch the images before systemd unit start. So systemd needs to wait
  for podman to pull images prior it starts container. Might take minutes
  depending on your network connection, and container image size.

Requirements
------------

Requires system which is capable of running podman, and that podman is found
from package repositories. Role installs podman. Role also installs firewalld
if user has defined ```container_firewall_ports``` -variable.

Role Variables
--------------

Role uses variables that are required to be passed while including it. As
there is option to run one container separately or multiple containers in pod,
note that some options apply only to other method.

- ```container_image_list``` - list of container images to run.
  If more than one image is defined, then the containers will be run in a pod.
- ```container_image_user``` - optional username to use when authenticating
  to remote registries
- ```container_image_password``` - optional password to use when authenticating
  to remote registries
- ```container_name``` - Identify the container in systemd and podman commands.
  Systemd service file be named container_name--container-pod.service.
- ```container_run_args``` - Anything you pass to podman, except for the name
  and image while running single container. Not used for pod.
- ```container_cmd_args``` - Any command and arguments passed to podman-run after specifying the image name. Not used for pod.
- ```container_run_as_user``` - Which user should systemd run container as.
  Defaults to root.
- ```container_run_as_group``` - Which grou should systemd run container as.
  Defaults to root.
- ```container_state``` - container is installed and run if state is
  ```running```, and stopped and systemd file removed if ```absent```
- ```container_firewall_ports``` - list of ports you have exposed from container
  and want to open firewall for. When container_state is absent, firewall ports
  get closed. If you don't want firewalld installed, don't define this.
- ```systemd_TimeoutStartSec``` - how long does systemd wait for container to start?
- ```systemd_tempdir``` - Where to store conmon-pidfile and cidfile for single containers.
  Defaults to ``%T`` on systems supporting this specifier (see man 5 systemd.unit) ``/tmp``
  otherwise.

This playbook doesn't have python module to parse parameters for podman command.
Until that you just need to pass all parameters as you would use podman from
command line. See ```man podman``` or
[podman tutorials](https://github.com/containers/libpod/tree/master/docs/tutorials)
for info.

If you want your
[images to be automatically updated](http://docs.podman.io/en/latest/markdown/podman-auto-update.1.html),
add this label to container_cmd_args: ```--label "io.containers.autoupdate=image"```

Dependencies
------------

* [containers.podman](https://galaxy.ansible.com/containers/podman) (collection)
* [ansible.posix](https://galaxy.ansible.com/ansible/posix) (collection)

Example Playbook
----------------

See the tests/main.yml for sample. In short, include role with vars.

Root container:

```
- name: tests container
  vars:
    container_image_list: 
      - sebp/lighttpd:latest
    container_name: lighttpd
    container_run_args: >-
      --rm
      -v /tmp/podman-container-systemd:/var/www/localhost/htdocs:Z
      --label "io.containers.autoupdate=image"
      -p 8080:80
    #container_state: absent
    container_state: running
    container_firewall_ports:
      - 8080/tcp
      - 8443/tcp
  import_role:
    name: podman-container-systemd
```

Rootless container:

```
- name: ensure user
  user:
    name: rootless_user
    comment: I run sample container

- name: ensure directory
  file:
    name: /tmp/podman-container-systemd
    owner: rootless_user
    group: rootless_user
    state: directory

- name: tests container
  vars:
    container_run_as_user: rootless_user
    container_run_as_group: rootless_user
    container_image_list: 
      - sebp/lighttpd:latest
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
