## Initial Setup
1. Fill out .env to connect to Postgres
2. Go to FQDN
3. Customize options (Email e.g.)
4. Bring docker compose up

## SSH Keys
1. On gitea... -> Profile -> SSH -? Add Key -> upload public key
2. On client, e.g. MacOS, test ssh using port 222. 
```
ssh -T -p 222 git@gitea.hughboi.cc
```
3. If that works, we need to edit the ssh config file so we don't need to keep specifying port
Ctrl + Shift + P -> Remote-SSH: Open SSH Configuration File and add the following: <br>
```
Host gitea.hughboi.cc
  HostName gitea.hughboi.cc
  User git
  Port 222
  IdentityFile ~/.ssh/id_ed25519 # optional
  IdentitiesOnly yes # optional
```
- This also adds this to the ~/.ssh/config file on your cliend (ubnt-prod e.g.)

## Git Process
1. Git clone
2. Cd into repository
3. Make changes
4. Git add . (or git add filename)
5. Git commit -m "Message"
6. Git pull # optional but verify changes were made
7. Git push # git push origin main if I need to specify


## Runner / Actions
1. Clone desired repo to my machine
2. Add a '.gitea' folder
   1. add a 'workflows' folder under '.gitea.'
   2. Give file a name like 'test.yaml'
   3. Commit/push back to repo and it should start going

```
mkdir -p .gitea/workflows
```

```
code gitea/workflows/test.yaml
```

```
git add .gitea/workflows/test.yaml
git commit -m "Add test Gitea Actions workflow"
git push origin main
```
