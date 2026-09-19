# 사례 3 원시 자료 아카이브 · Suno 음악 생성의 비동기 상태 관리(Suno 音乐生成的异步状态管理)

> 본 문서는 **증거 아카이브**이며, 논문 본문이 아니다. 목적은 사례 1·사례 2와 동일하다: 제5장 사례 3의 모든 진술을 제3자가 검증할 수 있게 하는 것이다.
> 본 문서는 `docs/thesis/05-collaboration-cases.md`, `05-case1-*.md`, `05-case2-*.md`를 **수정하지 않으며**, 어떤 소스 코드, 마이그레이션, 프롬프트 부록도 변경하지 않는다.
>
> **검증 기준 버전**: 저장소 HEAD = `4dbd6bfd`(2026-08-05 11:31). 본 문서의 모든 행 번호는 이 버전에서 `rg -n`으로 현장 검증한 결과이다.
> **대화 아카이브 기준**: 본 프로젝트의 채팅 아카이브는 현재 총 **2258개**의 사용자 가시 메시지를 포함한다(사용자 메시지 + AI 답변 본문; 도구 호출과 도구 출력은 아카이브에 포함되지 않아 검색 불가).
>
> **본 사례가 다루는 것은 바로 플랫폼이 현재 사용 중인 링크이다**: `sg-suno-generate → sg-suno-poll → sg-suno-lyrics → video-trigger →(GitHub Actions / `yuuisohe-ui/melody-video-renderer` 저장소 ffmpeg 합성 + YouTube 업로드)→ video-callback → sg-finalize-song`. 이 함수들과 호출 지점이 HEAD에서 모두 존재하며 `src/components/songs/YoutubeVideoGenerateDialog.tsx`에서 실제로 호출됨을 파일 단위로 확인했고, 폐기된 브랜치가 아니다.

---

## 1. 시간 창 경계

| 항목 | 값 | 설명 |
| --- | --- | --- |
| 창 시작점 | **메시지 #1478, 2026-05-19 05:54 (UTC)** | "비동기 작업 실패 시 자동 전환/계속 진행"을 요구한 첫 번째 사용자 요구(1번: Suno 키 잔액 부족 시 다음 키로 전환). 이것이 직접적으로 `b65ef32b` / `a8ab129f`(이중 키 장애 전환, 05-19 06:18)를 산출했다. |
| 창 종점 | **메시지 #1612, 2026-05-30 14:37 (UTC)**; 본 사례 핵심 파일의 마지막 기능성 커밋 **`57472a96`(다이얼로그)과 `49e623dd`(`sg-finalize-song`), 둘 다 2026-05-30 14:40 (UTC)**에 해당 | 이후 핵심 파일에는 두 종류의 비기능성 변경만 남아 있다. 아래 "창 말미 제외 항목" 참조. |
| 창 길이 | **11일 8시간 43분**(2026-05-19 05:54 → 2026-05-30 14:37 UTC) | |
| 실제 활동일 | **7일**: 05-19, 05-23, 05-24, 05-25, 05-26, 05-27, 05-28, 05-30(05-20~05-22에는 해당 링크의 커밋이 0건이며 증빙 가능) | 중단 기간에 이 링크의 파일에 어떤 커밋도 없었음은 `git log --since/--until`로 재검증 가능 |
| 창 내 당사(사용자) 항목 | **본 아카이브에 33건을 일일이 열거**(자연어 요구 17건 + 오류 보고 템플릿 2건 + 승인/빈 메시지 14건) | 제2절 참조; 선정 규칙은 제2절 서두 참조 |

**경계 설정 설명(독자가 범위를 오해하지 않도록)**

