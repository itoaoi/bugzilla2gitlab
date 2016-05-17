[![Build Status](https://travis-ci.org/xmunoz/bugzilla2gitlab.svg?branch=master)](https://travis-ci.org/xmunoz/bugzilla2gitlab)

# bugzilla2gitlab

## Introduction

This is a tool for developers or admins who want to migrate the issue management for their software project from Bugzilla to Gitlab Issues. Here is a screenshoot of an issue created by bugzilla2gitlab:
![bugzilla2gitlab created issue](https://raw.githubusercontent.com/xmunoz/bugzilla2gitlab/master/snapshot.png)

## Installation

This library is very much under development. That said, if you like to feel the wind in your hair, simply `pip install bugzilla2gitlab`.

More than likely, you will need to roll up your sleaves and hack on the package to achieve a migration that you are happy with. In this case:

```
git clone git@github.com:xmunoz/bugzilla2gitlab.git
cd bugzilla2gitlab
virtualenv venv
source venv/bin/activate
pip install -r requirements.txt
```

## Usage

bugzilla2gitlab synchronously migrates a user-defined list of bugzilla bugs to a single GitLab project. There are two interfaces for this library. The command line usage:

```
$  bin/run_migrator.py -h
usage: run_migrator.py [-h] [FILE] [CONFIG_DIRECTORY]

Migrate bugs from bugzilla to gitlab.

positional arguments:
  [FILE]              A file containing a list of bugzilla bug numbers to
                      migrate, one per line.
  [CONFIG_DIRECTORY]  The directory containing the required configuration
                      files.

optional arguments:
  -h, --help          show this help message and exit
```

This package can also be used as a python module.

```
from bugzilla2gitlab import Migrator

client = Migrator(config_path="/path/to/config")
bugs_list = [1,2,3,4,5,6,7]
client.migrate(bugs_list)
```

## Configuration

To begin using bugzilla2gitlab, the following list of configuration files is required in the specified `config` directory:

- `defaults.yml`: Core default values used throughout the modules.
- `user_mappings.yml`: key, value pairs of Bugzilla usernames to GitLab users
- `component_mappings.yml`: key, value pairs of Bugzilla components to Gitlab labels

Samples of all of these files with documentation for each configuration variable can be found in `tests/test_data/config`.

bugzilla2gitlab creates issues and comments in GitLab with the user accounts specified in `user_mappings.yml`, perserving the integrity of the original Bugzilla commenter. This, however, may not always be possible. In `tests/test_data/config/user_mappings.yml`, users with the designation "bugzilla" may have left the organization and therefore not have current GitLab accounts, or might simply be machine users. Comments for such users will be left under a generic "bugzilla" account. bugzilla2gitlab doesn't create any new user accounts. All of the accounts specified in `user_mappings.yml` must already exist in your GitLab installation.

## How it works

### GitLab

Gitlab has a comprehensive and extensively documented API. Here are the main endpoints that this library makes use of.

    - [Creating new issues](http://doc.gitlab.com/ce/api/issues.html#new-issue)
    - [Adding comments to issues](http://doc.gitlab.com/ce/api/notes.html)
    - [Uploading files](http://doc.gitlab.com/ce/api/projects.html#upload-a-file)
    - [Changing an issue status](http://doc.gitlab.com/ce/api/issues.html#edit-issue)
    - [Getting user ids](http://doc.gitlab.com/ce/api/users.html#for-admins)

Calls to the Gitlab API must be made with an administrator private token in order to [impersonate other users](http://doc.gitlab.com/ce/api/#sudo).`

### Bugzilla

This script relies on being able to fetch bug data by simply appending `&ctype=xml` to the end of the bugzilla bug url, and then parsing the resultant xml. If this trick doesn't work on your bugzilla installation, then bugzilla2gitlab won't work for you.

## Caveats

Every comment or mention in GitLab typically sends a notification. This is true even for comments/issues created programatically. To avoid users inboxes being flooded with meaningless email notifications and avoid overwhelming your SMTP servers, GitLab users should disable all email notifications (global and group-specific) just prior to the running of this script. This can be done through the [gitlab UI](https://git.zib.de/profile/notifications).

## Tests

There is currently a need for more tests. The tests that are in place can be run by:
```
./runtests tests
```

If you want to test this library on a non-production GitLab instance, I recommend spinning up a [one-click GitLab droplet from DigitalOcean](https://www.digitalocean.com/features/one-click-apps/gitlab/). From there, create a code repository, add some user accounts, and take bugzilla2gitlab for a spin.
