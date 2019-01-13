<!-- This file was automatically generated by the `build-harness`. Make all changes to `README.yaml` and run `make readme` to rebuild this file. -->
[![README Header][readme_header_img]][readme_header_link]

[![Cloud Posse][logo]](https://cpco.io/homepage)

# tfenv [![Build Status](https://travis-ci.org/cloudposse/tfenv.svg?branch=master)](https://travis-ci.org/cloudposse/tfenv) [![Latest Release](https://img.shields.io/github/release/cloudposse/tfenv.svg)](https://github.com/cloudposse/tfenv/releases/latest) [![Slack Community](https://slack.cloudposse.com/badge.svg)](https://slack.cloudposse.com)


Command line utility to transform environment variables for use with Terraform (e.g. `HOSTNAME` → `TF_VAR_hostname`)

__NOTE__: `tfenv` is **not** a [terraform version manager](https://github.com/tfutils/tfenv).


---

This project is part of our comprehensive ["SweetOps"](https://cpco.io/sweetops) approach towards DevOps. 
[<img align="right" title="Share via Email" src="https://docs.cloudposse.com/images/ionicons/ios-email-outline-2.0.1-16x16-999999.svg"/>][share_email]
[<img align="right" title="Share on Google+" src="https://docs.cloudposse.com/images/ionicons/social-googleplus-outline-2.0.1-16x16-999999.svg" />][share_googleplus]
[<img align="right" title="Share on Facebook" src="https://docs.cloudposse.com/images/ionicons/social-facebook-outline-2.0.1-16x16-999999.svg" />][share_facebook]
[<img align="right" title="Share on Reddit" src="https://docs.cloudposse.com/images/ionicons/social-reddit-outline-2.0.1-16x16-999999.svg" />][share_reddit]
[<img align="right" title="Share on LinkedIn" src="https://docs.cloudposse.com/images/ionicons/social-linkedin-outline-2.0.1-16x16-999999.svg" />][share_linkedin]
[<img align="right" title="Share on Twitter" src="https://docs.cloudposse.com/images/ionicons/social-twitter-outline-2.0.1-16x16-999999.svg" />][share_twitter]




It's 100% Open Source and licensed under the [APACHE2](LICENSE).












## Introduction


If you answer "yes" to any of these questions, then look no further!

* Have you ever wished you could easily pass environment variables to terraform *without* adding the `TF_VAR_` prefix?
* Do you use [`chamber`](https://github.com/segmentio/chamber) and get annoyed when it transforms environment variables to uppercase?
* Would you like to use common environment variables names with terraform? (e.g. `USER` or `AWS_REGION`)
* Is there some argument to `terraform init` you want to specify with an environment variable? (e.g. a `-backend-config` property)

**Yes?** Great! Then this utility is for you.

The `tfenv` utility will perform the following transformations:

  1. Lowercase all envs (Terraform convention)
  2. Strip leading or trailing underscores (`_`)
  3. Replace consequtive underscores with a single underscore (`_`)
  4. Prepend prefix (`TF_VAR_`)

__NOTE__: `tfenv` will preserve the existing environment and add the new environment variables with `TF_VAR_`. This is because some terraform providers expect non-`TF_VAR_*` prefixed environment variables. Additionally, when using the `local-exec` provisioner, it's convenient to use regular environment variables. See our [`terraform-null-smtp-mail`](https://github.com/cloudposse/terraform-null-smtp-mail) module for an example of using this pattern.


**But wait, there's more!**

With `tfenv` we can surgically assign a value to any terraform argument using per-argument environment variables.

## Usage


__NOTE__: The utility supports a number of configuration settings which can be passed via environment variables.

  * `TFENV_PREFIX` - Prefix used for all normalized environment variables (default is `TF_VAR_`, but you could do something like `TF_VAR_app_`)
  * `TFENV_WHITELIST` - Whitelist of allowed environment variables. Processed *after* blacklist. Regular expression should match wanted environment variables (by default `.*`)
  * `TFENV_BLACKLIST` - Blacklist of excluded environment variables. Processed *before* whitelist. Regular expression should exclude unwanted/dangerous environment variables (e.g. AWS credentials)

The basic usage looks like this. We're going to run some `command` and pass it `arg1` ... `argN`:

```sh
    tfenv command arg1 arg2 arg3 ... argN
```

So for example, we can pass our current environment to terraform by simply running:

```sh
tfenv terraform plan
```

### Direnv

You can use `tfenv` with direnv very easily. Running `tfenv` without any arguments will emit `export` statements.

Example `.envrc`:

```sh
# Export terraform environment
tfenv
```

### Bash

Load the terraform environment into your shell.

Just add the following into your shell script:

```sh
source <(tfenv)
```

### Terraform Args

With `tfenv` we can populate the [`TF_CLI_ARGS` and `TF_CLI_ARGS_*` environment variables](https://www.terraform.io/docs/configuration/environment-variables.html#tf_cli_args-and-tf_cli_args_name) automatically. This makes it easy to toggle settings. 

For example, if you want to pass `-backend-config=bucket=terraform-state-bucket` to `terraform init`, then you would do the following:

```sh
export TF_CLI_INIT_BACKEND_CONFIG_BUCKET=terraform-state-bucket
```

Running `tfenv` will populate the `TF_CLI_ARGS_init=-backend-config=bucket=terraform-state-bucket`

Multiple arguments can be specified and they will be properly concatenated.

### Initializing Modules

Terraform as the built-in capability to initialize "root modules" from a remote sources by passing the `-from-module` argument to `terraform init`.

We can turn this into a 12-factor style invocation using `tfenv`.

```sh
export TF_CLI_INIT_FROM_MODULE=git::git@github.com:ImpactHealthio/terraform-root-modules.git//aws/$(SERVICE)?ref=tags/0.5.7
source <(tfenv)
terraform init
```

Learn more about `TF_CLI_ARGS` and `TF_CLI_ARGS_*` in the [official documentation](https://www.terraform.io/docs/configuration/environment-variables.html#tf_cli_args-and-tf_cli_args_name).




## Examples


### Compiling the Binary

```sh
make go/build
```

### Use with Chamber

```sh
chamber exec foobar -- tfenv terraform plan
```

### Use with Terragrunt

```sh
tfenv terragrunt plan
```

### Print Environment

```sh
tfenv printenv
```

### Use with Direnv

You can easily integrate `tfenv` with [`direnv`](https://github.com/direnv/direnv) so that your environment is automatically setup for Terraform.

Add the following to your `.envrc`:

```
eval $(tfenv sh -c "export -p")
```

<details>
  <summary>Example Output</summary>

  ```
  direnv: loading .envrc
  direnv: export +TF_VAR_aws_profile +TF_VAR_aws_vault_backend +TF_VAR_colorterm +TF_VAR_dbus_session_bus_address +TF_VAR_desktop_session +TF_VAR_direnv_diff +TF_VAR_direnv_in_envrc +TF_VAR_direnv_watches +TF_VAR_display +TF_VAR_fzf_orig_completion_git +TF_VAR_github_token +TF_VAR_gtk_overlay_scrolling +TF_VAR_histcontrol +TF_VAR_histsize +TF_VAR_home +TF_VAR_hostname +TF_VAR_lang +TF_VAR_lessopen +TF_VAR_logname +TF_VAR_ls_colors +TF_VAR_mail +TF_VAR_mate_desktop_session_id +TF_VAR_okta_user +TF_VAR_oldpwd +TF_VAR_path +TF_VAR_pwd +TF_VAR_qt_auto_screen_scale_factor +TF_VAR_qt_scale_factor +TF_VAR_session_manager +TF_VAR_sessiontype +TF_VAR_shell +TF_VAR_shlvl +TF_VAR_ssh_auth_sock +TF_VAR_term +TF_VAR_user +TF_VAR_vte_version +TF_VAR_windowid +TF_VAR_xauthority +TF_VAR_xdg_current_desktop +TF_VAR_xdg_runtime_dir +TF_VAR_xdg_seat +TF_VAR_xdg_session_id +TF_VAR_xdg_vtnr
  ```

</details>





## Share the Love 

Like this project? Please give it a ★ on [our GitHub](https://github.com/cloudposse/tfenv)! (it helps us **a lot**) 

Are you using this project or any of our other projects? Consider [leaving a testimonial][testimonial]. =)


## Related Projects

Check out these related projects.

- [Packages](https://github.com/cloudposse/packages) - Cloud Posse installer and distribution of native apps
- [build-harness](https://github.com/cloudposse/build-harness) - Collection of Makefiles to facilitate building Golang projects, Dockerfiles, Helm charts, and more
- [geodesic](https://github.com/cloudposse/geodesic) - Geodesic is the fastest way to get up and running with a rock solid, production grade cloud platform built on strictly Open Source tools.



## Help

**Got a question?**

File a GitHub [issue](https://github.com/cloudposse/tfenv/issues), send us an [email][email] or join our [Slack Community][slack].

[![README Commercial Support][readme_commercial_support_img]][readme_commercial_support_link]

## Commercial Support

Work directly with our team of DevOps experts via email, slack, and video conferencing. 

We provide [*commercial support*][commercial_support] for all of our [Open Source][github] projects. As a *Dedicated Support* customer, you have access to our team of subject matter experts at a fraction of the cost of a full-time engineer. 

[![E-Mail](https://img.shields.io/badge/email-hello@cloudposse.com-blue.svg)][email]

- **Questions.** We'll use a Shared Slack channel between your team and ours.
- **Troubleshooting.** We'll help you triage why things aren't working.
- **Code Reviews.** We'll review your Pull Requests and provide constructive feedback.
- **Bug Fixes.** We'll rapidly work to fix any bugs in our projects.
- **Build New Terraform Modules.** We'll [develop original modules][module_development] to provision infrastructure.
- **Cloud Architecture.** We'll assist with your cloud strategy and design.
- **Implementation.** We'll provide hands-on support to implement our reference architectures. 




## Slack Community

Join our [Open Source Community][slack] on Slack. It's **FREE** for everyone! Our "SweetOps" community is where you get to talk with others who share a similar vision for how to rollout and manage infrastructure. This is the best place to talk shop, ask questions, solicit feedback, and work together as a community to build totally *sweet* infrastructure.

## Newsletter

Signup for [our newsletter][newsletter] that covers everything on our technology radar.  Receive updates on what we're up to on GitHub as well as awesome new projects we discover. 

## Contributing

### Bug Reports & Feature Requests

Please use the [issue tracker](https://github.com/cloudposse/tfenv/issues) to report any bugs or file feature requests.

### Developing

If you are interested in being a contributor and want to get involved in developing this project or [help out](https://cpco.io/help-out) with our other projects, we would love to hear from you! Shoot us an [email][email].

In general, PRs are welcome. We follow the typical "fork-and-pull" Git workflow.

 1. **Fork** the repo on GitHub
 2. **Clone** the project to your own machine
 3. **Commit** changes to your own branch
 4. **Push** your work back up to your fork
 5. Submit a **Pull Request** so that we can review your changes

**NOTE:** Be sure to merge the latest changes from "upstream" before making a pull request!


## Copyright

Copyright © 2017-2019 [Cloud Posse, LLC](https://cpco.io/copyright)



## License 

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0) 

See [LICENSE](LICENSE) for full details.

    Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at

      https://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.









## Trademarks

All other trademarks referenced herein are the property of their respective owners.

## About

This project is maintained and funded by [Cloud Posse, LLC][website]. Like it? Please let us know by [leaving a testimonial][testimonial]!

[![Cloud Posse][logo]][website]

We're a [DevOps Professional Services][hire] company based in Los Angeles, CA. We ❤️  [Open Source Software][we_love_open_source].

We offer [paid support][commercial_support] on all of our projects.  

Check out [our other projects][github], [follow us on twitter][twitter], [apply for a job][jobs], or [hire us][hire] to help with your cloud strategy and implementation.



### Contributors

|  [![Erik Osterman][osterman_avatar]][osterman_homepage]<br/>[Erik Osterman][osterman_homepage] |
|---|


  [osterman_homepage]: https://github.com/osterman
  [osterman_avatar]: https://github.com/osterman.png?size=150



[![README Footer][readme_footer_img]][readme_footer_link]
[![Beacon][beacon]][website]

  [logo]: https://cloudposse.com/logo-300x69.svg
  [docs]: https://cpco.io/docs
  [website]: https://cpco.io/homepage
  [github]: https://cpco.io/github
  [jobs]: https://cpco.io/jobs
  [hire]: https://cpco.io/hire
  [slack]: https://cpco.io/slack
  [linkedin]: https://cpco.io/linkedin
  [twitter]: https://cpco.io/twitter
  [testimonial]: https://cpco.io/leave-testimonial
  [newsletter]: https://cpco.io/newsletter
  [email]: https://cpco.io/email
  [commercial_support]: https://cpco.io/commercial-support
  [we_love_open_source]: https://cpco.io/we-love-open-source
  [module_development]: https://cpco.io/module-development
  [terraform_modules]: https://cpco.io/terraform-modules
  [readme_header_img]: https://cloudposse.com/readme/header/img?repo=cloudposse/tfenv
  [readme_header_link]: https://cloudposse.com/readme/header/link?repo=cloudposse/tfenv
  [readme_footer_img]: https://cloudposse.com/readme/footer/img?repo=cloudposse/tfenv
  [readme_footer_link]: https://cloudposse.com/readme/footer/link?repo=cloudposse/tfenv
  [readme_commercial_support_img]: https://cloudposse.com/readme/commercial-support/img?repo=cloudposse/tfenv
  [readme_commercial_support_link]: https://cloudposse.com/readme/commercial-support/link?repo=cloudposse/tfenv
  [share_twitter]: https://twitter.com/intent/tweet/?text=tfenv&url=https://github.com/cloudposse/tfenv
  [share_linkedin]: https://www.linkedin.com/shareArticle?mini=true&title=tfenv&url=https://github.com/cloudposse/tfenv
  [share_reddit]: https://reddit.com/submit/?url=https://github.com/cloudposse/tfenv
  [share_facebook]: https://facebook.com/sharer/sharer.php?u=https://github.com/cloudposse/tfenv
  [share_googleplus]: https://plus.google.com/share?url=https://github.com/cloudposse/tfenv
  [share_email]: mailto:?subject=tfenv&body=https://github.com/cloudposse/tfenv
  [beacon]: https://ga-beacon.cloudposse.com/UA-76589703-4/cloudposse/tfenv?pixel&cs=github&cm=readme&an=tfenv
