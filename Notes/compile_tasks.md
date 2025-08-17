# Compile tasks 



do_fetch: fetches source code
do_unpack: unpacks the source code
do_patch: locates any patch files and applies them 
do_configure: optional configures the source by enabing any build-time 
    and configuration options for the software being built
do_compile: compiles the source in the compilation directory
do_install: copies files from the compilation directory to a holding area 



### Fetch 
gets stuff from git

You may not need to define function: just define variables like so: 

```
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://..."

SRC_URI = "git://github.com/..."
SRCREV = "f12adfja0912... git hash"
```

#### Unpack 

unpack is done after the fetch task. Fetch will automatically be run before 
unpack. Unpack unpacks it into the work directory

```
S = "${WORKDIR}/git"
```

default value of ````S```` is ````${WORKDIR}/${BPN}-${PV}````: 


### Patch 
locates patch files if any available. Else is skipped. optional step 

```
SRC_URI:append = " file://name.patch"
```


### Configure 
configures the source, before compilation. Optional step 

```
do_configure() {
    // adds a macro to a header file 
    echo "#define A 1" > example.h 
}
```





