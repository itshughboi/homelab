- Taken from official documentation: https://github.com/tubearchivist/tubearchivist

## Permissions for elasticsearch

If you see a message similar to Unable to access 'path.repo' (/usr/share/elasticsearch/data/snapshot) or failed to obtain node locks, tried [/usr/share/elasticsearch/data] and maybe these locations are not writable when initially starting elasticsearch, that probably means the container is not allowed to write files to the volume. To fix that issue, shutdown the container and on your host machine run:
```
chown 1000:0 -R /path/to/mount/point
```

This will match the permissions with the UID and GID of elasticsearch process within the container and should fix the issue.