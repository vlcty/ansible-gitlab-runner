# ansible-gitlab-runner

Installs the [official GitLab Runner](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner) on CentOS 7 machines using docker as executor.

This is a forked and adapted repository from [haroldb/ansible-gitlab-runner](https://github.com/haroldb/ansible-gitlab-runner).

## Variables

These are the available variables:

| Variable | Description | Type | Required | Default value |
|---|---|---|---|---|
| gitlab_runner_concurrent | How many parallel jobs the runner should work on | Integer | Yes | Numer of CPU cores ansible detects |
| gitlab_runner_coordinator_url | The URL to the coordinator. You can get it from the GitLab UI | String | Yes | https://gitlab.com/ci |
| gitlab_runner_registration_token | The register token you get from the GitLab UI | String | Yes | None |
| gitlab_runner_description | The description of the runner (visible in the GitLab UI) | String | No | The inventory hostname |
| gitlab_runner_docker_image | The default docker image to use when not specified in the CI file | String | No | base/archlinux |
| gitlab_runner_tags | The tags associated with the runner | List of strings | No | List containing only 'docker' |
| gitlab_runner_allow_untagged | Allow this runner to take untagged jobs | Bool | No | false |
| gitlab_runner_privileged | Should docker images run privileged | Bool | No | false |

## Implementation suggestion

You probably want to set up more than one runner. I suggest creating a group `gitlab-runner` and add every runner into it. The group variables for that group should contain:

* gitlab_runner_coordinator_url
* gitlab_runner_registration_token

The rest config should be done using more groups or individual host variables.

## Security consideration running privileged containers

Backstory:   
I wanted to build docker containers inside a docker container. That's not possible without a privileged container. GitLab runner can start privileged containers if desired. However for most of the tasks it shoul not be necesarry. Therefore the default value for that is false.

Consideration:   
If strongly suggest using tags to prevent running a job which doesn't need a privileged container on a runner which spawns only privileged containers. Privileged containers circumvent the security settings enforced by docker.

If you haven't set your own value for `gitlab_runner_tags` and `gitlab_runner_privileged` then the default tag for every runner is `docker` and it only spawns unprivileged containers.   
I suggest overwriting the variable using host vars (or group vars if you have more privilged runners) and set new defaults.   
Set `docker-privileged` for `gitlab_runner_tags` and `gitlab_runner_privileged` to `true`.

In your `.gitlab-ci.yml` you then set for every job the keyword `tags` to `docker`. You are on the safe side. No job can run privileged by accident.   
If your job requires a privileged container then simply set the keyword `tags` to `docker-privileged`. No job can then run unprivileged by accident.

## Running a runner without tags

If you are in need of a privileged container I stringly suggest not to run untagged jobs. Chances are high that containers not needing privileges run privileged. See the security consideration chapter.

If you don't run privileged containers then you could set this option. But in the future you maybe stumble over problems and please don't blame me then.
