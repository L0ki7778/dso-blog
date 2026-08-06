# Notes to Docs — Commands Reference

Collected commands from setting up and deploying this project on an Ubuntu VM via Docker.


## 1. Getting the code onto the VM

```bash
git clone git@github.com:<org-or-user>/<repo-name>.git
cd <project-name>

git status
git fetch origin
git checkout <branch-name>      # tracks origin/<branch-name> automatically if it exists remotely
git pull

git branch --list               # list local branches
git branch -D <branch-name>     # delete a local branch (e.g. a wrongly created decoy)
git log --oneline -5            # sanity-check you're on the right commits
git diff <file>                 # confirm an edit actually landed
```

> Note: `git checkout -b <name>` **creates a new branch** from the current commit — it does not look at the remote. Use plain `git checkout <name>` (no `-b`) to properly track an existing remote branch.

## 2. Environment configuration

```bash
cp example.env src/.env
# then edit src/.env: set ALLOWED_HOSTS to the VM's IP/domain, DEBUG=false
```

## 3. Build the image

```bash
docker build -t <prefered_name_here>:local .
```

## 4. Run the container

```bash
docker run -d \
  --name <choosen_name_here> \
  --restart unless-stopped \
  -p 8000:8000 \
  --env-file src/.env \
  <prefered_name_here>:local
```

With a persistent volume for the SQLite DB (so data survives container recreation):
```bash
docker run -d \
  --name <choosen_name_here> \
  --restart unless-stopped \
  -p 8000:8000 \
  --env-file src/.env \
  -v <choosen_name_here>-db:/app/src/db.sqlite3 \
  <choosen_name_here>:local
```

## 5. Verifying the deployment

```bash
docker ps
docker logs -f <prefered_name_here>
curl http://localhost:8000
curl -I -H "Host: <one-of-your-ALLOWED_HOSTS-entries>" http://localhost:8000/<path>
```

## 6. Checking if a port is already in use

```bash
sudo ss -tulnp | grep :8000
sudo lsof -i :8000
```

## 7. Redeploying after code changes

```bash
git pull
docker build -t <prefered_name_here>:local .
docker stop <prefered_name_here>
docker rm <prefered_name_here>
docker run -d --name <prefered_name_here> --restart unless-stopped -p 8000:8000 --env-file src/.env <prefered_name_here>:local
```

## 8. Creating a Django superuser in a running container

```bash
docker exec -it <prefered_name_here> python manage.py createsuperuser
```


## 9. Running other management commands / scripts in a running container

```bash
docker exec <prefered_name_here> python manage.py <command>
docker exec <prefered_name_here> python some_script.py
docker exec -it <prefered_name_here> python manage.py shell
```

## 10. Editing a file inside a running container without an installed editor

Option A - change file within project and copy changes to running container:
```bash
nano src/btw_app/urls.py
docker cp src/btw_app/urls.py <prefered_name_here>:/app/src/btw_app/urls.py
```

Option B — install an editor inside the container (temporary, lost on container recreation):
```bash
docker exec -it <prefered_name_here> bash
# then, inside that shell:
apt update && apt install -y nano
nano btw_app/urls.py
```

Restart the container if a change doesn't get picked up by the autoreloader:
```bash
docker restart <prefered_name_here>
```

## 11. Directory/permission checks

```bash
ls -ld /etc/apt/keyrings
stat /etc/apt/keyrings
[ -d /etc/apt/keyrings ] && echo "exists" || echo "does not exist"
```

## 12. General shell/debugging helpers

```bash
echo $?              # print the exit code of the last command
man <command>         # read a command's manual page
<command> --help      # quick built-in help, faster than man for a single flag
```
