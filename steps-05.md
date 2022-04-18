# 도커 기반 커스텀 깃헙 액션 만들기 #

## `entrypoint.ps1` 설정 ##

```powershell
Param(
    [string]
    [Parameter(Mandatory=$true)]
    $WebexToken,

    [string]
    [Parameter(Mandatory=$true)]
    $WebexRoomId,

    [string]
    [Parameter(Mandatory=$false)]
    $BodyText = "",

    [string]
    [Parameter(Mandatory=$false)]
    $BodyMarkdown = ""
)

$headers = @{ Authorization = "Bearer $WebexToken"; }
$body = @{ roomId = $WebexRoomId; text = $BodyText; markdown = $BodyMarkdown }

$response = Invoke-RestMethod `
    -Method Post `
    -Uri https://webexapis.com/v1/messages `
    -Headers $headers `
    -Body $body | ConvertTo-Json -Depth 100

Write-Output "::set-output name=response::$response"

Remove-Variable response
Remove-Variable body
Remove-Variable headers
```


## `action.yml` 작성 ##

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
  - -WebexToken
  - ${{ inputs.webexToken }}
  - -WebexRoomId
  - ${{ inputs.webexRoomId }}
  - -BodyText
  - ${{ inputs.bodyText }}
  - -BodyMarkdown
  - ${{ inputs.bodyMarkdown }}
```


## `Dockerfile` 설정 ##

* [GitHub Actions branding cheat sheet](https://haya14busa.github.io/github-action-brandings/)

```dockerfile
FROM mcr.microsoft.com/azure-powershell:latest

LABEL "com.github.actions.name"="Send Webex Message to Room"
LABEL "com.github.actions.description"="Send a message to the given room on Webex"
LABEL "com.github.actions.icon"="message-circle"
LABEL "com.github.actions.color"="orange"

LABEL "repository"="http://github.com/justinyoo/send-webex-message-to-room"
LABEL "homepage"="http://github.com/justinyoo"
LABEL "maintainer"="Justin Yoo <no-reply@aliencube.com>"

ADD entrypoint.ps1 /entrypoint.ps1
RUN chmod +x /entrypoint.ps1

ENTRYPOINT ["pwsh", "-File", "/entrypoint.ps1"]
```
