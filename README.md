# Image builder

From here are building Openstack components' container images (LOCI, but not only) with CI/CD pipelines. 

Some parts were adopted from Vexxhost's Atmosphere (https://github.com/vexxhost/atmosphere). 

# TODO

1/ We probably don't need cmd and internal directories as they are specific for Veexhost deployment but we 
need more generic builds. We added them just have working PoC builder as a good start...

2/ Add our custom changes to Nova, Cinder, etc.
