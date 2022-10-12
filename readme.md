Create a new Gitlab Variable `SLACK_WEBHOOK` with your slack webhook.

Replace `<CHANNEL>` and other bits for your needs.

`touch SUCCESS` is a workaround for: `https://gitlab.com/gitlab-org/gitlab-runner/-/issues/27693` and `https://gitlab.com/gitlab-org/gitlab/-/issues/340075`

`.gitlab-ci.yml` example:

```
stages:
  - build

build:
  image: <IMAGE>
  stage: build
  script:
    - ls -alh
    - touch SUCCESS
  after_script:
    - if [[ -f "SUCCESS" ]]; then EXIT_STATUS=0; else EXIT_STATUS=1; fi
    - source ./slacknotification.sh $SLACK_WEBHOOK
```


`slacknotification.sh` example

```
#!/bin/bash

function print_slack_summary_build() {

if [[ "${EXIT_STATUS}" == 0 ]]; then
        slack_msg_header=":heavy_check_mark: *Build succeeded*"
        else
        slack_msg_header=":x: *Build failed*"
    fi

cat <<-SLACK
            {   "channel": "#<CHANNEL>",
                "blocks": [
                    {
                        "type": "section",
                        "text": {
                            "type": "mrkdwn",
                            "text": "${slack_msg_header}"
                        }
                    },
                    {
                        "type": "divider"
                    },
                    {
                        "type": "section",
                        "fields": [
                            {
                                "type": "mrkdwn",
                                "text": "*Stage:*\n${CI_JOB_NAME}\n"
                            },
                            {
                                "type": "mrkdwn",
                                "text": "*Author:*\n${GITLAB_USER_NAME}\n"
                            },
                            {
                                "type": "mrkdwn",
                                "text": "*Job URL:*\n${CI_JOB_URL}\n"
                            },
                            {
                                "type": "mrkdwn",
                                "text": "*Project Name:*\n${CI_PROJECT_NAME}\n"
                            },
                            {
                                "type": "mrkdwn",
                                "text": "*Commit Branch:*\n${CI_COMMIT_REF_NAME}\n"
                            }
                        ]
                    },
                    {
                        "type": "divider"
                    }
                ]
}
SLACK
}

curl -X POST --data-urlencode "payload=$(print_slack_summary_build)" $1
```
