
```
linkerd profile -n emojivoto --open-api web.swagger web-svc > web-sp.yaml

linkerd profile -n emojivoto --proto Voting.proto voting-svc > voting-sp.yaml

linkerd viz profile emoji-svc --tap deploy/emoji --tap-duration 10s -n emojivoto > emoji-sp.yaml
```
