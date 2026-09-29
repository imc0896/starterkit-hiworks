This is a [Next.js](https://nextjs.org) project bootstrapped with [`create-next-app`](https://nextjs.org/docs/app/api-reference/cli/create-next-app).

## Getting Started

First, run the development server:

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
# or
bun dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

You can start editing the page by modifying `app/page.tsx`. The page auto-updates as you edit the file.

This project uses [`next/font`](https://nextjs.org/docs/app/building-your-application/optimizing/fonts) to automatically optimize and load [Geist](https://vercel.com/font), a new font family for Vercel.

## Learn More

To learn more about Next.js, take a look at the following resources:

- [Next.js Documentation](https://nextjs.org/docs) - learn about Next.js features and API.
- [Learn Next.js](https://nextjs.org/learn) - an interactive Next.js tutorial.

You can check out [the Next.js GitHub repository](https://github.com/vercel/next.js) - your feedback and contributions are welcome!

## Deploy on Vercel

The easiest way to deploy your Next.js app is to use the [Vercel Platform](https://vercel.com/new?utm_medium=default-template&filter=next.js&utm_source=create-next-app&utm_campaign=create-next-app-readme) from the creators of Next.js.

Check out our [Next.js deployment documentation](https://nextjs.org/docs/app/building-your-application/deploying) for more details.

## 하이웍스 메일 MCP 서버

`mcp-servers/hiworks-mail/`에는 하이웍스 메일을 Claude Code에서 조회·발송할 수 있는 MCP 서버가 들어 있습니다.
[hiworks-mail-mcp@1.0.12](https://www.npmjs.com/package/hiworks-mail-mcp)(MIT)를 기반으로 수정한 버전입니다.

- 계정 정보는 환경변수로만 읽습니다. 도구 입력 schema나 응답에 비밀번호가 노출되지 않습니다.
- 첨부파일 목록 조회와 로컬 저장을 지원합니다.
- 날짜는 KST(`+09:00`) 기준으로 표시합니다.

### 설치

```bash
cd mcp-servers/hiworks-mail
npm install
```

### 계정 설정

서버는 아래 환경변수에서 하이웍스 계정을 읽습니다.

| 환경변수 | 필수 | 설명 |
|---|---|---|
| `HIWORKS_USERNAME` | ✅ | 하이웍스 메일 주소 (예: `user@company.com`) |
| `HIWORKS_PASSWORD` | ✅ | 하이웍스 비밀번호 |
| `HIWORKS_ATTACHMENT_DIR` | | 첨부파일 저장 폴더 (기본: `downloads/hiworks`, 상대경로는 프로젝트 루트 기준) |
| `NODE_ENV` | | `development`로 설정하면 stderr에 디버그 로그 출력 |

`.mcp.json`은 `${HIWORKS_USERNAME}`, `${HIWORKS_PASSWORD}`를 참조하므로, 셸 환경변수로 설정하거나
`.claude/settings.local.json`의 `env`에 넣으면 됩니다.

```json
{
  "env": {
    "HIWORKS_USERNAME": "user@company.com",
    "HIWORKS_PASSWORD": "********"
  }
}
```

> ⚠️ `.claude/settings.local.json`에는 비밀번호가 평문으로 저장됩니다. 이 파일이 git에 커밋되지 않도록
> `.gitignore`에 포함되어 있는지 반드시 확인하세요. (`git check-ignore .claude/settings.local.json`)

하이웍스 관리 설정에서 외부 메일 프로그램(POP3/SMTP) 사용이 차단되어 있으면 연결되지 않습니다.
서버 주소는 `pop3s.hiworks.com:995`(수신), `smtps.hiworks.com:465`(발신)입니다.

### Claude Code 연결

프로젝트 루트의 `.mcp.json`에 이미 등록되어 있습니다. Claude Code를 실행한 뒤 `/mcp`에서
`hiworks-mail-mcp`가 연결되었는지 확인하세요. 서버 코드를 수정했다면 `/mcp`에서 다시 연결하거나 Claude Code를 재시작해야 반영됩니다.

### 제공 도구

| 도구 | 입력 | 설명 |
|---|---|---|
| `read_username` | 없음 | 설정된 계정(username)을 확인합니다. |
| `search_email` | `limit?`, `query?` | 최근 `limit`건(기본 100) 중 제목·보낸사람·받는사람에 `query`가 포함된 메일을 최신순으로 반환합니다. 대소문자는 구분하지 않습니다. |
| `read_email` | `messageId`, `includeHtml?` | 메일 본문과 첨부파일 목록을 반환합니다. `messageId`는 `search_email` 결과의 `id` 또는 POP3 메시지 번호입니다. `includeHtml: false`면 HTML 원문을 빼서 토큰을 절약합니다. |
| `save_attachment` | `messageId`, `filename?` | 첨부파일을 로컬 폴더에 저장합니다. `filename`을 생략하면 전체를 저장하고, 같은 이름이 있으면 `이름 (2).확장자`로 저장합니다. |
| `send_email` | `to`, `subject`, `text?`, `html?`, `cc?`, `bcc?`, `attachments?` | 설정된 계정으로 메일을 발송합니다. |

### 사용 예시

Claude Code에서 자연어로 요청하면 됩니다.

```
오늘 온 메일 중 "출고요청" 들어간 메일 찾아줘
가장 최근 메일 본문 텍스트만 읽어줘
그 메일 첨부파일 저장해줘
```

### 제한사항

- POP3 방식이라 **받은편지함만** 조회할 수 있습니다. 보낸편지함이나 다른 폴더는 볼 수 없습니다.
- 서버 측 검색이 없어서 `search_email`은 최근 `limit`건의 헤더를 받아 로컬에서 필터링합니다. `limit`이 클수록 느려집니다.
- `read_email`, `save_attachment`는 Message-ID로 찾을 때 최근 100건까지만 탐색합니다. 더 오래된 메일은 POP3 메시지 번호로 지정하세요.
