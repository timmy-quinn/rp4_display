
### Build freezes on a particular task 
```
# in the shell it says something like this
0% qtbase-6.8.4-r0 do_install_ptest_base.
```

```sh
# to run that task alone, I can do something like this, with a verbose output 
# as well
bitbake qtbase -c install_ptest_base -f -v
```

It may be that my tmp directory is not very big/does not have a lot of space left




