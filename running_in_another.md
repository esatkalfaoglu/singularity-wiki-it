We are generally creating containers in a sandbox fashion. However, when we are copying these sandboxes, some files might need root file which is not a desirable situation in servers where users lack sudo commands

Therefore the proper way is to convert to sif image with the command
```
singularity build --fakeroot mmseg_2107.sif mmseg_2107.sig/
```
where `--fakeroot` command is for sudo copy operations. 

When the copy of the sandbox is tried, the observed problem was that the folder links are not copied. Therefore, for instance, /usr/local/cuda which is linked to /usr/local/cuda11-4 does not appear. Although that problem is resolved by adding link, other problems might arise because of the missing links or missing copy operations. Therefore, converting it to a sif container is a better solution for deployment options. 

For conversion from sif image to sig image is 
```
singularity build --fakeroot --sandbox mmseg_2107.sig/ mmseg_2107.sif
```

For more informations: [Build a Container](https://sylabs.io/guides/3.0/user-guide/build_a_container.html)
