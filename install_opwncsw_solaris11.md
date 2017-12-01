# Install OpenCSW on Solaris 11

Install CSWpkgutil:
```
pkgadd -d http://get.opencsw.org/now
```

Install CSWpki:
```
/opt/csw/bin/pkgutil -y -i cswpki
```

Import gpg key:
```
/opt/csw/bin/cswpki --import
```

Comment in use_gpg=true and use_md5=true in /etc/opt/csw/pkgutil.conf¬

Update catalogue:
```
/opt/csw/bin/pkgutil -U
```
