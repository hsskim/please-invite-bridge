# Please! 초대 링크 브리지

[Please!](https://github.com/hsskim/project-please) 앱의 가족 초대 링크가 도착하는 페이지다.
`https://invite.yellodevs.space/?token=...` 으로 들어오면 앱을 열어준다.

## 왜 있나

앱의 초대 링크는 원래 커스텀 스킴(`please://invite?token=...`)이었는데, **안드로이드
카카오톡에서 눌리지 않았다.** 이유는 둘이다.

1. 채팅의 자동 링크 변환은 `http(s)://` 위주라 `please://…`는 텍스트로 남아 누를 수조차 없다
2. 눌려도 카카오톡은 인앱 WebView로 여는데, 안드로이드 WebView는 모르는 스킴을
   시스템으로 넘기지 않고 실패한다(`ERR_UNKNOWN_URL_SCHEME`)

iOS가 되는 건 앱이 잘 만들어져서가 아니라 OS가 커스텀 스킴을 일급으로 다루기 때문이다.

이 페이지가 https 한 단계를 끼워 그 문제를 없앤다. **앱이 실제로 받는 형식은 그대로
`please://invite?token=...` 이다** — 앱의 되읽는 쪽(`invite-token.ts`)은 손대지 않았다.

## 하는 일

| 상황 | 동작 |
|---|---|
| 안드로이드 | `intent://` 문법으로 리다이렉트 (인앱 WebView가 처리하는 유일한 방법) |
| iOS | `please://invite?token=...` 로 리다이렉트 |
| 앱 없음 (안드로이드) | `S.browser_fallback_url` 로 Play 스토어 |
| 토큰 없음·깨짐 | 안내 문구 |

자동 이동이 실패해도 "앱에서 열기" 버튼이 남아 사용자가 직접 누를 수 있다. 카카오톡
인앱 브라우저는 버전에 따라 `intent://`도 막으므로 "다른 브라우저로 열기" 안내도 함께 둔다.

## 앞으로 (App Links / Universal Links)

지금은 브리지가 커스텀 스킴으로 넘긴다. 커스텀 스킴은 **아무 앱이나 같은 스킴을 등록해
가로챌 수 있고**, 이 링크에는 가족 참여 자격인 초대 토큰이 실려 있다. 도메인 소유를
증명하는 방식으로 옮겨야 한다.

- `/.well-known/assetlinks.json` (안드로이드) — 앱 서명 키 SHA-256:
  `ec377c88d8940a52fbedd6877e731acb0e72cc518e0e08505f61b60ccc8b433a`
- `/.well-known/apple-app-site-association` (iOS)
- 앱 쪽 `app.json`에 `android.intentFilters`(`autoVerify`)와 `ios.associatedDomains` 추가 —
  **네이티브 변경이라 재빌드가 필요하다**

두 파일을 여기 올려도 앱이 선언하기 전까지는 아무 일도 하지 않으므로, 앱 빌드와 순서를
맞출 필요는 없다.

**그때도 링크 주소는 바뀌지 않는다** — 이미 나간 초대가 계속 동작한다.

관련: hsskim/project-please#312
