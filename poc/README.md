# create 257 length passwd
```
```
 python -c 'print("A" * 257, end="")'
```
```

# login
```
```
 ssh -o PreferredAuthentications=password \
  -o PubkeyAuthentication=no \
  -o NumberOfPasswordPrompts=3 \
  -p 2222 alice@127.0.0.1
```
```
