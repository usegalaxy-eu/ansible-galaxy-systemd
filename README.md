# ansible-galaxy-systemd

Sets up Galaxy server processes responsible for:

- servicing web requests for the UI/API
- setting up, starting, monitoring and submitting jobs to a cluster (if configured)

## Requirements

- systemd

## Major Changes

:warning: **You need to reassign jobs to the newly named handlers/workflow schedulers, for example with:**

```sh
mutate reassign-job-to-handler <job_id> <handler_id> [--commit]
```

and with for loop for example like this (be careful with mutate, no warranties):

```bash
for job in $(gxadmin query queue-details | awk '/handler_key_0/ {print $3}'); do gxadmin mutate reassign-job-to-handler $job handler_sn06_0 --commit; done

```

- Workflow Schedulers now have a scalable prefix, too. `galaxy_systemd_workflow_scheduler_prefix` By default the first part of `ansible_hostname`
- Handlers have a `galaxy_systemd_handler_prefix` var now, which should be used to give them a unique key together with their process number. For example by using the hostname as prefix, like in the [deafults][defaults]. This makes an **update of the job_conf necessary**.
- Gunicorn is now scalable: just define a number in `galaxy_systemd_gunicorns` and like the handlers, that many of them will spawn.
- The Socket name should now be without the '.sock' prefix, as the sockets are automatically created for each Gunicorn process.
- Be aware that you may have to rename the sockets in your NGINX config.

## Role Variables

I don't feel like syncing the readme with my default file every time a variable gets added. Just [go there][defaults].

[defaults]: defaults/main.yml

## Dependencies

- [galaxyproject.galaxy](https://github.com/galaxyproject/ansible-galaxy)

Many variables from that role are set. It is assumed you will use that in your playbook as well.

## Example Playbook

### Basic

Install Galaxy on your local system with all the default options:

```yaml
- hosts: localhost
  vars:
    galaxy_server_dir: /srv/galaxy
  connection: local
  roles:
    - galaxyproject.galaxy
    - usegalaxy-eu.galaxy-systemd
```

## License

GPLv3

## Author Information

This role was written and contributed to by the following people:

- [Ott Oopkaup](https://github.com/ooobik)
- [Helena Rasche](https://github.com/hexylena)
