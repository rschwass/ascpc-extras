# Internal CI/CD Pivots Lab Setup

Use this guide to get your local lab environment ready before following along with the instructor.

## Requirements

- VMware Workstation
- The provided WKL lab OVA
- A VMware display session to the Ubuntu Guest GUI
- At least 8 GB RAM available for the VM
- Enough disk space for the imported appliance

## Import And Start The VM

1. Open VMware Workstation.
2. Import the provided OVA.
3. Start the imported VM.
4. Wait several minutes after first boot. GitLab is the slowest service to become ready.

Log in to the Ubuntu Guest console:

```text
Username: student
Password: use the value provided in your lab manual via student portal
```

## Work Directly In The VM GUI

All lab work is done directly inside the Ubuntu Guest GUI:

- open the browser inside the VM
- open terminal tabs inside the VM
- run listeners inside the VM
- use `localhost:8080` for the lab services

You do not need SSH, a separate attack VM, or a host browser workflow for the standard class path.

## Optional SSH Convenience

The standard workflow is to work directly in the Ubuntu Guest GUI. If you prefer easier copy/paste from your host terminal, you may optionally SSH into the Ubuntu Guest after it boots.

Inside the Ubuntu Guest terminal, find the VM IP:

```bash
hostname -I
```

From your host terminal:

```bash
ssh student@<ubuntu-guest-ip>
```

Use the `student` password provided in your lab manual or student portal.

Use the same `localhost:8080` service URLs from the SSH session:

```bash
export GITLAB_URL=http://localhost:8080/gitlab
export JENKINS_URL=http://localhost:8080/jenkins
export NEXUS_URL=http://localhost:8080/nexus
export WEBAPP_URL=http://localhost:8080/webapp
```

Keep the browser workflow inside the Ubuntu Guest GUI unless your instructor tells you otherwise.

## Service URLs

Set these in an Ubuntu Guest terminal (or ~/.bashrc if you want it to survive reboots and new shells):

```bash
export GITLAB_URL=http://localhost:8080/gitlab
export JENKINS_URL=http://localhost:8080/jenkins
export NEXUS_URL=http://localhost:8080/nexus
export WEBAPP_URL=http://localhost:8080/webapp
```

Open the same URLs in the browser inside the Ubuntu Guest GUI.

## Starting Credentials

Use the starting credentials provided in your lab manual or student portal.

```text
GitLab username: wkluser
GitLab password: use the value provided in your lab manual via student portal
```

You should discover later credentials during the lab instead of reading them here.

## First Health Checks

Run these from the Ubuntu Guest terminal:

```bash
curl -I "$GITLAB_URL/users/sign_in"
curl -I "$JENKINS_URL/login"
curl -I "$NEXUS_URL/"
curl -s "$WEBAPP_URL/health"
```

Expected results:

- GitLab returns HTTP `200`.
- Jenkins returns HTTP `200`.
- Nexus returns HTTP `200`.
- Webapp returns JSON containing `rubylog 0.1.0`.

If GitLab returns an error immediately after boot, wait a few more minutes and retry.

## Callback Networking

Some lab steps require Jenkins, GitLab CI, or the webapp container to call back to a listener running on the Ubuntu Guest.

Do not use `localhost` for those callback URLs. Inside a container, `localhost` means that container.

Use the Docker bridge gateway:

```bash
export LAB_DOCKER_NET=$(docker network ls --format '{{.Name}}' | awk '/_labs$/ { print; exit }')
export UBUNTU_GUEST_CALLBACK_HOST=$(docker network inspect "$LAB_DOCKER_NET" --format '{{(index .IPAM.Config 0).Gateway}}')
echo "$UBUNTU_GUEST_CALLBACK_HOST"
```

Use that value when the lab asks for a callback host.

## Quick Troubleshooting

Check running containers:

```bash
docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'
```

Check the webapp:

```bash
curl -s http://localhost:8080/webapp/health
```

Check the GitLab runner:

```bash
docker logs --tail 80 wkl-persistent-runner
```

If the appliance was just imported or rebooted, give it time before troubleshooting. GitLab can take longer than the other services.

## Working Notes

- Keep one Ubuntu Guest terminal for commands.
- Keep the Ubuntu Guest browser open to the proxy URLs.
- Use the lab guide for the student path.
- Use the instructor's screen for pacing and explanations.
- SSH is optional for easier copy/paste, but the standard class workflow uses the VM GUI.
- Do not reset or prune Docker state during the lab unless instructed.
