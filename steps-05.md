# 도커 기반 커스텀 깃헙 액션 만들기 #

## `entrypoint.sh` 설정 ##

```bash
#!/bin/bash

set -e

# Usage function
function usage() {
    cat <<USAGE

    Usage: $0 [-k|--api-key Webex API key] [-r|--room-id Webex Room ID] [-t|--text text body] [-m|--markdown markdown body] [-h|--help]

    Options:
        -k|--api-key:    Webex API key.
        -r|--room-id:    Webex Room ID.
        -t|--text:       Message body in plain text.
        -m|--markdown:   Message body in markdown. It takes precedence over -t|--text.

        -h|--help:       Show this message.

USAGE

    exit 1
}

# Set up arguments
webex_token=""
webex_room_id=""
body_text=""
body_markdown=""

if [[ $# -eq 0 ]]; then
    webex_token=""
    webex_room_id=""
    body_text=""
    body_markdown=""
fi

while [[ "$1" != "" ]]; do
    case $1 in
    -k | --api-key)
        shift
        webex_token=$1
        ;;

    -r | --room-id)
        shift
        webex_room_id=$1
        ;;

    -t | --text)
        shift
        body_text=$1
        ;;

    -m | --markdown)
        shift
        body_markdown=$1
        ;;

    -h | --help)
        usage
        exit 1
        ;;

    *)
        usage
        exit 1
        ;;
    esac

    shift
done

if [[ $webex_token == "" ]]; then
    echo "Webex API Key not found"
    usage

    exit 1
elif [[ $webex_room_id == "" ]]; then
    echo "Webex Room ID not found"
    usage

    exit 1
elif [[ $body_text == "" ]] && [[ $body_markdown == "" ]]; then
    echo "Either text or markdown value must be provided"
    usage

    exit 1
fi

json_template='{"roomId":"%s","text":"%s","markdown":"%s"}\n'
json_body=$(printf "$json_template" "$webex_room_id" "$body_text" "$body_markdown")

# Send message
response=$(curl --location --request POST 'https://webexapis.com/v1/messages' \
    --header "Authorization: Bearer $webex_token" \
    --header "Content-Type: application/json" \
    --data "$json_body")

echo "::set-output name=response::$response"
```

위 파일 작성 후 로컬에서 테스트하기 위해서는 아래 명령어를 순차적으로 입력합니다.

```bash
chmod +x ./entrypoint.sh

webexToken="<Webex API Token>"
webexRoomId="<Webex Room ID>"
bodyText="Message sent directly from local machine"
bodyMarkdown="Message sent directly from **local machine**"

./entrypoint.sh -k $webexToken -r $webexRoomId -t $bodyText -m $bodyMarkdown
```


## `action.yml` 작성 ##

* [GitHub Actions branding cheat sheet](https://haya14busa.github.io/github-action-brandings/)

```yaml
name: Send Webex Message to Room
author: Justin Yoo
description: Send a message to the given room on Webex

branding:
  icon: message-circle
  color: orange

inputs:
  webexToken:
    description: API token for Webex.
    required: true
  webexRoomId:
    description: Space/Room ID on Webex.
    required: true
  bodyText:
    description: Message in plain text format.
    required: false
    default: ''
  bodyMarkdown:
    description: Message in markdown format.
    required: false
    default: ''

outputs:
  response:
    description: Response from Webex

runs:
  using: docker
  image: Dockerfile
  args:
  - --api-key
  - ${{ inputs.webexToken }}
  - --room-id
  - ${{ inputs.webexRoomId }}
  - --text
  - ${{ inputs.bodyText }}
  - --markdown
  - ${{ inputs.bodyMarkdown }}
```


## `Dockerfile` 설정 ##

```dockerfile
FROM ubuntu:focal

LABEL "com.github.actions.name"="Send Webex Message to Room"
LABEL "com.github.actions.description"="Send a message to the given room on Webex"
LABEL "com.github.actions.icon"="message-circle"
LABEL "com.github.actions.color"="orange"

LABEL "repository"="http://github.com/justinyoo/send-webex-message-to-room"
LABEL "homepage"="http://github.com/justinyoo"
LABEL "maintainer"="Justin Yoo <no-reply@aliencube.com>"

# Install curl
RUN apt-get update && apt-get install -y \
    sudo \
    curl \
 && rm -rf /var/lib/apt/lists/*

ADD entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
```

위와 같이 도커 이미지를 정의한 후 로컬에서 테스트하려면 아래와 같이 순차적으로 명령어를 입력합니다.

```bash
docker build . -t swm-gha
docker run -it swm-gha -k $webexToken -r $webexRoomId -t $bodyText -m $bodyMarkdown
```


## 깃헙 액션 워크플로우 작성 ##

```yaml
name: 'SWM GitHub Actions Basic'

on: push

jobs:
  first-job:
    name: 'First Job'

    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Send message to Webex space
      uses: ./
      with:
        webexToken: ${{ secrets.WEBEX_TOKEN }}
        webexRoomId: ${{ secrets.WEBEX_ROOM_ID }}
        bodyText: "Message sent by GitHub Actions from ${{ github.repository }}"
        bodyMarkdown: "Message sent by **GitHub Actions** from `${{ github.repository }}`"
```
