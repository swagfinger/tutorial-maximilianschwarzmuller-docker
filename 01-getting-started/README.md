## build a docker file

- build within current directory

```shell
docker build .
```

## running docker

- after build outputs: 'writing image sha256: <hash>'
- on port '3000:3000' which reaches application running on port 3000
- this allows us to goto localhost:3000

```shell
docker run -p 3000:3000 <hash>
```

## stopping

- 'docker ps' lists all running processes
- 'docker stop <container-name>'

```
docker ps
docker stop <X>
```
