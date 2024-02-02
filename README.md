# Github Project Createrepo

A script to download RPM release assets from GitHub and create a RPM repository using `createrepo`.

Runs on

* RHEL 8 (and compatible)


## Installation

Recommended: Use the [linuxfabrik.lfops.github_project_createrepo LFOps Ansible Role](https://github.com/Linuxfabrik/lfops/tree/main/roles/github_project_createrepo). Alternatively, follow the manual steps below.

Clone this repo:

```bash
cd /opt
git clone --recurse-submodules https://github.com/Linuxfabrik/github-project-createrepo.git
```

Create your configuration. The default path is `/etc/github-project-createrepo.yml`. Have a look at the `/opt/github-project-createrepo/example.yml` file and the Synopsis below.

Install the `createrepo` package.

Use a web server that points to the directory named `base_path` in the configuration file.

If using systemd, set up the timer and service to update your repositories at regular intervals:

```bash
cd /opt/github-project-createrepo
cp -v systemd/github-project-createrepo-update.service /etc/systemd/system/github-project-createrepo-update.service
cp -v systemd/github-project-createrepo-update.timer /etc/systemd/system/github-project-createrepo-update.timer

# adjust the OnCalendar option
$EDITOR /etc/systemd/system/github-project-createrepo-update.timer

systemctl daemon-reload
systemctl enable --now github-project-createrepo-update.timer
```


## Configuration

An example configuration to create a repository for [mydumper](https://github.com/mydumper/mydumper):

```yaml
base_path: '/var/www/html/github-repos'

github_repos:
  - github_user: 'mydumper'
    github_repo: 'mydumper'
    relative_target_path: 'mydumper/el/8'
    rpm_regex: 'mydumper-{latest_version}-\d\+.el8.x86_64.rpm'
```

Full reference:

`base_path`: Mandatory, string. Directory under which all the repos will be placed. This directory has to exist already and should be served by a webserver.

`github_repos`: Optional, list. List of repositories to create from GitHub the latest release assets.<br>Subkeys:

* `github_user`: Mandatory, string. The username of the GitHub repo path. For example, `'Linuxfabrik'`.
* `github_repo`: Mandatory, string. The repo name. For example, `'github-project-createrepo'`.
* `relative_target_path`: Mandatory, string. Target path where the repo should be placed, relative to `base_path`.
* `rpm_regex`: Optional, string. A [Python Regular Expression](https://docs.python.org/3/howto/regex.html) which will be matched against the names of the release assets to select the correct RPM file. You can use `{latest_version}` as a placeholder, which will be replaced by the latest version (retrieved via the GitHub API) before matching. Note that the regex should only match one file, as the first matching file will be downloaded. Defaults to `'.*{latest_version}.*\.rpm'`.
* `number_of_rpms_to_keep`: Optional, int. Number of older RPM files to keep. Note that this simply deletes all older files matching `*.rpm` in the target path directory. Defaults to `3`.


## Exit Codes

* 0: success / config valid
* 1: failed to read config / config invalid
