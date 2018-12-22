# Docker Swarm Deploy

Automated Docker Swarm cluster setup/provisioning/deployment using Ansible.

Host node OS is assumed to be Ubuntu (tested with 16.04 LTS (Xenial) and 18.04 LTS (Bionic)).

At the end of the provisioning, each node will be configured with:
- Latest Ubuntu updates
- Non-root user access
- SSH access restrained to non-root user and disabled password connection
- Docker Swarm use `ufw` instead of `iptables`
- `ufw` ports setup for generic web access (22, 80, 443) and Docker Swarm communication (2376, 2377, 7946, 4789)
- `fail2ban` setup with sensible ban rules
- `logwatch` setup for daily reports to provided e-mail address

## Initial setup

```
pipenv install -d
pipenv shell
```

## Local development only

1. Run vagrant

    ```
    vagrant up
    ```

2. Add local vm domain names to `/etc/hosts`

    ```
    10.0.0.10	leader.local
    10.0.0.11	manager.local
    10.0.0.12	worker.local
    ```

## Provision host

1. Generate crypted password for non-root user with:

    ```
    python -c "from passlib.hash import sha512_crypt; import getpass; print(sha512_crypt.using(rounds=5000).hash(getpass.getpass()))"
    ```

2. Setup `.env` file (with crypted password)

    ```
    HOST_USERNAME=user
    HOST_PASSWORD=********
    HOST_SSH_PUB_FILE=~/.ssh/id_rsa.pub
    HOST_ADMIN_EMAIL=
    ```

3. Reload `.env` file:
    ```
    pipenv shell
    ```

4. Run the main playbook with the proper inventory config:

    ```
    ansible-playbook -i {development, staging, production} site.yml
    ```

    The first time the whole playbook is run, if it fails to connect try running it with the `root` remote user:

    ```
    ansible-playbook -i {development, staging, production} site.yml -u root
    ```

## Remote management of Docker Swarm

### Setup remote access (with TLS certificates)

```
docker-machine create \
    --driver generic \
    --generic-ip-address=10.0.0.10 \
    --generic-ssh-port 22 \
    --generic-ssh-user root \
    --generic-ssh-key ~/.ssh/id_rsa \
    leader
```

### Connect

```
eval $(docker-machine env leader)
docker node ls
eval $(docker-machine env -u)
```

### Enable Docker Swarm auto-lock

1. Enable auto-lock:

    ```
    docker-machine ssh leader
    > docker swarm update --autolock=true
    > sudo service docker restart
    > exit
    ```

2. Un-lock the swarm using the displayed encryption key:

    ```
    eval $(docker-machine env leader)
    docker swarm unlock
    docker node ls
    ```

## License

The MIT License (MIT)

Copyright (c) 2018 Romain Clement
