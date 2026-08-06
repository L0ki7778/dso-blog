# Deploying a Dockerized App on the VM

Commands for getting a project onto an Ubuntu VM and running it as a Docker container.

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

> `git checkout -b <name>` **creates a new branch** from the current commit — it does not look at the remote. Use plain `git checkout <name>` (no `-b`) to properly track an existing remote branch.

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
