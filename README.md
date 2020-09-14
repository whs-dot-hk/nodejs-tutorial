# Install ansible
https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-with-pip

```sh
$ pip install --user ansible
```

https://pip.pypa.io/en/stable/reference/pip_install/#cmdoption-user
```sh
$ export PATH=$PATH:$HOME/.local/bin
$ which ansible
/home/whs/.local/bin/ansible
```

# Install nodejs and yarn
https://github.com/nvm-sh/nvm

https://yarnpkg.com/getting-started/install

```yaml
- args:
    creates: "{{ ansible_env.HOME }}/.nvm"
  name: Install node
  shell: >-
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh |
    bash && source {{ ansible_env.HOME }}/.nvm/nvm.sh && nvm install node
- file:
    path: "{{ ansible_env.HOME }}/.yarn"
    state: directory
  name: Create dot yarn directory
- name: Install yarn
  unarchive:
    creates: "{{ ansible_env.HOME }}/.yarn/bin"
    dest: "{{ ansible_env.HOME }}/.yarn"
    extra_opts: "--strip-components=1"
    remote_src: "yes"
    src: "https://yarnpkg.com/latest.tar.gz"
```

```sh
$ export PATH=$PATH:$HOME/.yarn/bin
$ source ~/.bashrc
```

# Typescript
```sh
$ mkdir my-tc-project
$ cd my-tc-project
$ yarn add -D typescript @types/node
$ touch tsconfig.json
```

Add `scripts` to `package.json`
```json
package.json
---
{
  "scripts": {
    "build": "tsc",
    "start": "node index.js"
  },
  "devDependencies": {
    "@types/node": "^14.6.4",
    "typescript": "^4.0.2"
  }
}
```

```sh
$ yarn build
$ yarn start
```

# Install bazel
https://docs.bazel.build/versions/master/install.html

```yaml
- become: "yes"
  get_url:
    dest: /usr/local/bin/bazel
    mode: "0755"
    url: >-
      https://github.com/bazelbuild/bazel/releases/download/3.5.0/bazel-3.5.0-linux-x86_64
  name: Install bazel
```

# Install docker
https://docs.docker.com/engine/install/binaries

```yaml
- become: "yes"
  name: Install docker
  unarchive:
    creates: /usr/local/bin/docker
    dest: /usr/local/bin
    extra_opts:
      - "--strip-components=1"
    remote_src: "yes"
    src: >-
      https://download.docker.com/linux/static/stable/x86_64/docker-19.03.9.tgz
```

Add user to `docker` group
```sh
$ sudo usermod -aG docker $USER
```

Logout and login

# Typescript with bazel
https://bazelbuild.github.io/rules_nodejs

```sh
$ yarn create @bazel my_bazel_tc_project
$ cd my_bazel_tc_project
$ yarn install
$ yarn build
```

## Add typescript
https://www.npmjs.com/package/@bazel/typescript

```sh
$ yarn add -D typescript @types/node @bazel/typescript
$ touch tsconfig.json
```

Update `BUILD.bazel`
```starlark
BUILD.bazel
---
exports_files(
    ["tsconfig.json"],
    visibility = ["//visibility:public"],
)

package(default_visibility = ["//visibility:public"])

load("@npm//@bazel/typescript:index.bzl", "ts_library")

ts_library(
    name = "index",
    srcs = ["index.ts"],
    deps = [
        "@npm//@types/node",
    ],
)
```

## Add `rules_docker`
https://github.com/bazelbuild/rules_docker

```sh
$ mv WORKSPACE.bazel WORKSPACE
```

Add the following lines to `WORKSPACE`
```starlark
WORKSPACE
---
...

http_archive(
    name = "io_bazel_rules_docker",
    sha256 = "4521794f0fba2e20f3bf15846ab5e01d5332e587e9ce81629c7f96c793bb7036",
    strip_prefix = "rules_docker-0.14.4",
    urls = ["https://github.com/bazelbuild/rules_docker/releases/download/v0.14.4/rules_docker-v0.14.4.tar.gz"],
)

load(
    "@io_bazel_rules_docker//repositories:repositories.bzl",
    container_repositories = "repositories",
)
container_repositories()

load("@io_bazel_rules_docker//repositories:deps.bzl", container_deps = "deps")

container_deps()

load("@io_bazel_rules_docker//repositories:pip_repositories.bzl", "pip_deps")

pip_deps()

load(
    "@io_bazel_rules_docker//nodejs:image.bzl",
    _nodejs_image_repos = "repositories",
)

_nodejs_image_repos()
```

Add the following lines to `BUILD.bazel`
```starlark
BUILD.bazel
---
...

load("@io_bazel_rules_docker//nodejs:image.bzl", "nodejs_image")

nodejs_image(
    name = "nodejs_image",
    data = [
        ":index",
    ],
    entry_point = "index.ts",
)
```

## Run `index.js`
```sh
$ bazel build //:index
$ node dist/bin/index.js
```

## Run nodejs image
Start `dockerd`
```sh
$ sudo dockerd
```

```sh
$ bazel run //:nodejs_image
INFO: Invocation ID: d1c650cb-f93e-4793-b3ae-9f0cc210e05d
INFO: Analyzed target //:nodejs_image (111 packages loaded, 7629 targets configured).
INFO: Found 1 target...
Target //:nodejs_image up-to-date:
  dist/bin/nodejs_image-layer.tar
INFO: Elapsed time: 2.291s, Critical Path: 0.78s
INFO: 32 processes: 32 remote cache hit.
INFO: Build completed successfully, 64 total actions
INFO: Build completed successfully, 64 total actions
Loaded image ID: sha256:484fba741352c3b252850d45a112001e064f8bbcc6670ad9e0871d3ac592616f
Tagging 484fba741352c3b252850d45a112001e064f8bbcc6670ad9e0871d3ac592616f as bazel:nodejs_image
Hello world
```

# Towards reproducable builds
On *another machine*
```dockerfile
Dockerfile
---
# 3.5.0
FROM gcr.io/cloud-marketplace-containers/google/bazel@sha256:ace9881e6e9c5d48b5fd637321361aeffe54000265894a65f7d818dc1065bd80

COPY . .
```

Notice there is no nodejs or yarn in the `Dockerfile`

Edit `.bazelversion`
```
.bazelversion
---
3.5.0
```

```sh
$ docker build --network=host -t test .
$ docker run -v/var/run/docker.sock:/var/run/docker.sock --network=host test run //:nodejs_image
```

## Jenkins
https://github.com/jenkinsci/kubernetes-plugin
```groovy
Jenkinsfile
---
podTemplate(yaml: """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: bazel
    image: gcr.io/cloud-marketplace-containers/google/bazel@sha256:ace9881e6e9c5d48b5fd637321361aeffe54000265894a65f7d818dc1065bd80 # 3.5.0
    command:
    - cat
    tty: true
    volumeMounts:
    - name: dockersock
      mountPath: /var/run/docker.sock
  volumes:
  - name: dockersock
    hostPath:
      path: /var/run/docker.sock
"""
) {
  node(POD_LABEL) {
    checkout scm

    container("bazel") {
      dir("my_bazel_tc_project") {
        sh "bazel run //:nodejs_image"
      }
    }
  }
}
```
