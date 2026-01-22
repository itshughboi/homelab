Not fully setup. Production has an encrypted database, so figure out how to do that before this is officially done. The config.yaml is new to me. I hadn't been running it the whole time but I should so that database can be encrypted <br>


database:
  encryption:
    enabled: false
  #    key: ''

<bre>
^^ this will need to be changed to
```
enabled: true
```