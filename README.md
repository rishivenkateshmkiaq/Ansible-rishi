# Ansible DevOps assessment

This project contains the assessment playbooks. The default entry point,
`site.yml`, is intentionally limited to inventory verification, Docker Engine,
and Nginx so it can run on the currently configured small EC2 instance.

## Configure the EC2 target

Edit `inventory/hosts.ini` when the instance public IP or key path changes.
Keep the private key outside this repository and restrict its permissions:

```bash
chmod 400 /path/to/ec2-key.pem
```

The inventory currently targets an Ubuntu EC2 instance as `ubuntu`.
Allow SSH (TCP 22) from your control machine in the EC2 security group. Allow
HTTP (TCP 80) from the client addresses that should view the Nginx page.

## Run the lightweight deployment

From this directory:

```bash
ansible-inventory --graph
ansible ansible_targets -m ping
ansible-playbook --syntax-check site.yml
ansible-playbook site.yml
```

The play installs Docker Engine and Nginx, starts both services, and publishes
a simple page on port 80. Verify from the control machine:

```bash
curl --fail http://<EC2_PUBLIC_IP>/
ansible ansible_targets -b -m shell -a \
  'docker --version; systemctl is-active docker nginx; df -h /; free -h'
```

If the playbook reports a sudo password error, rerun it with
`--ask-become-pass`.

## Other assessment playbooks

Playbooks `04_jenkins.yml` through `08_prometheus_grafana.yml` are not imported
by `site.yml` and are not part of this small-instance deployment. Do not run
them on the current EC2 instance: its RAM and disk capacity are insufficient
for those services. Harbor, SonarQube, Jenkins, and monitoring need a larger
host and service-specific security/secrets configuration. Review each
playbook and increase EC2 capacity before running any of them.

## Security

- Never commit the EC2 private key or real service passwords.
- Restrict inbound ports to trusted client IPs; do not expose administration
  services to the public internet.
- The example service playbooks include lab-only configurations. Review and
  secure them before use outside an isolated lab.
