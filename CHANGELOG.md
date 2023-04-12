# Changelog

## [1.2.6](https://github.com/chubchubsancho/ansible-pi-hole/compare/v1.2.5...v1.2.6) (2023-04-12)


### Bug Fixes

* **configure:** add become true to task ([677d570](https://github.com/chubchubsancho/ansible-pi-hole/commit/677d570c7266e3a028954fa14ee4d93e42e74eca))
* **handler:** add becom true ([606c4fc](https://github.com/chubchubsancho/ansible-pi-hole/commit/606c4fc7245b65e591546ede098ec031b1a19aca))
* **install:** add become true to task ([0f6a60e](https://github.com/chubchubsancho/ansible-pi-hole/commit/0f6a60e66180eb6660208e45e6fe40fe37bb38c3))
* **molecule:** add cgroupns_mode ([9b1e921](https://github.com/chubchubsancho/ansible-pi-hole/commit/9b1e921b33e527d7b09e4cf35c5a20c77956b05a))
* **molecule:** do not use pre-build image ([710edfb](https://github.com/chubchubsancho/ansible-pi-hole/commit/710edfb181dce0fbce1d34d2f86f466871a5bdc0))
* **pihole:** use generic service module ([68dc9f0](https://github.com/chubchubsancho/ansible-pi-hole/commit/68dc9f0e39da0169b712f2fbb293d0a6419254c1))
* **prerequisite:** add become true to task ([e42501d](https://github.com/chubchubsancho/ansible-pi-hole/commit/e42501d2dc689ec6a940f3075fa7bf270c7f987d))


### Continuous Integration

* **molecule:** no more fix molecule-docker version ([e8370d1](https://github.com/chubchubsancho/ansible-pi-hole/commit/e8370d1fb3bfe9bee6bb2aa5cbb351e9bd02880d))


### Miscellaneous Chores

* **molecule:** use ansible user to run playbook ([6c98a1a](https://github.com/chubchubsancho/ansible-pi-hole/commit/6c98a1a1b7cd5752f80cfffa9f95006ccdda8872))

## [1.2.5](https://github.com/chubchubsancho/ansible-pi-hole/compare/v1.2.4...v1.2.5) (2022-10-12)


### Bug Fixes

* **lint:** use fqdn for include_tasks ([c0da751](https://github.com/chubchubsancho/ansible-pi-hole/commit/c0da75120a940d9a9004045aceccbda58279ab62))


### Continuous Integration

* **release-please:** use PAT instead of GITHUB_TOKEN ([50f5b8c](https://github.com/chubchubsancho/ansible-pi-hole/commit/50f5b8ca941d8c0ccd8eec0626c90887737399df))

## [1.2.4](https://github.com/chubchubsancho/ansible-pi-hole/compare/v1.2.3...v1.2.4) (2022-10-04)


### Continuous Integration

* **publish:** fix job name ([84510a5](https://github.com/chubchubsancho/ansible-pi-hole/commit/84510a532ce30181def9eae2ee223b9aa358cbb1))


### Miscellaneous Chores

* **README:** add badge ([7f61355](https://github.com/chubchubsancho/ansible-pi-hole/commit/7f61355442ad9fe49073ee41e00134fc9a1a7841))

## [1.2.3](https://github.com/chubchubsancho/ansible-pi-hole/compare/v1.2.2...v1.2.3) (2022-10-04)


### Continuous Integration

* **molecule:** change event that trigger ([9552a11](https://github.com/chubchubsancho/ansible-pi-hole/commit/9552a11ffe6cb5e57b0d16e752ca793a0167e2cb))
* **publish:** do not trigger on prereleased ([4d7d30a](https://github.com/chubchubsancho/ansible-pi-hole/commit/4d7d30a6b634f82d7a1a185c436388d7c9e31357))

## [1.2.2](https://github.com/chubchubsancho/ansible-pi-hole/compare/v1.2.1...v1.2.2) (2022-10-04)


### Bug Fixes

* **molecule:** fix molecule-docker version to 2.0.0 ([322fb02](https://github.com/chubchubsancho/ansible-pi-hole/commit/322fb0216397c18a7909c19eab01bf7cc7ee70cd))


### Continuous Integration

* **publish:** create import role workflow ([99ed6b8](https://github.com/chubchubsancho/ansible-pi-hole/commit/99ed6b86433632141a1c5c6ff35aeb6ae55e459a))

## [1.2.1](https://github.com/chubchubsancho/ansible-pi-hole/compare/v1.2.0...v1.2.1) (2022-09-12)


### Bug Fixes

* **linter:** respect task name must start with upper case ([5687411](https://github.com/chubchubsancho/ansible-pi-hole/commit/5687411c865504d4327e8e12d8de6e6365d14a1a))


### Continuous Integration

* **release:** automate release process ([dd2376b](https://github.com/chubchubsancho/ansible-pi-hole/commit/dd2376b45523488e4ea08aad48c3b7c679414675))
* review linter and test workflow ([f228358](https://github.com/chubchubsancho/ansible-pi-hole/commit/f2283586768c175fb47b501c91b4b01ce8a3de57))


### Miscellaneous Chores

* **contributing:** add CONTRIBUTING guide ([f300a5a](https://github.com/chubchubsancho/ansible-pi-hole/commit/f300a5a39b6137069d92959cf67be7ed558038ed))
* **meta:** remove fedora ([e83dd39](https://github.com/chubchubsancho/ansible-pi-hole/commit/e83dd393d8285974298e31606dc3d608e11bc42e))
* **release-please:** configure release-please ([937538e](https://github.com/chubchubsancho/ansible-pi-hole/commit/937538eb5028d8a7cf670811497d7754147f31db))
