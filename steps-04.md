# 웹엑스 메시지 전송 워크플로우 만들기 #

## 웹엑스 챗봇 만들기 ##

* [Webex Bots](https://developer.webex.com/docs/bots)
* [List rooms](https://developer.webex.com/docs/api/v1/rooms/list-rooms)
* [List messages](https://developer.webex.com/docs/api/v1/messages/list-messages)
* [Send message](https://developer.webex.com/docs/api/v1/messages/create-a-message)


## 깃헙 시크릿 저장 ##

* `WEBEX_TOKEN`: 챗봇 API 키
* `WEBEX_ROOM_ID`: 스페이스 ID


## 웹엑스 메시지 보내기 ##

* [Postman](https://getpostman.com/)

```yaml
name: 'SWM GitHub Actions Basic'

on: push

jobs:
  first-job:
    name: 'First Job'

    runs-on: ubuntu-latest

    steps:
    - name: Send message to Webex space
      shell: bash
      run: |
        curl -x POST \
          https://webexapis.com/v1/messages \
          -H "Authorization: Bearer ${{ secrets.WEBEX_TOKEN }}" \
          -d '{ "roomId": "${{ secrets.WEBEX_ROOM_ID }}", "text": "깃헙 액션에서 보내는 메시지 from ${{ github.repository }}" }'
```
