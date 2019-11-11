
# ahp_cheatsheet

## Docker
Run from _Dockerfile_ in directory:
```docker build -t name .```


## Ubuntu
### Setup polish locales
Show locales:
`locale -a`

Show default locales:
`cat /etc/default/locale `

Install polish locale:
`sudo locale-gen pl_PL.utf8`

Setup locales:
`update-locale LANG=pl_PL.utf8`

