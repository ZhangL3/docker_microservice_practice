```sh
docker build .
```

```sh
# get dangling images
docker images -f "dangling=true"
# remove all dangling images
docker prune
```
