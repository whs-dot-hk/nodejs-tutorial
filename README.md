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
