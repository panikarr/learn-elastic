# Running Elasticsearch + Kibana (docker-compose) on Podman, behind Zscaler

Setup that worked on this Mac (Docker Desktop removed, Podman used instead).
Compose project: `~/Documents/EAGLE/POC-LEARNING/ELASTIC 9` (Elasticsearch/Kibana 9.2.4).

## Problems hit, in order

| Error | Cause | Fix |
|---|---|---|
| `Cannot connect to the Docker daemon at ...docker.sock` | Docker Desktop no longer installed | Use Podman (steps 1-3) |
| `exec: "docker-credential-desktop": executable file not found` | `~/.docker/config.json` still had `"credsStore": "desktop"` | Step 4 |
| `error saving credentials ... docker-credential-desktop` (during `docker login`) | Same `credsStore` line | Step 4, then step 5 |
| `x509: certificate signed by unknown authority` | Zscaler re-signs HTTPS; the Podman VM didn't trust its root CA | Step 6 |
| `Elasticsearch died ... exit code 137` | Podman VM had 2 GiB RAM, so it was OOM-killed | Step 7 |
| Kibana: `unable to authenticate user [kibana_system]` | `kibana_system` password never set in Elasticsearch | Step 9 |

## Steps

### 1. Install the Podman engine
Podman Desktop alone does not include the `podman` CLI.
```bash
brew install podman
```

### 2. Create the Podman VM
```bash
podman machine init
```

### 3. Point the `docker` CLI at Podman
```bash
docker context create podman --docker "host=unix://$(podman machine inspect --format '{{.ConnectionInfo.PodmanSocket.Path}}')"
docker context use podman
```
(`docker-compose` from Homebrew then works unchanged.)

### 4. Remove the dead Docker Desktop credential helper
Back up, then delete the `"credsStore": "desktop",` line from `~/.docker/config.json`:
```bash
cp ~/.docker/config.json ~/.docker/config.json.bak
sed -i '' '/"credsStore": "desktop",/d' ~/.docker/config.json
```

### 5. Log in to Artifactory (`docker login`)
Do this **after step 4**; before that, login fails with
`error saving credentials ... docker-credential-desktop: executable file not found`.
```bash
docker login docker-remote.rt.artifactory.tio.systems
docker login docker-knowledge-discovery-release-local.rt.artifactory.tio.systems
```
- Username: your Artifactory username.
- Password: an Artifactory API / identity token (not your SSO password).
- Expect `Login Succeeded` for each registry.
- With no credential helper, the login is saved base64-encoded in `~/.docker/config.json` under `auths`.
  Treat that file as secret and keep it out of git. If the `docker-remote` entry is still `{}` afterwards, the login did not
  save; run it again.
- To pull through Artifactory, prefix the `image:` lines in the compose file with the registry host. Check the path with
  `docker pull <registry>/<image>:<tag>` first (Artifactory UI -> "Set Me Up" shows the exact path).
- Only needed for Artifactory images. The public Elastic images pull without a login once step 6 is done.

### 6. Make the VM trust Zscaler (corporate CA)
```bash
podman machine stop podman-machine-default
podman machine set --import-native-ca=true podman-machine-default
podman machine start podman-machine-default
```
Reference: Confluence page "Fixing Podman/BuildKit TLS Certificate Errors on Corporate Network (Zscaler)"
https://elsevier.atlassian.net/wiki/x/_ID0RMds
(Its BuildKit/`buildx` section is only needed for `docker compose build`, e.g. Dockerfiles with `ADD https://...`; plain image pulls don't need it.)

### 7. Give the VM enough memory
```bash
podman machine stop podman-machine-default
podman machine set --memory 6144 podman-machine-default
podman machine start podman-machine-default
```
2 GiB was too little; Elasticsearch has a 1 GiB heap plus off-heap needs.

### 8. Start the stack
```bash
cd ~/"Documents/EAGLE/POC-LEARNING/ELASTIC 9"
docker-compose down       # clears the failed containers from earlier attempts
docker-compose up -d
```
Check: `curl -u elastic:<elastic-password> http://localhost:9200`

### 9. Set the `kibana_system` password (first run only)
Kibana connects as `kibana_system`, whose password is only set in the compose file, not in Elasticsearch, until you do this:
```bash
curl -u elastic:<elastic-password> -X POST -H 'Content-Type: application/json' \
  http://localhost:9200/_security/user/kibana_system/_password \
  -d '{"password":"<kibana_system-password-from-compose>"}'
docker restart kibana
```
Kibana is then at http://localhost:5601. Log in as `elastic`, not `kibana_system`.

## Day-to-day
- Start VM if stopped: `podman machine start`
- Start stack: `docker-compose up -d` (in the compose folder)
- Stop stack: `docker-compose down` (data stays in the `es_data` volume)
- `docker-compose down -v` deletes the volumes, so redo step 9 afterwards.

## Notes
- If Elasticsearch ever complains about `vm.max_map_count`:
  `podman machine ssh "sudo sysctl -w vm.max_map_count=262144"` (lost on VM restart).
- Harmless Kibana warnings: Fleet "encrypted saved object key required" (only matters if you use Fleet; set
  `XPACK_ENCRYPTEDSAVEDOBJECTS_ENCRYPTIONKEY` to 32+ random chars) and a telemetry certificate error (Zscaler).
- Pulling through Artifactory is optional; see step 5 (`docker login`) and prefix the `image:` lines in the compose file
  with the registry host. Not needed for the public Elastic images once step 6 is done.
- `docker-compose.yml` contains plaintext passwords; keep it out of git or move them to an untracked `.env`.
  The `version:` line is obsolete and can be deleted to silence the warning.
