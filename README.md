# Deploy EVE-NG PRO to a Digital Ocean droplet

Playbooks to install and configure EVE-NG PRO on Ubuntu 18.04 LTS host. I'm currently using this to deploy to Digital Ocean, but the playbooks are generally cloud-agnostic.

The playbooks accomplish the following:

* Install Docker
* Install EVE-NG PRO
* Install `eveng-dockers`
* Create and configure SSL certificates using Let's Encrypt

**Why is this not a role?:** It will probably be at some point. I just needed to stitch together a few things that I already had.

## Install Requirements

```sh
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Digital Ocean Inventory (Optional)

Since I'm deploying the Digital ocean, I rely on the `community.digitalocean` inventory plugin to pull and group hosts from my droplets.

I install the collection from Ansible galaxy using `ansible-galaxy`:

```sh
ansible-galaxy collection install -r ansible/collections/requirements.yaml
```

**:notebook: Note:** My droplets are tagged with `eve-ng`, the same group targeted by the playbooks. The inventory plugin automatically creates a group with droplet tags, so no additional configuration is necessary.

## Required Variables and configurations

### Variables

for the playbooks, the two required variables are:

* `certbot_email` - to register an account and use the `non-interactive` mode to create SSL certificates
* `certbot_domain` - the site name to generate a certificate for
* `eve_ng_password` - EVE-NG root password

The `pb-eveng.yaml` playbook currently sets these variables from environment variables (`CERTBOT_EMAIL`, `CERTBOT_DOMAIN`, and `EVE_NG_SSH_PASSWORD`).

The digital ocean inventory plugin [configuration](./ansible/inventory/digitalocean.yaml) also requires that `api_token` is set with your API TOKEN. I'm also setting this value using an environment variable.

To avoid committing potentially sensitive data in this repo, I opted to keep the required variables in a `.env` file.

Here's a sample for my `.env` file

```txt
export CERTBOT_EMAIL=someemail@host.com
export CERTBOT_DOMAIN=site.example.com
export DOC_RO_API_KEY=dop_v1_e8f25c....
export EVE_NG_SSH_PASSWORD="sup3rs3cr3t!"
```

### Configuration for SSH login with a public key

I set the path to my private key file for a password-less login in the [`ansible.cfg`](./ansible/ansible.cfg) file.

```ini
[defaults]
privatekeyfile = /Users/thiamt/.ssh/digital_ocean_rsa
```

## :rocket: Deployment

```sh
cd ansible
ansible-playbook deploy-eve-ng.yaml -i inventory -u root
```
