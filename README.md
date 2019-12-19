# Version stamp package

A simple package for auto increasing version numbers of any application agnostic to language or architecture. This package comes with three utilities:

* `ver_stamp`
* `goto_version`
* `get_version`


`ver_stamp` is fully compliant with https://semver.org semantics

## Design

The main tool in this package is the `ver_stamp` tool. It is responsible of maintaining version numbers of different applications in an organization.  Before we'll get into its usage, it's important to understand how it operates. The `ver_stamp` tool uses a  backend to save the version information of all of the applications it maintains. The supported back ends are:

*  `git` [WIP]
* `mercurial` - Fully supported

An additional `redis` backend is planned to be developed based on requirements.



## Usage

<u>First thing first</u>

- Init a new repository called `versions` (Depends on the back-end type; `git` or `mercurial`)
- Clone it to your build machine under `/path/to/your_repos/versions`



### ver_stamp

Let's say that your application's code is stored in a `git` repository called `my_app` and this repository's path is `/path/to/your_repos/my_app`

In order to stamp `my_app` you need to decide which backend you would like to use. Let's say that you've decided to go with the `git` back end.  Perform the following steps:

* Run the following command:

```sh
ver_stamp --repos_path /path/to/your_repos/ \
--app_version_file /path/to/your_repos/versions/whatever/path/you/like/version.py \
--release_mode patch \
--app_name my_app
```

The `ver_stamp` tool will create `whatever/path/you/like/version.py` inside the `versions` repository and make the necessary commits and push the whole thing. 

`--release_mode` supports `micro`, `patch`, `minor` and `major`

**Illustration:**

![ver_stamp](https://user-images.githubusercontent.com/5350434/55154276-09d0d480-515d-11e9-8add-f2cb42da3666.jpg)


A typical contents of the `version.py` file would be:

```python
# Autogenerated by version stamper. Do not edit
name = "my_app"
_version = "0.0.1.0"
template = "{0}.{1}.{2}"
version = "0.0.1"
release_mode = "patch"
changesets = {"my_app": {"hash": "74c2c20aa5025e2ab3cf9ab046e67403df7cb124", "vcs_type": "git"}}
```



That's it! Enjoy stamping :)



### goto_version

This utility is made for going back to a specific version. Let's say that you've released `my_app` with version of `2.0.3.4` a month ago. Let's also say that `my_app` depends on a list of other repositories: `repo_a` and `repo_b`  Now you wish to go back to the exact commits in all of the repositories. All you have to do is to use `goto_version` tool and it will checkout  and clone if necessary all of the relevant repositories **in parallel(!)**



```sh
goto_version --repos_path /path/to/your_repos/ --app_name my_app --app_version 2.0.3.4
--git_remote https://github.com/myname/
```



Now if you'd like to go back to the latest commits just run `goto_version`:

```sh
goto_version --repos_path /path/to/your_repos/
```



### get_version

A Very simple tool for printing the latest version of any app to stdout. 

```sh
get_version --repos_path /path/to/your_repos/ --app_name my_app
```



## Advanced features

### version_info.py

By default, `ver_stamp` tool will assume that everything under `/path/to/your_repos/` is required by your application. If you want to specify manually that your application depends on specific repositories, just create a file named `version_info.py` with the following content:

```python
repos = ['my_app_repo', 'repo_a', 'repo_b']
```



### Micro services

`ver_stamp` also supports stamping applications that are built with multiple sub applications. 

Let's assume that we have an application that consists of `service_a` and `service_b` and its called `my_system`. `service_a` and `service_b` have both versions of their own. We want to be able to advance `my_system` version every time `service_a` or `service_b` version is advanced. This is exactly what `ver_stamp` does. 

Run:

```sh
ver_stamp --repos_path /path/to/your_repos \
--app_version_file /path/to/your_repos/versions/apps/my_system/service_a/version.py \
--app_name service_a \
--main_version_file /path/to/your_repos/versions/apps/my_system/main_version.py \
--release_mode patch
```



Now a new file will be generated, committed and pushed along with the `version.py` file that we already know.

A typical contents of the `main_version.py` file would be:

```python
# Autogenerated by version stamper. Do not edit
name = "my_system"
build_num = "1"
version = "19.03.1"
services = {
"service_a": {"path": "/applications/my_system/service_a/version.py", "version": "0.0.1.0"}}
external_services = {
}
```



#### Couple of notes:

1. The `version` field is generated by the following format: `YY.MM.BUILD_NUM`

2. `external_services` can be specified manually by committing to `versions` repository.



### Simultaneous builds

`ver_stamp`  supports simultaneous builds. You can safely run multiple instances of `ver_stamp`



## Installation

```sh
pip3 install version-stamp
```

## How is that `ver_stamp` is agnostic to application language?
It is solely application's reposibility to actualy inject the version number that is in the `version.py` file to the application's code during its build phase. 


## Contributing

If you want to contribute to version-stamp development:

1. Make a fresh fork of `ver_stamp` repository

2. Modify code as you see fit

3. Add tests to your feature

4. Go to your Github fork and create new Pull Request

   

We will thank you for every contribution :)