1. 본 사례는 **비동기 상태 관리**라는 하나의 선만 다룬다: 외부 장시간 작업(Suno 작곡 1~3분, GitHub Actions 렌더링 + YouTube 업로드 수 분)이 상태 비저장(stateless) Edge Function과 언제든 닫힐 수 있는 브라우저 사이에서 어떻게 표현되고, 영속화되고, 복구되며, 실패 시 폴백되는가.
2. 그보다 이른 **05-18** 구간(`c9db6953` / `8cbdb0bb` / `26a8ec0a` / `803abb1c`, 05-18 15:54–15:57에 Suno 클라이언트와 세 개의 Edge Function 설립)은 이 링크의 **탄생 지점**이지만, 그 라운드의 사용자 메시지 주제는 "AI 노래 생성 기능 전체 출시"였고, 그중 타임라인 정렬 부분은 이미 **사례 1**(창 #1452–#1484)이 점유했다. 두 아카이브가 같은 메시지 묶음을 중복 인용하지 않도록, 본 아카이브는 05-18 설립기를 **전제 사실**로 제3절에서 설명하고, 제2절의 메시지 목록에는 포함하지 않으며, 제4절의 파일 단위 통계에서만 사실대로 표기한다.
3. **창 말미 제외 항목(검증 완료, 모두 비기능성 변경)**:
   - `0748a400` / `3642b743` / `8efadef7` / `cf520900`(2026-06-21 17:08–17:09, `src/pages/SongGenerator.tsx`): `data-tour="sg-form"` 등 투어 앵커와 `useGuideTour()` 호출만 추가한 것으로, "현장 체험" 가이드 모듈에 해당.
   - `09aec735`(2026-07-17 09:04, `YoutubeVideoGenerateDialog.tsx`): 버튼 문구 한 줄 `YouTube 영상 생성 (AI 노래)` → `AI로 나만의 곡 만들기`만 변경.
   둘 다 상태 기계를 변경하지 않으므로 창 종점을 7월까지 연장하지 않는다.

---

## 2. 창 내 당사(사용자) 메시지 · 원문 그대로

> **출처 표기: 이 절에서 「원문」으로 표기한 부분은 100% 저장된 대화 아카이브에서 직접 추출한 것이다**(`chat_search--read_chat_messages`, 메시지 번호별 판독; 의역, 윤문, 오타 및 한중 혼용 구두점 수정 없음). 아카이브에 `[Empty message]`로 표시된 항목도 그대로 보존한다.
>
> **선정 규칙(반드시 명시해야 함. 그렇지 않으면 독자가 창 내 메시지가 이것뿐이라고 오해할 수 있다)**: 본 창은 11일에 걸치며, 그 사이 다른 모듈도 병행으로 진행됐다. 이 절은 창 내에서 **Suno/렌더링/콜백/AI 노래 카드 링크와 관련된** 사용자 항목 **전부**를 나열한다. 같은 창 내에서 검증 결과 **이 링크와 무관한** 사용자 메시지는 이 절 말미의 "창 내에서 제외된 사용자 메시지"에서 하나씩 지명하여 누락이 아님을 증명한다.
>
> ⚠️ **보안 처리 선언**: #1532 원문에는 GitHub 개인 액세스 토큰 평문이 포함되어 있다. 본 아카이브는 이를 **`github_pat_***REDACTED***`로 마스킹**한다. 이것은 본 아카이브에서 원문에 가한 유일한 변경이며, 특별히 선언한다. 이 토큰은 유출된 자격 증명으로 간주해야 하며, 논문 본문에서 이 메시지를 인용할 때도 반드시 마스킹해야 한다.

---

### 제1단락 · 2026-05-19: 이중 API 키 장애 전환

**#1478 · 2026-05-19 05:54 UTC**

> 1.帮我再添加一个suno api的密匙，当现在这个出现费用不足的情况时，切换成下一个密匙。2.当我用ai生成这个canvas后，我能把这个当成视频内容自动上传到我的YouTube账号上吗，3.当我在ai 맞춤 노래 생성中选择 한국어的时候卡片是可以变更为蓝色 kr的吧？한중 이중언어这个你看怎么加比较好。4。由ai生成歌曲出现的音乐卡片中，어휘右边出现主题标签，这个主题和文化标签等同样要保存到数据库中，并且及时反映在통계보기 中。5.目前我删掉音乐卡片后，통계보기里面即使刷新，也没有显示当下的歌曲状况……

- **본 사례와 관련된 지점**: 1번(키 장애 전환), 2번(**"YouTube 자동 업로드" 최초 제안** — 이것이 이후 렌더링 링크 전체의 출발점).
- **최인접 관련 커밋**: `b65ef32b`(05-19 06:18:23, `sg-suno-generate/index.ts`, +40/−10), `a8ab129f`(05-19 06:18:26, `sg-suno-poll/index.ts`, +38/−7), `b54fd82c`(05-19 06:18, `sg-suno-lyrics/index.ts`). 세 커밋이 같은 배치로 `isCreditError()` + `sunoCall()` 이중 키 로테이션을 도입.

**#1480 · 2026-05-19 06:06 UTC**(구조화된 답변 카드)

> **YouTube 업로드**: Other: 只告诉我可以实现的详细计划就可以，这次不用实现
> **한중 이중언어**: 보라색 'KR/CN' 듀얼 배지
> **테마 태그 출처**: 가사 분석 시 GPT가 자동 추출 → DB에 저장
> **휴지통과 통계**: Other: 在统计、图表、计数、筛选、搜索中，都不要显示回收站里的项目……

- **전환의 의미**: 사용자가 YouTube 업로드를 **"계획만 수립, 이번 라운드에서는 미구현"으로 명시적으로 연기**했다. 이것이 `video-trigger`가 5일 뒤(05-24 08:56)에야 착수된 이유를 설명한다.

**#1482 · 2026-05-19 06:15 UTC** → `[Empty message]`
**#1484 · 2026-05-19 06:17 UTC** → `[Empty message]`

- **인접 커밋**: 이중 키 3연속 커밋(06:18).

---

### 제2단락 · 2026-05-24: Suno 크레딧 오류 보고 → GitHub 렌더링 링크 착수

**#1524 · 2026-05-24 07:56 UTC**(오류 보고 템플릿, 원문 발췌)

> For the code present, I get the error below. Please think step-by-step in order to resolve it.
> ``` Edge function returned 500: Error, {"error":"Suno did not return taskId. msg=The current credits are insufficient. Please top up."} …

- **인접 AI 응답(#1525, 07:57)**: Suno 측 잔액 문제로 판정하고, 500 예외 대신 **200 + 한국어 가독 안내**로 변경.
- **대응 커밋**: `d04f7c94`(05-24 07:57, `sg-suno-generate/index.ts`, +14/−3). `/credit|insufficient|top up|quota|balance/i` 문구 매칭을 추가하고 `"Suno 크레딧이 부족합니다. sunoapi.org에서 충전 후 다시 시도해 주세요."`를 반환.

**#1526 · 2026-05-24 08:11 UTC**

> 我查了一下我的账号，是有余额的，之前我给你两个账号不知道你有没有替换，重新让我输入一下suno api吧，以后固定先用这个

**#1528 · 2026-05-24 08:14 UTC** → `[Empty message]`(키 재등록에 해당)

**#1530 · 2026-05-24 08:37 UTC**(**핵심: 묻기만 하고 고치지 않음**)

> 刚才新建的YoutubeVideoGenerateDialog.tsx（不知道是不是这个，就是youtube영상생성按钮），不要修改代码，只告诉我：
> 1.Suno生成完音乐后，audio_url和Suno返回的aligned_words在哪个位置、用什么变量名接收的？
> 2.sg-finalize-song保存完成后，songId在哪里接收的？
> 3.现在这个对话框里有没有包含视频生成相关的功能（比如Pixabay背景视频）？有的话调用了哪些函数？
> 4.保存完成的那个时间点，audio_url、aligned_words、songId三个都有的代码位置在哪里？"。5.把YoutubeVideoGenerateDialog.tsx的完整流程一步一步讲清楚，每一步调用了什么Edge Function，拿到了什么数据，最后存了什么进数据库。 因为我发现现在这个按钮和我之前做的那个功能完全一致了，所以我可能会断掉一点功能接新的功能。我现在想的是接收到suno的歌词和时间戳和音频之后，然后接→ 调用video-trigger → 触发GitHub Actions → ffmpeg合成视频→ 上传YouTube → video-callback把链接存回数据库→ 前端用youtube url和歌词和时间戳进行音乐卡片教学分析 。所以希望你把现在的详细流程具体储存全部告诉我，并告诉我我接下来要这么做的话你有什么计划

**#1532 · 2026-05-24 08:49 UTC**(**핵심: 사용자가 완전한 구현 방안을 직접 제시**, 원문 발췌, 토큰 마스킹 처리됨)

> 지금 YouTube 영상 자동 생성 기능을 추가하려고 합니다.
> 흐름은 이렇습니다: Suno로 노래 생성 완료 → audio_url + aligned_words + song_id 확보 → video-trigger Edge Function 호출 → GitHub Actions 트리거 → ffmpeg로 가사 영상 합성 → YouTube 업로드 → video-callback으로 youtube_url을 DB에 저장 → 노래 카드에서 YouTube 영상 + 가사 타임스탬프로 교학 분석
> 1.songs 테이블에 video_status, video_error 컬럼 추가, render_jobs 테이블 신규 생성
> 2 video-trigger, video-callback Edge Function 2개 신규 생성
> 3.YoutubeVideoGenerateDialog.tsx에서 sg-pixabay-videos 호출 제거, sg-finalize-song으로 song_id 받은 직후 video-trigger 호출 추가
> 4.Supabase Edge Function 환경변수에 GH_TOKEN 추가
> （随后附上约 120 行的 SQL 与两个 Edge Function 的完整 TypeScript 草稿，并以 "나머지 코드는 수정하지 마세요." 收尾；其中 `GH_TOKEN = github_pat_***REDACTED***`）

- **인접 커밋**: `25349c98` / `709717d4`(05-24 08:56, `video-trigger`와 `video-callback` 신규 생성), `0c9d0624`(05-24 08:56, 다이얼로그 배선), 마이그레이션 `20260524085427_64cac0f1-...sql`(05-24 08:54).
- **사용자 초안과의 차이(AI 측 보강, HEAD 코드로 일일이 검증 가능)**:
  1. 마이그레이션에서 `render_jobs`에 `owner_id`, 두 개의 인덱스, **RLS 4개 정책**, `set_render_jobs_owner` / `set_render_jobs_updated_at` 두 개의 트리거가 추가됨(사용자 초안에는 RLS가 전혀 없었음).
  2. `video-trigger`에 `Authorization` 헤더에서 `ownerId`를 파싱하는 로직(`index.ts:22-32`)과 GitHub dispatch 실패 시 **`render_jobs.status='failed'` 및 `songs.video_status='failed'`를 되기록하는 로직**(`index.ts:89-103`)이 추가됨. 사용자 초안은 단순히 `throw`였음.

**#1534 · 08:53, #1536 · 08:55 · 2026-05-24** → 모두 `[Empty message]`(실행 승인)

**#1538 · 2026-05-24 09:12 UTC**

> 1.刚才改的是这个按钮的功能吧？？为什么还是生成了视频？2.YoutubeVideoGenerateDialog.tsx의 handleSaveToArchive 함수 안에 video-trigger 호출 코드가 실제로 들어가 있나요? 해당 코드 부분만 보여주세요. 3.GH_TOKEN이 Supabase Edge Function 환경변수에 실제로 등록되어 있나요?"。为啥我点开GitHub仓库的 Actions 页面也没有自动触发一个新的workflow在跑。

**#1540 · 2026-05-24 09:15 UTC**

> "네, 진행해주세요. YoutubeVideoGenerateDialog.tsx에서 sg-pixabay-videos 호출 3곳 완전 제거하고, video-trigger 호출 성공/실패를 toast로 표시해주세요. 나머지는 수정하지 마세요."

- **대응 커밋**: `5e470460` / `80364e5c` / `f43d9cd3` / `2be20c44`(05-24 09:15–09:16, 다이얼로그).

**#1542 · 2026-05-24 09:43 UTC**(오류 보고 템플릿, 원문 발췌)

> ``` Edge function returned 500: Error, {"error":"GitHub dispatch failed: 403 {\"message\":\"Resource not accessible by personal access token\",\"documentation_url…

- **인접 AI 응답(#1543, 09:44)**: **GitHub PAT 권한 부족**으로 판정(fine-grained PAT는 `Contents: Read and write` 필요, classic PAT는 `repo` scope 필요). **플랫폼 외부 설정** 문제이므로 코드 측에서 수정 불가 → 사용자에게 반환.
- **방법론적 의미**: 본 사례에서 "AI가 문제가 코드에 있지 않고 사람이 외부 시스템의 설정을 바꿔야 함을 명확히 지적"한 분기점으로, 사례 2의 `insufficient_quota` 처리와 상호 증거가 된다.

**#1544 · 09:51** → `[Empty message]`

**#1546 · 2026-05-24 10:22 UTC**(**본 사례에서 가장 결정적인 한 건: 사용자가 흐름 순서를 시정**)

> 1.现在这个流程很不对啊，你在作曲这一步流程就不对，不应该做视频，我们这个按钮的具体流程之前不是跟你说过了吗，是这样：用户点按钮 → 生成歌曲 → 拿到audio_url + aligned_words（已做） → 调用video-trigger → 触发GitHub Actions → ffmpeg合成视频（已做） → 上传YouTube → video-callback把链接存回数据库→ 用YouTube url和带有时间戳的歌词来生成音乐卡片。现在这个流程还在沿用之前那个ai 맞춤 노래 생성按钮，你要……

- **인접 AI 응답(#1547, 10:23)**: 재구성 계획 산출(`plan--show`, 승인 대기) — 다이얼로그를 `form → lyrics → music(Suno) → render(GitHub Actions + YouTube, 폴링) → ready → 노래 카드 생성`의 명시적 단계 기계로 변경.
- **대응 커밋**: `f03b1fda` / `81f99589` / `7b2c8e95` / `f1fded87` / `10d49c35` / `53b4c81c` / `ed6d316a`(05-24 10:25–10:26).

**#1548 · 10:25** → `[Empty message]`(승인)

**#1550 · 2026-05-24 10:45 UTC**(**두 번째 결정적 전환: "한 번에 끝내기"를 독립적으로 재시도 가능한 버튼들로 분할**)

> 能把作曲和向github传递分开吗，因为我要测试gethub的话要一直向他传，一直测试，没有必要反复生成歌曲，生成一次之后，点击向下传，如果失败就可以重新点击。把suno生成歌曲之后的步骤能拆的尽量拆一下 ，尽量让不同的按钮控制不同的步骤

- **대응 커밋**: `c28919e6` / `e08a57d2` / `98735251` / `9adc66f7` / `5153946c` / `a5d40dc1`(10:27–10:28) 및 `77f41ada` / `27078c0a` / `a31f549e` / `69b2f3de` / `e8fa7385` / `61a4945f` / `7f5d6199`(10:48–10:49).
- **코드 반영 지점**: `Step` 유니온 타입에 `"suno_done"` 중간 상태 추가(HEAD `YoutubeVideoGenerateDialog.tsx:52`). 이로써 "Suno 완료"와 "렌더링 개시"가 각각 별도로 트리거 가능하고 실패 시 제자리에서 재시도 가능한 두 동작이 됨. `video-trigger` 실패 시 전체 흐름을 폐기하지 않고 `setStep("suno_done")`으로 후퇴(`:329-331`).

**#1552 · 10:48** → `[Empty message]`

---

### 제3단락 · 2026-05-25: 안정성(콜백 불안정, TEXT_SUCCESS 조기 종료)

**#1554 · 2026-05-25 13:25 UTC**

> Load the security issues from the scan results and fix them.

- 본 링크와의 교집합: 이번 라운드의 보안 수정 배치가 같은 시기에 `sg-finalize-song`에도 적용됨(`70099106` / `80898343` / `cca0a284`, 05-25 13:41).

**#1556 · 2026-05-25 13:33 UTC**(**"render_jobs 5초 폴링"을 제안한 것이 바로 사용자 본인**)

> 目前这个按钮的功能有以下几个问题需要解决：
> 1.YouTube视频生成完成后，有时候能收到youtube_url，但不稳定，偶尔收不到，希望能稳定收到。
> 2.收到youtube_url之后，영상 모듈里的歌词没有随时间滚动，希望用Suno返回的aligned_words时间戳让歌词同步滚动。3.这个生成卡片后，所有内容的储存和分析要和之前从youtube搜索到的歌曲的储存方式和分析方式一样。比如右上角的歌曲仪表盘，比如主题分析，比如文化标签分析等等，之后在讲议案生成的时候也应该能找到新添加的歌曲。
> 下面的两点我不知道有没有必要，你参考一下有没有用：1..用户点击'노래 카드 생성'按钮后，希望songs表里同时保存youtube_url、video_id、aligned_words、歌词，这样歌曲卡片的教学分析功能才能正常工作。
> 2..render阶段需要每5秒轮询render_jobs表，确认status变成done且youtube_url有值后，自动进入ready阶段显示结果。
> 请告诉我目前这些问题出在哪里，然后修复。其他代码不要动。

- **코드 반영 지점(HEAD 검증 가능)**: `YoutubeVideoGenerateDialog.tsx:335-375`의 `render` 단계 폴링 — 주석에 *"Treat as 'done' only when …"*이라고 명시, `setInterval(tick, 5000)`(`:373`), `status==='done'`이고 `youtube_url`에 값이 있어야만 `setStep("ready")`(`:360`), 실패 시 `suno_done`으로 후퇴(`:363-368`). `:377-397`은 두 번째 `render_jobs` 폴러(ready 이후에도 링크를 폴백으로 보완).
- **주의**: 폴링 간격 5초와 "status=done이며 url이 비어 있지 않음"이라는 이중 조건은 **모두 사용자가 #1556에서 직접 지정한 것**이며, AI가 자주적으로 설계한 것이 아니다. 이것은 본 사례에서 "인간의 전문 판단"이 가장 직접적으로 드러나는 물증이다.

**#1558 · 13:39** → `[Empty message]`

**#1560 · 2026-05-25 13:44 UTC**

> 目前这一页的信息有正确传达给gpt api和suno吗，为什么有的时候我选了两分钟，他返回的确实三四分的音频。有的时候我选择了语言的等级去生成歌词，但是歌词难度就不太对。其他标签也都审核一下。

**#1562 · 2026-05-25 13:54 UTC**

> 1.如果suno不能根据选择的时长来生成对应长度的音频的话就取消这个标签。2.还是使用gpt-4o-mini，把提示词稍微更改一下就行，但为了自然，允许偶尔出现1-2个更高等级的单词。3.교학 주제  除了输入，下面新增一些不同的可选的标签，

- **전환의 의미**: **사용자가 외부 서비스가 보장할 수 없는 컨트롤을 자발적으로 삭제**(길이 태그) — "외부 서비스가 할 수 없는 파라미터를 UI에 남겨 사용자를 속이지 말라"는, 비동기 통합에서의 기대치 관리 결정이다.
- **대응 커밋**: `d9bb147d` / `2165cf72` / `1bde8d58`(05-25 13:58, 다이얼로그), `09758b61`(05-25 13:58, `SongGenerator.tsx`).

**#1570 · 2026-05-25 19:23 UTC**

> 刚才生成작은 행복这首歌的时候，"video-trigger Edge Function에서 GitHub repository_dispatch를 호출할 때 client_payload에 aligned_words가 포함되어 있나요? 而且音乐模块也没有时间戳，怎么回事

- **인접 AI 응답(#1571)**: 체인을 추적하여 `aligned_words`가 이미 다이얼로그 303행에서 `video-trigger`에 전달되고 있음을 확인하고, 문제를 Suno 상태 판정이 너무 이르다는 데로 좁힘.

**#1572 · 2026-05-25 19:25 UTC**(**사용자가 함수 단위로 정확한 수정 지시를 제공**)

> "1번과 2번만 수정해주세요: 1.suno-client.ts에서 isTerminalSuccess를 raw === 'SUCCESS' && !!audioUrl일 때만 true로 변경. TEXT_SUCCESS는 계속 폴링. 2.YoutubeVideoGenerateDialog.tsx에서 sg-suno-lyrics 호출 후 alignedWords가 비어있으면 5초 간격으로 최대 6번 재시도. 여전히 비어있으면 toast 경고만 표시하고 계속 진행. 다른 코드는 수정……

- **대응 커밋(세 곳 동일 배치, 05-25 19:26)**:
  - `271df1e4` — `src/lib/song-generator/suno-client.ts`: `(raw === "SUCCESS" || raw === "TEXT_SUCCESS") && !!audioUrl` → `raw === "SUCCESS" && !!audioUrl`(HEAD `suno-client.ts:190`)
  - `f171e45e` — `supabase/functions/sg-suno-poll/index.ts`: 같은 행, 같은 변경
  - `4c2c4528` — 다이얼로그: `for (let attempt = 0; attempt < 6; attempt++)` + `attempt < 5`일 때 `sleep 5000`, 소진 후에는 `toast` 경고만 표시하고 계속 진행(HEAD `:273-289`)
- **방법론적 의미**: 전형적인 "**낙관적 종료 상태 판정**" 결함 — `TEXT_SUCCESS`는 가사가 준비됐다는 뜻일 뿐 오디오는 아직 준비되지 않았다. 수정은 **종료 상태 정의를 조이는 것 + 하류 데이터에 유한 재시도와 소프트 실패를 추가하는 것**이며, 두 수정이 각각 "상태 기계"와 "데이터 무결성" 두 층위에 나뉘어 적용됐다.

**#1574 · 2026-05-26 04:18 UTC**

> "sg-suno-poll에서 TEXT_SUCCESS 수정이 실제로 적용됐나요? 새로 생성한 노래의 aligned_words가 DB에 저장됐는지 확인해주세요."另外，你发送给github的时间戳和目前영상 모듈的时间戳是不是不一样，你是直接把suno按单词的时间戳发给他了是吗，如果我想要这个영상 모듈的时间戳你有什么办法吗，顺便告诉我你这个时间戳是怎么排列分行的，

- **사례 1과의 접점**: 여기가 **사례 1(행 단위 그룹화)과 사례 3(비동기 렌더링)의 교차점**이다 — 렌더링 측이 받는 것은 단어 단위 타임스탬프이고, 플레이어 측이 쓰는 것은 `lines.ts`로 그룹화된 행 단위 타임스탬프이다. 인용 시 상호 표기해야 한다.

---

### 제4단락 · 2026-05-26 ~ 05-27: 산출물 일관성(AI 노래 카드 ≠ YouTube 노래 카드)

**#1576 · 05-26 05:41**

> 为什么별빛의 그리움 这首歌的영상 모듈已经实现了歌词滚动，为什么我点击这个링크 복사的按钮复制链接打开的网址里没有歌词滚动呢

**#1578 · 05-26 05:45** → `[Empty message]`

**#1580 · 05-26 05:59**

> 为什么ai歌曲比如별빛의 그림,点击한국어로 배우기 중국어로 배우기,就会分析不成功呢？或者点击重新分析，会短暂分析成功出现一次，然后我关掉之后再打开内容就又没有了。而且不管是ai生成的歌曲还是从YouTube拿到的歌曲，或者是链接加时间戳的歌曲，核心不都是url生成视频，时间戳歌词生成滚动歌词和教学内容吗，为什么现在音乐卡片的生成会有差别。ai 生成的歌曲，YouTube url 和时间戳你没有保存到数据库吗，

**#1582 · 05-26 06:04** → `위 1·2번만 진행`
**#1584 · 05-26 06:15**

> 我看별빛의 그리움 这首歌已经分析完了，点击这个링크복사的按钮另外打开这个网址发现显示没有数据

**#1586 · 05-26 06:16** → `[Empty message]`
**#1588 · 05-26 06:18** → `分享的内容中还是没有时间戳`

**#1590 · 05-27 17:16**

> ai生成歌曲创造出的音乐卡片和直接拿youtube链接创造的音乐卡片，"한국어 배우기 "的逻辑不一样吗？为什么我点击"첫사랑의 흔적"进行"한국어 배우기" 分析的时候"읽기 "模块里出现的是中文？即便是ai生成音乐后拿回的youtube url进行分析得出的音乐卡片，教学内容和各个按钮的功能应该是一样的。你检查一下，然后修正。你回答我的时候用中文

**#1592 · 05-27 17:34** → `[Empty message]`

**#1594 · 05-27 17:41**(**"별도 체계를 만들지 말라"는 원칙이 가장 명확하게 표명된 지점**)

> 分析的结果很多都是错的，比如影像模块没有拼音，文法里好像也没有中文解释传回等等。你没有直接使用原有的音乐卡片的分析逻辑吗？目前ai生成的歌曲问什么会出现这种情况，同样是根据youtube url 和时间戳和歌词去生成的音乐卡片，为什么分析结果会有不同？你不能再得到youtube url和歌词时间戳之后，分析逻辑完全使用以前进行分析的所有代码吗

**#1596 · 05-27 17:44** → `[Empty message]`

- **최종 반영 지점(05-30로 지연)**: `49e623dd`(05-30 14:40, `sg-finalize-song/index.ts`, +15/−8). 그 diff 주석이 근본 원인을 직접 명시한다:
  > `// 3) Fire-and-forget full analysis. Use the SAME analyze-song pipeline as YouTube-imported songs (the previously-called "analyze-generated-song" function did not exist, so analysis never ran automatically).`
  즉: **이전에는 존재하지 않는 Edge Function `analyze-generated-song`을 호출했고, 비동기 fire-and-forget이 404를 삼켜 버려 "자동 분석"이 장기간 소리 없이 실패**했다. 수정은 `analyze-song` 호출로 변경, 파라미터명 `lyrics` → `custom_lyrics`, 그리고 `"chinese"` 하드코딩 대신 `learnLanguage`/`dbLang`에서 `learn_language`를 도출하는 것이다.

---

### 제5단락 · 2026-05-28: 콜백 인증

**#1598 · 2026-05-28 03:34** → `Load the security issues from the scan results and fix them.`
**#1600 · 2026-05-28 03:50** → `[Empty message]`

- **대응 커밋**: `67a3daba`(05-28 03:50, `video-callback/index.ts`, +13) 공유 비밀 키 검증 신규 추가; `02d9cd6e`(05-28 03:51, `video-trigger/index.ts`) `client_payload`에 `callback_secret` 하달. 같은 시기 마이그레이션 `20260528033659_8c4a77f5-...sql`(profiles 열 단위 권한 부여, user_roles 권한 상승 방지 등).

**#1602 · 2026-05-28 04:14**(**사용자가 "수정이 명실상부한지" 검수식으로 추궁**)

> video-callback Edge Function에서 이제 어떤 방식으로 callback secret을 검증하나요? x-callback-secret 헤더인가요, 아니면 기존 HMAC 방식인가요? 코드 수정하지 마세요.

- **HEAD 사실**: `video-callback/index.ts:14-24`는 **공유 비밀 키 상수 비교**(`x-callback-secret` 또는 `Authorization: Bearer`를 읽어 `VIDEO_CALLBACK_SECRET`과 완전 일치 비교)이며, **HMAC 서명이 아니고**, **타임스탬프 재전송 방지도 없다**.
- ⚠️ **본 아카이브의 정정 의견**: `docs/thesis/05-ai-integration.md` 제5.2.6절은 현재 "`video-trigger` / `video-callback`(`VIDEO_CALLBACK_SECRET` **서명**)"로 기술되어 있으나 코드와 부합하지 않는다 — 실제로는 **공유 비밀 키 비교**이지 서명이 아니다. 논문 본문에서 인용할 때는 코드를 기준으로 해야 한다(여기서는 기록만 하고, 이번 라운드에서 해당 파일을 수정하지 않았다).

---

### 제6단락 · 2026-05-30: 종료 상태 UI와 자동 분석 폐쇄 루프

**#1610 · 2026-05-30 14:09 UTC**(발췌, 1·4·5·6번이 본 사례와 직접 관련)

> 我选中的这个这个按钮有几个功能我想调整一下。1.在到达suno 완료这一步的时候，目前有的内容可以说的再详细一点。比如说aligned，words，这个写成单词量或者什么，用英文可能会听不太懂。audio url应该是歌曲的mps吧？……4.点击영상 렌더 요청之后，右下角弹框很丑，就是那个YouTube영상 렌더 시작这个，不用加这个弹框 5. youtube 영상이 준비되었습니다  这个页面太长了，可以视频小小的放在左边，其他内容放在右边。 而且视频刚生成好时如果出现youtube那边视频还没有上传好的情况，可以简单提示一下说正在上传账号，现在可以直接点击教学卡片生成，如果需要立刻看视频请稍等一下 6.全是视频生成好后，我点击生成音乐教学卡片，卡片倒是正常出现了，但是没有分析好的内容，需要我点进卡片之后再次点击지금분석하기才能开始分析。我记得之前已经实现了自动化教学分析的。按照以前都做好的教学卡片的生成流程自动化进行。不应该因为是ai生成的音乐视频就区别对待。

- **5번의 공학적 의미**: **YouTube 측에 "콜백은 도착했지만 영상은 아직 처리 중"인 최종 일관성(eventual consistency) 창이 존재함을 인정**하고, 해법은 계속 기다리는 것이 아니라 **UI에서 디커플링**하는 것 — 사용자가 즉시 다음 단계(교수 카드 생성)로 진입할 수 있게 하고, "지금 바로 영상을 보고 싶은" 시나리오에만 문구 안내를 한다.
- **대응 커밋**: `cc46f444` / `3e34b2c0` / `20a0db98` / `3260996b` / `8d874c61` / `d7539a95` / `a2f73b75` / `50610b97` / `57472a96`(05-30 14:38–14:40, 다이얼로그, 총 9회) + `49e623dd`(`sg-finalize-song`).

**#1612 · 2026-05-30 14:37 UTC**(창 종점)

> 1，video_keywords → 改为 「배경 영상 검색어 / 背景视频搜索词」，仍展示原始关键词。aligned_words → 改为 「가사 단어 수 / 歌词单词总数」，值显示 alignedWords.length   这两个都不要，直接删除，我是说表面给使用者呈现时的内容直接删除，把MP3 음원 링크 完整显现，这种只要有韩文词就可以了，不用说"/ MP3 音频链接"2. 中文（这个应该是韩文吧？）：**한국어 가사**: 한 줄은 한글 13자(공백 최대 3개)를 넘지 마세요……

---

### 창 내에서 제외된 사용자 메시지(일일이 지명, 누락이 아님을 증명)

| 메시지 | 시각 | 주제 | 제외 이유 |
| --- | --- | --- | --- |
| #1486 | 05-19 06:34 | Pixabay 배경 영상 루프/가용성 자가 점검 | 배경 영상 층에 해당, 비동기 상태 기계 아님; 또한 이 경로는 05-24(#1540)에 전체 제거됨 |
| #1566 | 05-25 15:36 | `reanalyze-wordlist` 400 오류 보고 | 단어 목록 모듈, 본 링크와 무관 |
| #1568 | 05-25 15:46 | `tag-song-culture` 유니크 제약 충돌 | 문화 태그 모듈 |
| #1604 / #1606 | 05-28 16:43 / 16:51 | 사이트 사용 설명 모듈(가이드) 방안 자문 | 설명서 모듈 |
| #1608 | 05-29 06:30 | `[Empty message]`(설명서 모듈 승인) | 상동 |
| #1614 | 05-31 15:01 | 두 버튼 기능 상호 교체 | 이미 창 종점 이후; 또한 진입점 교환이지 상태 기계 변경 아님 |

> 일일이 열거하지 않은 것으로는 창 내의 짝수 번호 **AI 답변 본문**(이 절은 사용자 측 항목만 집계)과 05-20~05-22 사흘이 있다 — `git log --since=2026-05-20 --until=2026-05-23`으로 검증한 결과, 본 링크의 모든 핵심 파일은 이 사흘 동안 **커밋 0건**이다.

---

## 3. 코드 위치 확인(`rg -n` 현장 검증, 기준 HEAD = `4dbd6bfd`)

### 3.0 전제 사실: 링크 탄생기(2026-05-18 15:54–15:57, 제2절 집계에 미포함)

| 커밋 | 시각 (UTC) | 파일 | 내용 |
| --- | --- | --- | --- |
| `803abb1c` | 05-18 15:54 | `src/lib/song-generator/suno-client.ts` | Suno 클라이언트 설립(+237행), **머리 주석에 본 사례의 핵심 설계 제약이 명시됨** |
| `c9db6953` | 05-18 15:55 | `supabase/functions/sg-suno-generate/index.ts` | 개시 함수 설립 |
| `8cbdb0bb` | 05-18 15:55 | `supabase/functions/sg-suno-poll/index.ts` | 폴링 함수 설립 |
| `26a8ec0a` | 05-18 15:55 | `supabase/functions/sg-suno-lyrics/index.ts` | 단어 단위 타임스탬프 취득 함수 설립 |
| `c7427531` | 05-18 15:56 | `supabase/functions/sg-finalize-song/index.ts` | 저장 함수 설립 |
| `cd1b49e3` | 05-18 15:57 | `src/pages/SongGenerator.tsx` | 프런트엔드 페이지 설립 |

**`suno-client.ts:1-9`(아키텍처 주석 원문, 논문 인용문으로 직접 사용 가능)**:

```
// ARCHITECTURE NOTE
//   Suno generation takes 1–3 minutes. Edge Functions are stateless and have
//   short execution limits, so DO NOT loop+sleep here. Each function below
//   performs ONE HTTP call. The caller (browser or scheduler) is responsible
//   for polling.
```

이 주석이 바로 본 사례의 "설계 공리"이다: **상태 비저장·단수명 함수는 장시간 작업을 보유할 수 없으며, 장시간 작업의 상태는 반드시 외부에 둬야 한다**. 이후의 모든 변경은 이것의 추론으로 볼 수 있다.

### 3.1 `src/lib/song-generator/suno-client.ts`(237행; 커밋 2회, +238/−1)

| 행 번호 | 내용 |
| --- | --- |
| `L11` | `const SUNO_BASE = "https://api.sunoapi.org/api/v1"` |
| `L64-72` | `SunoStatusValue` 유니온 타입: `PENDING / TEXT_SUCCESS / FIRST_SUCCESS / SUCCESS / FAILED / ERROR / CREATE_TASK_FAILED` — **공급자 상태의 명시적 열거**가 이후 종료 상태를 조일 수 있었던 전제 |
| `L117-127` | `SunoStatus` 정규화 인터페이스: `raw` / `isTerminalSuccess` / `isTerminalFailure` / `track`, **공급자의 다형태 페이로드를 3값 판정으로 수렴** |
| `L141-167` | `sunoGenerate()`: 단일 POST, 6가지 가능한 필드 위치에서 `taskId` 추출(`data.taskId / data.task_id / data.id / taskId / task_id / id`) |
| `L170-202` | `getSunoStatus()`: 단일 GET, 오디오 주소 폴백 체인 `audioUrl → audio_url → streamAudioUrl → stream_audio_url` |
| `L190` | **`const isTerminalSuccess = raw === "SUCCESS" && !!audioUrl;`** ← `271df1e4`(05-25)의 조임 지점, 원래는 `(raw === "SUCCESS" \|\| raw === "TEXT_SUCCESS")` |
| `L191` | `isTerminalFailure`: `["FAILED","ERROR","CREATE_TASK_FAILED"]` |
| `L205-222` | `getSunoTimestampedLyrics()`: 단어 단위 `alignedWords` + `waveformData` |
| `L228-237` | `sunoError()`: 429 → 한국어 "크레딧이 부족합니다", 401 → "API Key가 유효하지 않습니다" |

### 3.2 `supabase/functions/sg-suno-generate/index.ts`(85행; 커밋 3회, +98/−13)

| 행 번호 | 내용 |
| --- | --- |
| `L11-15`(`isCreditError`, `b65ef32b` 도입) | `status === 401 \|\| 402 \|\| 429` 또는 본문이 `/credit\|insufficient\|quota\|balance\|unauthorized\|invalid api key/`에 히트 |
| `L17-47`(`sunoCall`) | `SUNO_API_KEY`와 `SUNO_API_KEY_2`를 읽고 `for`로 사용 가능한 키를 순회; `isCreditError`에 히트하고 다음 키가 있으면 키를 바꿔 재시도, 아니면 throw |
| `L62-73`(`d04f7c94` 도입) | `taskId`가 없고 `msg`가 크레딧 키워드에 히트 → **HTTP 200 + 한국어 안내**(프런트엔드가 복구 가능한 비즈니스 상태를 시스템 장애로 오인하지 않도록) |

### 3.3 `supabase/functions/sg-suno-poll/index.ts`(74행; 커밋 3회, +82/−8)

- `L1-47`: 3.2와 구조가 대칭인 `isCreditError` + `sunoCall` 이중 키 로직(`a8ab129f`).
- **종료 상태 판정 행**: `isTerminalSuccess = raw === "SUCCESS" && !!audioUrl`(`f171e45e`, 05-25 19:26, `271df1e4`와 같은 분에 착수 — **같은 의미의 수정이 클라이언트 라이브러리와 Edge Function 두 곳에 각각 한 번씩 작성됨**. 이것은 본 사례에서 지적할 수 있는 중복 구현 리스크이다).

### 3.4 `supabase/functions/video-trigger/index.ts`(114행; 커밋 6회, +135/−21)

| 행 번호 | 내용 |
| --- | --- |
| `L22-32` | `Authorization`에서 `ownerId` 파싱(`auth.getClaims`), `render_jobs` 귀속과 RLS에 사용 |
| `L39-52` | service role로 `render_jobs`에 삽입, `status: "dispatched"`, `dispatched_at` |
| `L54-56` | `songs.video_status = "rendering"` 동기화, `video_error` 비우기 |
| `L58-59` | `GH_TOKEN` 누락 시 즉시 throw |
| `L61-87` | `POST https://api.github.com/repos/yuuisohe-ui/melody-video-renderer/dispatches`, `event_type: "render-video"`, `client_payload`에 `job_id / song_id / audio_url / cover_url / song_title / video_keywords / aligned_words / callback_url / callback_secret` 포함 |
| `L81` | `callback_url`은 `SUPABASE_URL`로 동적 조합, 환경 하드코딩 회피 |
| `L82` | `callback_secret: Deno.env.get("VIDEO_CALLBACK_SECRET")`(`02d9cd6e`, 05-28) |
| `L89-103` | dispatch 실패: `render_jobs.status='failed'` + `error_message` + `finished_at`, 그리고 `songs.video_status='failed'` 되기록 — **실패도 종료 상태이므로 반드시 저장해야 함**. 그렇지 않으면 프런트엔드 폴링이 영구히 매달림 |

### 3.5 `supabase/functions/video-callback/index.ts`(87행; 커밋 3회, +96/−9)

| 행 번호 | 내용 |
| --- | --- |
| `L14-24` | 공유 비밀 키 검증: `x-callback-secret` 또는 `Authorization: Bearer`, `VIDEO_CALLBACK_SECRET`과 **완전 일치 비교**; 불일치 시 401 반환(`67a3daba`) |
| `L26-33` | `job_id`와 `status` 필수 검증 |
| `L41-46` | `job_id`로 `song_id` 역조회(**콜백은 `job_id`만 지니고, `song_id`는 서버가 역조회** — 외부가 임의의 노래를 지정할 수 없음. 이것이 05-28 보안 수정의 실질적 의미) |
| `L48-62` | `status === "done"`: `render_jobs` 기록(`youtube_video_id` / `youtube_url` / `finished_at`) + `songs` 기록(`video_status='done'` / `youtube_url` / `video_id`) |
| `L63-76` | 기타 상태: 두 테이블에 `failed` + `error_message` / `video_error` 기록 |

### 3.6 `src/components/songs/YoutubeVideoGenerateDialog.tsx`(747행; 커밋 46회, 그중 **45회가 본 창 내**, +914/−167)

| 행 번호 | 내용 |
| --- | --- |
| `L52` | **`type Step = "form" \| "lyrics" \| "music" \| "suno_done" \| "render" \| "ready" \| "saving"`** — 7상태 명시적 상태 기계. 그중 `suno_done`과 `ready`는 #1546 / #1550 / #1610 세 차례 피드백의 직접적 산물 |
| `L96-106` | 비동기 작업 컨텍스트: `taskId` / `pollStatus` / `audioUrl` / `alignedWords` / `jobId` / `renderStatus` / `youtubeUrl` / `youtubeVideoId` |
| `L124 / L127 / L136` | `localStorage` 기록 / 삭제 / 실행 중 작업 읽기 |
| `L153-154` | 기존 작업 복구: `setStep("music")` + toast `"이전 작곡 작업을 이어갑니다"` — **새로고침 또는 다이얼로그를 닫은 후에도 이어갈 수 있음**, 바로 3.0 아키텍처 주석의 구현 |
| `L169-171` | 복구 경로의 5초 폴링 `sg-suno-poll` |
| `L237` | 메인 경로에서 `sg-suno-generate` 호출 |
| `L257-259` | 메인 경로 5초 폴링 `sg-suno-poll` |
| `L273-289` | `sg-suno-lyrics` 최대 6회, 5초 간격 재시도; 소진 후에는 toast 경고만 하고 `setStep("suno_done")`으로 계속(**소프트 실패**, `4c2c4528`) |
| `L308-331` | `handleRender`: `setStep("render")` → `video-trigger` 호출 → 실패 시 `suno_done`으로 후퇴 |
| `L335-375` | `render` 단계 `render_jobs` 폴링: `setInterval(tick, 5000)`; `done`이고 `youtube_url`이 있어야만 `setStep("ready")`; `failed`면 `suno_done`으로 후퇴 |
| `L377-397` | 두 번째 `render_jobs` 폴러(`ready` 이후 링크를 폴백으로 보완, #1610 5번의 "업로드 중" 창에 대응) |
| `L401-405` | `setStep("saving")` → `sg-finalize-song` |

### 3.7 `supabase/functions/sg-finalize-song/index.ts`(202행; 커밋 9회, +217/−15)

- **`L14-41`**: 행 단위 타임스탬프 병합(사례 1과 공유, 본 사례에서는 상호 참조만 하고 중복 집계하지 않음).
- **`49e623dd` 수정 지점**: fire-and-forget 호출을 존재하지 않는 `analyze-generated-song`에서 `analyze-song`으로 변경; `lyrics` → `custom_lyrics`; `learn_language`를 `learnLanguage`/`dbLang`에서 도출하고 더 이상 `"chinese"`를 하드코딩하지 않음.

### 3.8 데이터베이스 마이그레이션

**`supabase/migrations/20260524085427_64cac0f1-7654-42cc-8de0-ac77a466e693.sql`(05-24 08:54)**

- `ALTER TABLE public.songs ADD COLUMN IF NOT EXISTS video_status text NOT NULL DEFAULT 'idle', ADD COLUMN IF NOT EXISTS video_error text;`
- `CREATE TABLE IF NOT EXISTS public.render_jobs (...)`: `id / song_id(FK ON DELETE CASCADE) / owner_id / status(default 'queued') / audio_url / cover_url / song_title / youtube_video_id / youtube_url / error_message / dispatched_at / finished_at / created_at / updated_at`
- 두 개의 인덱스(`song_id`, `owner_id`); `ENABLE ROW LEVEL SECURITY`; 네 개의 정책 `owned_select / owned_insert / owned_update / owned_delete`(모두 `owner_id = auth.uid() OR public.is_admin()`을 축으로 하며, `select`는 `owner_id IS NULL`을 추가로 허용); 두 개의 트리거 `set_render_jobs_owner`, `set_render_jobs_updated_at`.
- **대조 가치**: 사용자가 #1532에서 제공한 SQL 초안에는 **`owner_id`도, RLS도, 트리거도 없었다**. 세 가지 모두 AI 측 보강이다. 이것은 본 사례에서 "AI가 플랫폼 수준 제약을 보완"한 확실한 증거이다.

**`supabase/migrations/20260528033659_8c4a77f5-04b4-4d8d-952a-60e0279e650e.sql`(05-28 03:36)**: 보안 스캔 배치(profiles 열 단위 권한 부여, `get_my_contact_info()`, `user_roles` 권한 상승 방지 등), `67a3daba`와 같은 배치이지만 `render_jobs`를 직접 변경하지는 않음.

---

## 4. 정량적 수정 라운드 통계

### 4.1 파일 단위(`git log --numstat` 전체 이력 기준)

| 파일 | 전체 이력 커밋 수 | 창 내 커밋 수 | 전체 이력 +/− | 첫 커밋 | 마지막 기능성 커밋 |
| --- | --- | --- | --- | --- | --- |
| `src/lib/song-generator/suno-client.ts` | 2 | 1 | +238 / −1 | `803abb1c` 05-18 15:54 | `271df1e4` 05-25 19:26 |
| `supabase/functions/sg-suno-generate/index.ts` | 3 | 2 | +98 / −13 | `c9db6953` 05-18 15:55 | `d04f7c94` 05-24 07:57 |
| `supabase/functions/sg-suno-poll/index.ts` | 3 | 2 | +82 / −8 | `8cbdb0bb` 05-18 15:55 | `f171e45e` 05-25 19:26 |
| `supabase/functions/sg-suno-lyrics/index.ts` | 2 | 1 | +83 / −6 | `26a8ec0a` 05-18 15:55 | `b54fd82c` 05-19 06:18 |
| `supabase/functions/video-trigger/index.ts` | 6 | 6 | +135 / −21 | `25349c98` 05-24 08:56 | `02d9cd6e` 05-28 03:51 |
| `supabase/functions/video-callback/index.ts` | 3 | 3 | +96 / −9 | `709717d4` 05-24 08:56 | `67a3daba` 05-28 03:50 |
| `supabase/functions/sg-finalize-song/index.ts` | 9 | 8 | +217 / −15 | `c7427531` 05-18 15:56 | `49e623dd` 05-30 14:40 |
| `src/components/songs/YoutubeVideoGenerateDialog.tsx` | 46 | **45** | +915 / −168 | `a155f50a` 05-23 16:12 | `57472a96` 05-30 14:40 |
| `src/pages/SongGenerator.tsx` | 12 | 3 | +576 / −95 | `cd1b49e3` 05-18 15:57 | `09758b61` 05-25 13:58 |
| **합계(9개 파일)** | **86** | **71** | **+2440 / −336** | 05-18 15:54 | 05-30 14:40 |

> 설명: `sg-suno-lyrics`와 `sg-finalize-song`의 일부 변경은 **사례 1** 주제(타임스탬프 행 나누기/병합)에 해당한다. 여기서는 "물리적으로 본 사례의 파일을 수정했다"는 기준으로 사실대로 집계하되, 성질을 여기에 주기하여 두 사례가 같은 코드 양을 중복 주장하지 않도록 한다.

### 4.2 일별 커밋 밀도(본 사례 9개 파일 합계)

| 날짜 (UTC) | 커밋 수 | 주제 |
| --- | --- | --- |
| 05-18 | 6 | 링크 탄생(전제 사실) |
| 05-19 | 7 | 이중 키 장애 전환 + finalize 조정 |
| 05-23 | 1 | 다이얼로그 설립 |
| 05-24 | **30** | 크레딧 오류 폴백 → GitHub 렌더링 링크 → 흐름 순서 시정 → 단계 분할 |
| 05-25 | 13 | 보안 배치 + 5초 폴링 + TEXT_SUCCESS 조임 + 가사 재시도 |
| 05-28 | 2 | 콜백 공유 비밀 키 |
| 05-30 | 10 | 종료 상태 UI 재배치 + 자동 분석 폐쇄 루프 |
| 06-21 / 07-17 | 5 | **비기능성(투어 앵커, 버튼 문구), 제외됨** |

**05-24 하루 30회 커밋**이 전체 창의 최고점이며, 바로 "사용자가 #1530→#1532→#1538→#1540→#1542→#1546→#1550 일곱 건의 시정을 연발"한 당일이다 — **커밋 밀도와 인간 개입 밀도가 고도로 동기화**되어 있으며, 이 점은 제5장 "인간-AI 협업 리듬"의 정량 논거로 직접 사용할 수 있다.

### 4.3 핵심 파라미터 진화표(바로 그래프로 만들 수 있음)

| 차원 | 초기(05-18) | 중기 | 종료 상태(HEAD `4dbd6bfd`) | 트리거 메시지 |
| --- | --- | --- | --- | --- |
| Suno API 키 | 단일 `SUNO_API_KEY` | — | 이중 로테이션 `SUNO_API_KEY` / `SUNO_API_KEY_2` + `isCreditError` | #1478 |
| 크레딧 부족 응답 | HTTP 500 예외 | — | HTTP 200 + 한국어 가독 안내 | #1524 |
| 종료 상태 판정 | `SUCCESS \|\| TEXT_SUCCESS` | — | 오직 `SUCCESS` **그리고** `audioUrl` 비어 있지 않음 | #1572 |
| 단어 단위 타임스탬프 취득 | 단일 호출 | — | 최대 6회 × 5초, 소진 시 소프트 실패(경고만) | #1572 |
| 프런트엔드 단계 | 암묵적(생성 중/완료) | `form/lyrics/music/render/ready` | 7상태, `suno_done` 중간 상태 포함, 실패 시 폐기 대신 후퇴 | #1546 / #1550 |
| 작업 영속화 | 없음 | `localStorage` `sg:active-job` | 좌동 + 복구 toast `"이전 작곡 작업을 이어갑니다"` | 사례 1 창 #1452 3번 |
| 렌더링 상태 매체 | 없음 | `render_jobs` 테이블(사용자 초안, RLS 없음) | `render_jobs` + `owner_id` + RLS 4정책 + 2트리거 + `songs.video_status/video_error` 중복 미러 | #1532 |
| 렌더링 진행 취득 | 없음 | 없음 | 프런트엔드 5초 폴링 `render_jobs`, 이중 조건 종료 상태 판정 | #1556 |
| 콜백 인증 | 없음 | 없음 | 공유 비밀 키 상수 비교(HMAC 아님, 재전송 방지 없음) | #1598 / #1602 |
| Pixabay 배경 영상 | 프런트엔드 3곳 호출 | — | **전부 제거**, GitHub Actions 측 처리로 변경 | #1540 |
| 길이 컨트롤 | UI에 "2분/3분" 있음 | — | **제거**(Suno가 보장하지 않음) | #1562 |
| 생성 후 자동 분석 | `analyze-generated-song` 호출(**존재하지 않음**, 소리 없이 실패) | — | `analyze-song` 호출, YouTube 가져오기 노래와 동일 파이프라인 | #1590 / #1594 / #1610-6 |

---

## 5. 핵심 전환점 주석(보충 설명, 제2절의 완전한 목록을 대체하지 않음)

1. **#1480(05-19 06:06) "계획만 수립, 이번 라운드 미구현"** — 인간이 비동기 링크에 **일정을 배정**하고, 리스크가 가장 높은 외부 의존(GitHub + YouTube)을 5일 연기. 기술적 의미: Suno 측이 아직 안정되지 않은 시점에 두 번째 통제 불가 외부 시스템을 얹는 것을 회피.

2. **#1532(05-24 08:49) 사용자가 SQL + 두 개의 Edge Function 초안을 직접 납품** — 본 사례에서 인간 개입의 세밀도가 가장 깊은 한 차례. 방법론적 의미: "무엇을 원하는가"를 자연어로 무오역 표현하기 어려울 때, 인간은 **코드 층으로 내려가 요구 언어로 삼는다**. 그리고 AI의 가치는 인간 초안에 빠진 플랫폼 수준 제약(`owner_id`, RLS, 실패 저장)을 보완하는 데서 드러난다. 두 역할 분담은 논문 "협업 계층화" 절의 표본이 될 수 있다.

3. **#1542(05-24 09:43) GitHub PAT 403** — AI가 "**문제는 코드가 아니라 저장소 밖의 토큰 권한에 있다**"고 명확히 판정하고 코드 수정을 중지. 사례 2의 `insufficient_quota` 처리와 같은 패턴을 이룬다: **"코드로 해결 가능"과 "사람이 외부 시스템에서 해결해야 함"을 구분**.

4. **#1546(05-24 10:22) "现在这个流程很不对啊"** — 인간이 점별 버그 기술이 아니라 **완전한 흐름 체인**(화살표 8개)을 시정 언어로 사용. 이 한 건이 명시적 `Step` 상태 기계를 직접 산출했다.

5. **#1550(05-24 10:45) "把 suno 生成歌曲之后的步骤能拆的尽量拆"** — **본 사례에서 재사용성이 가장 강한 판단**. 이유는 순수하게 공학적이다: "GitHub를 테스트하려면 계속 전송해야 하고, 노래를 반복 생성할 필요가 없다". 이것이 "한 번 성공하는 긴 링크"를 "독립적으로 재시도 가능한 여러 짧은 링크"로 개조했고, 산출물은 `suno_done` 중간 상태와 실패 후퇴이다. 이 원칙은 이후 플랫폼 전체의 장시간 작업 처리에서 재현됐다.

6. **#1556(05-25 13:33) 사용자가 5초 폴링과 이중 조건 종료 상태를 직접 제안** — 인간이 요구 단계에서 이미 구현 제약(간격, 판정 조건)을 제시했고, "下面的两点我不知道有没有必要，你参考一下有没有用"라는 표현으로 **결정권을 AI에게 돌려줬다**. 이런 "방안을 제시하되 강제하지 않는" 표현 방식은 논문에서 별도 항목으로 논의할 가치가 있다.

7. **#1594(05-27 17:41) "你不能……分析逻辑完全使用以前进行分析的所有代码吗"** — 인간이 **산출물 일관성**을 고집: 새 진입점이 분석 파이프라인을 별도로 세워서는 안 된다. 최종적으로 "존재하지 않는 Edge Function을 호출했고 fire-and-forget이 삼켜 버린" 조용한 장애를 드러냈다. 방법론적 의미: **fire-and-forget 비동기 호출에는 반드시 관측 가능한 실패 통로가 있어야 한다**. 그렇지 않으면 사용자의 "뭔가 이상하다"는 감각이 유일한 오류 보고 경로이다.

8. **#1610 5번(05-30 14:09) "正在上传账号，现在可以直接点击教学卡片生成"** — YouTube의 최종 일관성 창에 직면하여, 인간의 해법은 기술적 대기가 아니라 **상호작용 층에서 불확실성을 인정하고 사용자가 우회하게 하는 것**. 이것은 "비동기 상태 관리"가 백엔드 개념에서 UX까지 확장된 완전한 폐쇄 루프이다.

---

## 6. 변경 전 / 변경 후 효과 증빙: 조회 가능한 부분과 불가능한 부분

### 6.1 검증 가능한 "변경 전 상태"(모두 대화 아카이브 내의 실제 오류 텍스트)

| 증거 | 위치 | 내용 |
| --- | --- | --- |
| Suno 크레딧 소진으로 전체 흐름 500 | **#1524, 05-24 07:56** | `Edge function returned 500: Error, {"error":"Suno did not return taskId. msg=The current credits are insufficient. Please top up."}` |
| GitHub dispatch 403 | **#1542, 05-24 09:43** | `Edge function returned 500: Error, {"error":"GitHub dispatch failed: 403 {\"message\":\"Resource not accessible by personal access token\"…}"}` |
| 콜백 불안정 | **#1556 1번, 05-25 13:33** | 「YouTube视频生成完成后，有时候能收到youtube_url，但不稳定，偶尔收不到」 |
| 단어 단위 타임스탬프 누락 | **#1570, 05-25 19:23** | 「作은 행복这首歌……音乐模块也没有时间戳」 |
| 자동 분석 조용한 실패 | **#1610 6번, 05-30 14:09** | 「卡片倒是正常出现了，但是没有分析好的内容，需要我点进卡片之后再次点击지금분석하기才能开始分析」 |

위 다섯 건은 모두 **사용자가 그 자리에서 붙여 넣은 오류 원문 또는 직접 경험한 기술**이며, 사후 회술이 아니다. 대화 아카이브의 해당 메시지 번호 전문을 열람하여 대조할 수 있다.

### 6.2 검증 가능한 "변경 후 상태"

- **코드 층 검증 가능(HEAD `4dbd6bfd`)**: 이중 키 로테이션 `sg-suno-generate:11-47`; 크레딧 200 폴백 `:62-73`; 종료 상태 조임 `suno-client.ts:190` 및 `sg-suno-poll`; 6회 재시도 소프트 실패 `다이얼로그:273-289`; 5초 이중 조건 폴링 `다이얼로그:335-375`; 실패 저장 `video-trigger:89-103`; 공유 비밀 키 `video-callback:14-24`; 분석 파이프라인 통일 `sg-finalize-song`(`49e623dd`).
- **마이그레이션 층 검증 가능**: `render_jobs` 테이블 구조, RLS 4정책, 2트리거가 모두 `20260524085427_...sql`에 있다.
- **`49e623dd`의 커밋 diff 주석 자체**가 곧 근본 원인 설명이다(`the previously-called "analyze-generated-song" function did not exist, so analysis never ran automatically`). 이것은 본 사례에서 **코드 주석이 스스로 근본 원인을 증명하는 유일한 지점**이므로 논문에서 직접 인용을 권한다.

### 6.3 확실한 근거를 찾을 수 없는 부분(사실대로 표기)

| 요구 항목 | 결론 |
| --- | --- |
| 변경 전 / 변경 후 **스크린샷** | **이 항목은 확실한 근거를 찾을 수 없다.** 저장소에 어떤 스크린샷 파일도 없으며, 채팅 아카이브의 검색 가능 부분은 텍스트뿐이고 첨부와 도구 출력은 인덱스에 없다. |
| **자동화 테스트 / 회귀 테스트** | **이 항목은 확실한 근거를 찾을 수 없다.** 저장소에는 `src/test/example.test.ts`와 `src/test/setup.ts`만 있고 본 링크와 무관하며, 본 사례 9개 파일에는 한 번도 수반 테스트 커밋이 없었다. |
| **Edge Function 런타임 로그** | **일부 조회 가능, 이력 소급 불가.** `sg-suno-generate` / `sg-suno-poll`에 `console.log`(이중 키 전환 로그)가 있지만, 당시 로그는 보존 기간이 지나 아카이브 증거로 쓸 수 없다. #1539의 AI 답변이 "`video-trigger`의 로그를 확인했다"고 언급하지만 **로그 내용이 아카이브 본문에 남지 않았다**. |
| **GitHub Actions 측 실행 기록** | **이 항목은 확실한 근거를 찾을 수 없다.** 렌더링 저장소 `yuuisohe-ui/melody-video-renderer`는 본 저장소에 없으며, 그 workflow 정의, 실행 이력, ffmpeg 파라미터는 **본 프로젝트에서 검증할 수 없다**. 논문이 렌더링 세부를 기술하려면 해당 저장소에서 별도로 증거를 채취해야 한다. |
| **사용자 측 "고쳐졌다"는 명시적 회신** | **약함.** 사례 1·사례 2와 동일: 아카이브에 "이제 안정됐다/정렬됐다" 류의 확인 메시지가 없고, 매 라운드 수정 후 사용자는 바로 다음 문제로 넘어갔다. **"수정이 효과를 냈다"는 점에 사용자 측의 독립 확인이 부족**하므로, 논문 진술 시 "이후 이 문제는 다시 제기되지 않았다" 같은 반증 가능한 표현을 사용해야 한다. |
| **커밋 메시지의 "왜 바꿨는가"** | **없음.** 본 사례 창 내 71회 커밋의 메시지는 전부 `Changes`로, 어떤 의미 정보도 담지 않는다. 인과 체인은 "메시지 시각 → 커밋 시각"의 인접 관계로만 재건할 수 있으며, 본 아카이브의 모든 귀인은 이에 기반한 **추론이지 직접 증거가 아님**을 특별히 선언한다. |
| **`VIDEO_CALLBACK_SECRET`이 서명 메커니즘인지** | **반증됨.** `docs/thesis/05-ai-integration.md` 5.2.6절은 "서명"으로 기술하지만 `video-callback/index.ts:14-24`는 실제로 공유 비밀 키 상수 비교이다. 논문은 코드를 기준으로 해야 한다. |

---

## 7. 본 아카이브의 검증 체크리스트(답변 시 일일이 재현용)

1. `git log -1 --format='%h %ad'` → `4dbd6bfd`(2026-08-05 11:31)이어야 함.
2. `git log --format='%h %ad' --date=iso -- supabase/functions/video-trigger/index.ts` → 6회 커밋이 나와야 하며, `25349c98`(05-24 08:56)부터 `02d9cd6e`(05-28 03:51)까지.
3. `git log --since=2026-05-19 --until=2026-05-31 --numstat --format='' -- src/components/songs/YoutubeVideoGenerateDialog.tsx | awk '{a+=$1;d+=$2}END{print a,d}'` → `914 167`이 나와야 함(45회 커밋).
4. `git show 271df1e4` / `git show f171e45e` → 각각 1행 변경, `TEXT_SUCCESS` 삭제여야 함.
5. `git show 49e623dd` → `analyze-generated-song` → `analyze-song`과 주석 `…did not exist, so analysis never ran automatically`가 보여야 함.
6. `rg -n 'type Step' src/components/songs/YoutubeVideoGenerateDialog.tsx` → `L52`에 히트, 7개 상태여야 함.
7. `rg -n 'setInterval\(tick, 5000\)' src/components/songs/YoutubeVideoGenerateDialog.tsx` → `L373`과 `L395`에 히트(두 개의 폴러).
8. `cat supabase/migrations/20260524085427_64cac0f1-7654-42cc-8de0-ac77a466e693.sql` → `render_jobs` 테이블 생성, 4개 RLS 정책, 2개 트리거가 보여야 함.
9. 대화 아카이브 대조: `chat_search--read_chat_messages`로 #1478 / #1532 / #1546 / #1550 / #1556 / #1572 / #1594 / #1610 여덟 건을 읽어 제2절 인용문과 글자 단위로 대조(#1532의 토큰은 본 아카이브에서 마스킹 처리됨).
10. 중단 기간 반증: `git log --since=2026-05-20 --until=2026-05-23 -- supabase/functions/sg-suno-* supabase/functions/video-* src/components/songs/YoutubeVideoGenerateDialog.tsx` → 비어 있어야 함.
