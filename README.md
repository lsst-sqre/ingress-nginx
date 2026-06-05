# ingress-nginx (Rubin Observatory DM-SQuaRE fork)

## USE AT YOUR OWN RISK

We're not ingress-nginx maintainers, and we are not qualified to be.
If you found this repository while you were looking for some way to put a more modern NGINX into ingress-nginx-controller, you're welcome to try.
However, it's entirely on you to succeed or fail, and we accept no responsibility if something horrible happens to your Kubernetes deployment if you use this package.
We offer no support, and issues filed against it will most likely be cheerfully ignored.

## Rationale

This repository is designed to be a stopgap until Rubin Observatory can migrate to the K8s Gateway API.

We accept that [ingress-nginx is no longer maintainable](README-orig.md).

Nevertheless, we were unable to move away from ingress-nginx by the maintenance deadline and then the subsequent discovery of a major vulnerability.
Thus we've decided to rebuild the ingress-nginx controller with a version of NGINX that does not include the vulnerability.

## Changes from the parent repository

In addition to the obvious changes to the nginx base image and what repository it lives in, we've done a few other things:

 * We dropped almost all of the GitHub Actions, keeping only one that rebuilds the NGINX base container and the controller container.  We are not intending to maintain this package as a going concern.
 * We dropped 32-bit ARM support, leaving only amd64 and arm64 architectures.  Nothing in the Rubin environment that runs Kubernetes will ever need 32-bit ARM.
 * We added instructions for updating to a new version of NGINX, which is the only maintenance action we ever intend to take.
 * We dropped the patch to nginx that had already been addressed upstream.

## How to update NGINX (instructions for Rubin DM SQuaRE)

We're probably going to have to do this again before we've moved away from ingress-nginx.
Here's the process for updating the underlying NGINX version.

If you're doing this for some other project, you will need to read on past this section as well.

### Choose your version

Decide on a version.
Go to https://nginx.org/download and pick the version you want.
Once you've done that, you should get its sha256sum.
Do something like the following:

```bash
NGINX_VERSION=1.30.2
wget https://nginx.org/download/nginx-$NGINX_VERSION.tar.gz
sha256sum nginx-$NGINX_VERSION.tar.gz | awk '{print $1}'
```

### Update files

Make a new branch of this repository (presumably `tickets-DM/something` or `t/DM-something`).

Then edit [images/nginx/rootfs/build.sh](images/nginx/rootfs/build.sh).
Change `NGINX_VERSION` on line 21 to the version you selected.
Then change the checksum on line 192 (which starts with `get_src`) to the checksum you just extracted.

Increment the version numbers of the containers.
[TAG](TAG) holds the controller tag, and [images/nginx/TAG](images/nginx/TAG) holds the NGINX base container tag.
Keep the `-devsquare` label that's appended to the tag.

Change [NGINX_BASE](NGINX_BASE) to pull the base image with the new base container tag.

Update [Changelog.md](Changelog.md) to note the NGINX upgrade.

Commit and push your changes, then open a pull request.

### Let the build run

GitHub Actions will do the base container build and the controller build.
It takes about two hours to rebuild the base container.
The controller is very quick after the base container is done.

Keep an eye on the base container build.
Depending on how extensive the changes to NGINX have been, you may have to regenerate or drop patches, which are found in [images/nginx/rootfs/patches](images/nginx/rootfs/patches).

### Iterate and repeat as necessary

Once all the patches apply (or you've confirmed they are no longer needed), then you're ready to move on.

### After the build

When everything is finished, `ghcr.io/lsst-sqre/nginx:NGINX_TAG` and `ghcr.io/lsst-sqre/ingress-nginx-controller:CONTROLLER_TAG` should both exist, and you can update [Phalanx](https://phalanx.lsst.io) to use the new controller container image.

## How to update if you're not Rubin DM SQuaRE

You have to do all the steps above, but also you're going to need to change `ghcr.io/lsst-sqre` to whatever your container registry is.

[.github/workflows/build.yaml](.github/workflows/build.yaml) contains instances on lines 10 and 95.  Also, if you're not using ghcr.io, you'll need to change the authentication information at lines 119, 157, and 198

Then you'll need to change `REGISTRY` on line 17 of [images/nginx/Makefile](images/nginx/Makefile) and line 61 of [Makefile](Makefile).

The tags in [TAG](TAG), [images/nginx/TAG](images/nginx/TAG), and [NGINX_BASE](NGINX_BASE) should also probably get a label that is something other than `-devsquare`; likewise for your [Changelog.md](Changelog.md) entry.

## Conclusion

This kicks the can down the road a little farther.
It's currently June 5, 2026.
Let's see how long it takes us to move away from ingress-nginx entirely.
