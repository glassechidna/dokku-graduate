# dokku-graduate [![Build Status](https://img.shields.io/travis/glassechidna/dokku-graduate.svg?branch=master "Build Status")](https://travis-ci.org/glassechidna/dokku-graduate)

[Homepage](http://glassechidna.github.io/dokku-graduate/): http://glassechidna.github.io/dokku-graduate/

Dokku Graduate is a simple plug-in for Dokku that helps manage the deployment of multiple Dokku apps to multiple environments.

This plugin is designed to be flexible, but in general builds off the natural
relationship between environments e.g.

```development``` -> ```staging``` -> ```uat```  -> ```production```

## Foreword

Dokku Graduate is brand new. I wrote it to assist with managing my clients'
environments. Whilst I am currently using it for that purpose, I would hardly
consider that battle tested.

## Setup

### Installation

You'll need to install Dokku Graduate on at least two machines (environments):

```shell
# on 0.3.x
cd /var/lib/dokku/plugins
git clone https://github.com/glassechidna/dokku-graduate.git dokku-graduate
dokku plugins-install

# on 0.4.x
dokku plugin:install https://github.com/glassechidna/dokku-graduate.git dokku-graduate
```

### Linking environments

Assuming two environments ```A``` and ```B``` exist with the relationship:

```A``` -> ```B```

Where ```B``` is the successor to ```A```, then apps will graduate *from*
```A``` *to* ```B```.

From a development machines, with access to both environments, execute:

    ssh -t dokku@<A-url> graduate:key | ssh admin@<B-url> "sudo sshcommand acl-add dokku \"dokku-graduate\""
    ssh -t dokku@<A-url> graduate:add-environment <B> <B-url>

## Basic Usage

### Graduating

Whilst logged into an environment:

    sudo dokku graduate <destination-environment>

### Keeping track of environment state

Dokku Graduate turns your Dokku home directory (```/home/dokku```) into a Git
repo. You will not be able to graduate to another environment until you have
committed any changes made to the Dokku home directory.

This mechanism is basically just a way to force you to consciously acknowledge
the presence of factors, outside of each app's Git repo, that have an impact on
the environment.

For example, if a recent app commit required that you define the env var
```MAILER_ADDRESS```, then you'll want to remember to add this same env var to
Dokku app running in the graduation destination environment.

**Before the first time you graduate**, you'll probably just want to do the
following:

    sudo -u dokku bash
    cd /home/dokku
    git add --all .
    git commit -m "Initial environment state"
    exit

### Rolling back

Another advantage of having ```/home/dokku``` as a Git repo is that it makes it
easier for us to roll-back an entire environment to a particular "graduation
version".

Each time a successful graduation to an environment is occurs, an
environment-specific, timestamped tag is added to your Dokku home Git
repository.

Whilst checking out an old commit (or simply looking at the diffs) can certainly
assist in rolling back; Dokku Graduate knows nothing about the intricacies of
your apps. Consequently, it's still your responsibility to actually roll back in
a safe manner.

## Hooks

Dokku Graduate currently provides two named hooks **PRE** and **POST**.

#### PRE

This hook is executed before a graduation has commenced. If the hook fails, the
graduation is aborted.

#### POST

This hook is executed after all apps have been deployed, *but* the apps are not
yet live (old instances of the apps are still live). If the hook fails, all the
newly deployed app instances are shutdown and the graduation is aborted.

### Adding a hook

    sudo dokku graduate:add-hook <PRE | POST> <command>

### Listing hooks (with their *index*)

    sudo dokku graduate:hooks <PRE | POST>

### Removing a hook

    sudo dokku graduate:rm-hook <PRE | POST> <index>
