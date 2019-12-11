# The Foreman Release Engineering scripts

These are the scripts I use while releasing [The Foreman](https://theforeman.org). They automate most of the [release process](https://github.com/theforeman/tool_belt/blob/master/procedures/foreman/release.md.erb).

## Setup

Required dependencies:

 * gopass
 * python3
 * click (python package)
 * python-jenkins

### Gopass

First, clone this repository to your local machine. [gopass](https://github.com/gopasspw/gopass) will need to be installed and then set up on your machine for the release environment.

Ensure that `gopass` is initialized after installing the first time (and that your GPG private key is present on the system):

```
gopass init
```

After installing `gopass`, the shared password stores need to be cloned. This requires setting up SSH configuration to the Foreman puppetmaster:

Add to ~/.ssh/config:

```
Host puppetmaster.theforeman.org
  Port 8122
  User <your SSH user>
  LocalForward 9443 localhost:443
  ExitOnForwardFailure yes
```

Now clone:

```
gopass clone --sync gitcli puppetmaster.theforeman.org:/srv/secretsgit/theforeman-release.git theforeman/releases
gopass clone --sync gitcli puppetmaster.theforeman.org:/srv/secretsgit/theforeman-passwords.git theforeman/shared
```

You will need to create a few gopass passwords for jobs:

For commands on the foreman infrastructure, add your `sudo` password:

```
gopass edit theforeman/unix --create
```

For running jobs in Jenkins, add your Jenkins password:

```
gopass edit theforeman/jenkins-token --create
```

### Importing an existing release

When a key has already been generated, it can be imported from the backups:

```bash
import_gpg_private
```

## Typical release flow

Make sure `VERSION` is correct in `settings` and `FULLVERSION` in `releases/$PROJECT/$VERSION/settings`. This assumes a GPG key is already present.

```bash
./tag_project
./release_tarballs
./download_tarballs
./inspect_tarballs
./sign_tarballs
./bump_deb_packaging
./bump_rpm_packaging
./release_packages
# These steps can happen during the build after RPMs have been built but DEBs are still running
./download_rpms
./sign_rpms
./upload_rpm_signatures
./upload_rpms
./process_rpms
```

When handling non-Foreman releases (currently supported: Katello and Client), set `PROJECT` to the lowercase name of the project and `VERSION` to the version of the project (if it differs from the Foreman one).

```bash
PROJECT=client ./download_rpms
PROJECT=katello VERSION=3.13 ./download_rpms
```

### Generating a new GPG Key for a X.Y release

When starting a new release, the following scripts can be used to generate a new key:

```bash
generate_gpg
# Now set FULLGPGKEY in releases/$VERSION/settings
export_gpg_private
export_gpg_public
sign_gpg
upload_gpg
```
