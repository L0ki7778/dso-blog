# Managing a Running Container

Commands for interacting with and maintaining an already-running Docker container on the VM.

## Creating a Django superuser

```bash
docker exec -it <prefered_name_here> python manage.py createsuperuser
```

## Running other management commands / scripts

```bash
docker exec <prefered_name_here> python manage.py <command>
docker exec <prefered_name_here> python some_script.py
docker exec -it <prefered_name_here> python manage.py shell
```

## Editing a file inside a running container without an installed editor

Option A — change the file within the project and copy it into the running container:

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

## Directory/permission checks

```bash
ls -ld /etc/apt/keyrings
stat /etc/apt/keyrings
[ -d /etc/apt/keyrings ] && echo "exists" || echo "does not exist"
```

## General shell/debugging helpers

```bash
echo $?              # print the exit code of the last command
man <command>         # read a command's manual page
<command> --help      # quick built-in help, faster than man for a single flag
```
