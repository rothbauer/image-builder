# Image builder

From here are building Openstack components' container images (LOCI, but not only) with CI/CD pipelines. 

Some parts were adopted from Vexxhost's Atmosphere (https://github.com/vexxhost/atmosphere). 

# TODO

1/ Use Zuul instead as Earthly builds are deprecated and superseded by Zuul in Atmosphere repo since 2024.1

# Repository upgrade (needs to be done everythime when upgrading images to a new version until we migrate to Zuul)

## Update tags

`grep -R '2023\.2' -l | while read file ; do sed -i 's/2023\.2/2024.1/g' $file ; done`

## Update references

`grep -R 'global PROJECT_REF' | sed 's/images\/\([^\/]*\)\/Earthfile.*PROJECT_REF=/\1 /' | grep -v staffeln | while read project ref ; do new_ref=$(curl -s "https://github.com/openstack/${project}/tree/stable/2024.1" 2>&1 | grep -Po '(?<=currentOid":")[^"]*') ; sed "s/${ref}/${new_ref}/" -i images/${project}/Earthfile ; done`

## Use current patches

Just delete old patches, clone Atmosphere repository and copy the patches from it to respective directories (e.g. `rm -r image-builder/images/nova/patches/* ; cp -a atmosphere/images/nova/patches/* image-builder/images/nova/patches/`)

or use these commands:
`cd image-builder ; find -name "patches" | while read dir ; do rm -rf ${dir}/* ; done`
`git clone https://github.com/vexxhost/atmosphere.git ; cd atmosphere ; git checkout <tag>`
`cd image-builder ; find /tmp/atmosphere -name "patches" | while read dir ; do cp -a ${dir}/* images/$(echo $dir | grep -oP '(?<=images/)[^\/]*')/patches ; done`
