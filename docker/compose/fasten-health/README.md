Not fully setup. Production has an encrypted database, so figure out how to do that before this is officially done. The config.yaml is new to me. I hadn't been running it the whole time but I should so that database can be encrypted. Current implementation doesn't even use the config.yaml file, but to fully setup in production, I should be using that <br>

```sh
database:
  encryption:
    enabled: false
  #    key: ''
```

<br>
^^ this will need to be changed to <br>

```sh
enabled: true
```
