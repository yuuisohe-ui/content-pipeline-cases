# 사례 4 원자료 아카이브 · 교육 콘텐츠 확보의 복원력 (논문 5.1.4)

> 본 문서는 **증거 아카이브**이며 논문 본문이 아니다. 목적은 사례 1, 2, 3과 동일하다: 제5장 사례 4의 모든 진술이 제3자에 의해 검증될 수 있도록 하는 것이다.
> 본 문서는 `docs/thesis/05-collaboration-cases.md`, `05-case1-*.md`, `05-case2-*.md`, `05-case3-*.md`를 **수정하지 않으며**, 어떤 소스 코드, 마이그레이션, 프롬프트 부록도 변경하지 않는다.
>
> **검증 기준 버전**: 저장소 HEAD = `c897cc5d`(2026-08-06 03:54 UTC). 본 문서의 모든 행 번호는 이 버전에서 `code--view` / `rg -n`으로 현장 검증한 결과이다.
> **대화 아카이브 기준**: 본 프로젝트의 채팅 아카이브는 현재 총 **2260건**의 사용자 가시 메시지를 포함한다(사용자 메시지 + AI 응답 본문; 도구 호출과 도구 출력은 아카이브에 포함되지 않아 검색 불가).
>
> **증거 등급 표기**(사례 1~3의 4단계 체계 계승)
> - `<V-msg#N>`축자 인용: 채팅 아카이브 제N번 메시지에서 원문 그대로 발췌.
> - `<G-hash>` 커밋 증거: `git log` / `git log --numstat`으로 재현 가능.
> - `<C-file:line>` 코드 위치: HEAD `c897cc5d`에서 `rg -n`으로 현장 검증.
> - `<X>` 무증거: "찾을 수 없음"을 명시하고 추측성 서술을 하지 않음.

---

## 1. 시간 창구 경계

본 사례는 네 사례 중 **유일한 2단계 사례**이다: 폴백 체인이 한 번에 설계된 것이 아니라, 41일 간격을 두고 두 차례에 걸쳐 서로 다른 실패 계층을 각각 해결했다. 따라서 창구를 구간별로 선언해야 한다.

| 항목 | 값 | 설명 |
| --- | --- | --- |
| **단계 A 기점** | **메시지 #207, 2026-03-22 09:46 (UTC)** | 사용자가 처음으로 "기존 가사 로직 위에 **타임스탬프가 있는 자막** 우선 계층을 추가하고, 실패 시 조용히 폴백해야 한다"고 제안. 이 메시지 자체가 폴백 아키텍처의 형태를 규정했다. |
| 단계 A 종점 | **메시지 #225, 2026-03-22 11:16 (UTC)**; 대응 커밋 `634dcc5d`(SRT 파싱, 03-22 11:18) 및 `e2d6492e`(03-22 11:29), `3dc7180f`(03-22 12:40), `29113580`(03-22 14:02) | 제3단계 "수동 SRT 붙여넣기" 확정. 이후 당일 커밋은 HSK 등급 분류와 Korean prompt로 전환되어 자막 수집과 무관. |
| 단계 A 기간 | **1시간 30분**(대화) / 당일 커밋은 14:02까지 연장 | 단일 고강도 세션 |
| **단계 B 기점** | **메시지 #1238, 2026-05-02 13:50 (UTC)** | 사용자가 단계 A 방안의 실패를 명확히 선언: "**当然你现在的办法同样也不能准确下载**"라고 하며 4가지 후보 아키텍처를 능동적으로 제시. |
| 단계 B 종점 | **메시지 #1277, 2026-05-04 13:56 (UTC)**(비차단 추가 UX 확정); 대응 커밋 `62c9bae1` / `97848834`(05-04 13:56) | 이후 #1278은 "공유" 버튼으로, #1282는 가사 카드보내기로 전환되어 주제가 자막 수집에서 벗어남. |
| 단계 B 기간 | **2일 0시간 6분**(2026-05-02 13:50 → 2026-05-04 13:56 UTC) | 실제 활동일 3일: 05-02, 05-03, 05-04 |
| 양 단계 간격 | **41일**(2026-03-22 → 2026-05-02) | 간격 기간 중 `analyze-song`에 다수의 커밋이 있으나 주제는 재개 가능한 처리(사례 2, 04-04~04-13)와 어휘표/문형이며 자막 수집 계층이 아님 |
| 창구 내 사용자 항목 | 단계 A **10건**(#207, 209, 211, 213, 215, 217, 219, 221, 223, 225); 단계 B **21건**(#1238~#1278 중 짝수 번호) | 제2절에 일괄 열거 |

### 1.1 메시지 선정 규칙("완전성"의 범위를 설명하여 과도한 주장을 회피)

- 단계 A는 **#205–#225 연속 구간 내 모든 사용자 메시지**를 대상으로 함; 이 구간의 사용자 메시지는 홀수 번호(#205, 207, 209…225)이며 전부 직접 읽었다. 그중 **#205**(공용/내 곡 귀속 라벨)는 본 사례 주제가 아니므로 **제외 항목**으로 제2절 말미에 명시한다.
- 단계 B는 **#1238–#1278 연속 구간 내 모든 사용자 메시지**를 대상으로 함; 이 구간의 사용자 메시지는 짝수 번호(#1238, 1240…1278)이며 전부 직접 읽었다. 그중 **#1252 제1, 2항**(스크롤바와 검색창 높이), **#1278**(공유 버튼)은 본 사례 주제가 아니므로 **제외 항목**으로 명시한다.
- 양 단계 사이의 **#226–#1237(약 1012건)은 일일이 정독하지 않았고** 키워드 검색만 수행했다. 검색 결과는 제6.3절에 있는 그대로 보고하며, **본 아카이브는 이 구간에 관련 메시지가 전혀 없다고 주장하지 않는다**. 이는 본 아카이브가 사례 1~3과 "완전성" 주장에서 다른 유일한 지점이므로 특별히 명기한다.

---

## 2. 창구 내 사용자 메시지 ·축자 원문

### 단계 A · 2026-03-22: "AI가 가사를 추측"에서 "3단계 폴백" 프로토타입으로

---

**#207 · 2026-03-22 09:46 · 사용자** `<V-msg#207>`(영문 원문, 핵심 단락 발췌; 전문은 4개 우선순위와 7개 Important requirements 포함)

> Please do NOT rewrite my current lyrics logic for now.
> […] I suspect the current result may come from AI inference or lyrics matching based on the song title and artist, rather than from the actual timestamped transcript of the selected YouTube video.
> **Desired priority order:**
> 1. First, try to fetch transcript/caption data with timestamps from the currently selected YouTube video itself.
> 2. If timestamped transcript data is successfully retrieved, use it as the primary source for synced lyric scrolling.
> 3. If transcript/timestamp data is unavailable, fall back to my current existing lyrics/analysis logic without changing its behavior.
> 4. In fallback mode, keep showing the current lyrics/analysis result, but do not enable auto-scroll.
> **Important requirements:**
> - The timestamp feature must be optional and **safe to fail**.
> - If timestamp fetching fails, the app should **silently fall back** to the current behavior.
> […] Before coding, first tell me what you think my current lyrics pipeline is doing, then add the timestamp feature with minimal changes.

**이 메시지가 전체 사례의 아키텍처 모본이다.** 세 가지 전문적 판단이 한 번에 모두 제시되었고, 순서도 이후 구현과 일치한다:
1. **비파괴적 증분**("Do NOT rewrite" / "non-destructive enhancement layer") — 폴백 체인이 "교체"가 아니라 "중첩"이 되도록 결정;
2. **safe to fail + silently fall back** — 각 단계가 예외를 던지지 않고 반드시 `try/catch`로 삼키도록 결정;
3. **다운그레이드는 실패가 아님**(제4항: 폴백 시에도 가사를 표시하되 자동 스크롤만 끔) — "가사 있음, 타임라인 없음"이 오류가 아니라 **합법적 종료 상태**가 되도록 결정.

인접 AI 응답(#208): 기존 pipeline을 먼저 읽고 요약한 뒤, transcript 검색을 위한 신규 Edge Function 추가를 제안.
인접 커밋: `1ceccbce`「Enhance lyric timeline support」(03-22 10:03) `<G-1ceccbce>`

---

**#209 · 2026-03-22 09:48 · 사용자** `<V-msg#209>`

> Lovable tool use: Approved.
> **개선 방향**: 매핑 정확도 향상 ||| 자막 미리보기 추가

(승인 + 두 개의 후속 방향 라벨)

---

**#211 · 2026-03-22 09:58 · 사용자** `<V-msg#211>`

> 甜蜜蜜 这个音乐视频 This video definitely has timestamped subtitles.
> Please debug why auto-scroll still does not work.
> I want you to check these exact points:
> 1. Are we successfully fetching transcript entries with real timestamps (startMs / durationMs), not just subtitle text?
> 2. How many displayed lyric lines are actually matched to timestamped transcript entries?
> 3. Are the displayed lyrics different from the original subtitle text in a way that breaks matching?
> 4. Is the frontend actually using the YouTube player current time to update active line and auto-scroll?
> Please show me: the fetched timestamped transcript sample / the mapped lyric lines with timestamps / the unmatched lines / the reason auto-scroll is not triggered.

**방법론적 의의**: 사용자는 "고쳤다"는 말을 받아들이지 않고 **반드시 출력해야 할 네 가지 중간량을 지정**했다. 이것이 이후 `useClientTranscript.ts`의 다량 `console.log`의 직접적 출처이다 `<C-src/hooks/useClientTranscript.ts:41-50>`(`sample: entries.slice(0,5).map(e => ({ text, startMs, durationMs }))` — `startMs`/`durationMs`라는 필드명이 바로 #207의 요청 형식과 #211의추궁 용어이다).

인접 커밋: `b8130b65`「Debug transcript pipeline traces」(03-22 10:12) `<G-b8130b65>`

---

**#213 · 2026-03-22 10:08 · 사용자** `<V-msg#213>`(다섯 항의층별 추궁, 앞뒤 발췌)

> However, auto-scroll still does not work. Please debug the full pipeline step by step and show me the actual data at each stage. […]
> 5. If auto-scroll is still not happening, **identify the exact failing layer**: transcript fetch / database save / mapping / active line calculation / scroll UI trigger
> **Please do not give me a general explanation. Please print the real runtime values and tell me exactly where the chain breaks.**

**본 사례에서 가장 방법론적 가치가 높은 메시지이다**: 사용자가 하나의 체인을 귀인 가능한 5개 계층으로 분해하고, AI에게 설명이 아니라 단절 지점의 지목을 요구했다. 이는 디버깅을 "추측"에서 "이진 탐색 위치 특정"으로 바꾼 것이다.

인접 AI 응답(#212, 아카이브에서 검색 가능) `<V-msg#212>`:

> …Function 里的服务端字幕抓取（page scraping + timedtext API + innertube API）全部失败。视频确实有 `zh-Hant` 字幕（通过 YouTube Data API `captions.list` 确认），但内容下载被阻断。**修复方案**：新增了**客户端字幕获取层**（`useClientTranscript`）：1. 当服务端 `timed_entries` 为空时，自动在用户浏览器中通过 innertube API 获取字幕 2. 浏览器请求不受…

---

**#215 · 2026-03-22 10:18 · 사용자** `<V-msg#215>`

> Thanks, this confirms the exact break point.
> So the problem is no longer mapping or scrolling logic. The transcript never enters the app because:
> 1. server-side fetch is blocked by YouTube bot detection on cloud IPs
> 2. client-side fetch is blocked by YouTube CORS
> **Please stop trying the direct browser fetch approach.**
> Next, I want you to propose the best practical fallback architecture for my app.
> Requirements: 1. Keep my current lyrics/analysis flow unchanged 2. Preserve the current fallback when no timestamps are available 3. Propose **2-3 realistic ways** to add timestamped sync despite YouTube blocking: proxy / relay service approach / third-party subtitle source approach / any other practical approach 4. For each option, explain: implementation complexity / reliability / maintenance risk / whether it works well inside Lovable + Supabase. Then recommend the best option for my current product stage.

**핵심 전환점 T1.** 사용자는 여기서 세 가지를 완수했다: (a) 귀인 결론을 **사람이 직접 제시** — 서버 측은 IP 봇 탐지에, 클라이언트 측은 CORS에 막혔으며, 이 둘은 서로 다른 벽이다; (b) AI가 추진 중이던 경로를 **능동적으로 중단**시킴; (c) "하나의 방안"이 아니라 **2~3개 방안 + 4차원 평가 행렬**(복잡도 / 신뢰성 / 유지보수 리스크 / 스택 적합성)을 요구한 뒤 스스로 결정. 이것이 "인간의 전문적 판단"이 가장 집중적으로 드러난 지점이다.

---

**#217 · 2026-03-22 10:38 · 사용자** `<V-msg#217>`

> I already have a Supadata API key from the free plan. Please proceed with the integration now. Tell me the exact Supabase secret name I need to add for the key.
> Important: - Do not change my current lyrics/analysis flow - Only add the timestamp transcript layer - If timestamped transcript is available, enable auto-scroll - If not, keep the current behavior unchanged

(사용자가 제3자 계정을 직접 준비; 네 가지 제약이 #207과 일치 — 20분 전의 제약이 그대로재술됨은 "제약 안정성"의 증거)

인접 커밋군(03-22 10:40–10:42, 5분 내 5회) `<G>`:
`76a34f79`「Add supadata integration」→ `0fb6d97d`「Add Supadata transcript fetch」→ `a6c20cb5`「Add transcript edge function」→ `955cbe0a`/`569e599e`/`189fb8bc`「Add Supadata fetch edge function」

---

**#219 · 2026-03-22 10:39 · 사용자** `<V-msg#219>` — `Lovable tool use: Approved. tool_submit:Adding secret(s)`(`SUPADATA_API_KEY` 등록)

---

**#221 · 2026-03-22 10:55 · 사용자** `<V-msg#221>`

> 给我输出月亮代表我的心的时间戳数据，这边显示 19 句都对应上了，但是歌词并不能随着视频播放滚动，只显示最后一句是亮的。不管我点击哪一句都是从头播放，我想要点击当句时，视频也跳到该句唱的地方

인접 커밋: `8a8850bd`/`cd7f10be`(03-22 10:55)

---

**#223 · 2026-03-22 10:57 · 사용자** `<V-msg#223>`

> 在我点击重新分析的按钮之前，其他的歌先不用获取字幕数据，先做月亮代表我的心就好

**엔지니어링 판단: 자동 가져오기를 필요 시 트리거로 변경하여, 단일 곡으로 디버깅 범위를 수렴시키고 동시에 제3자 무료 할당량을 보호한다.** 이 구두 지시는 코드에 장기적 흔적을 남겼다 — `useClientTranscript`는 지금까지 `autoFetch` 스위치를 유지하며 기본값이 `false`이다 `<C-src/hooks/useClientTranscript.ts:65>`(`autoFetch: boolean = false`), 그리고 `<C-src/hooks/useClientTranscript.ts:83-86>`에서 곧바로 `return`하며 `"auto-fetch disabled, skipping"`을 출력한다.

인접 커밋군: `b8c903d3`「Limit analysis to Moon」, `8e6489eb`「调整月亮代表我的心转录获取优先级并禁用自动获取」, `23bc7b1c`「Disable auto-fetch in transcript hook」, `4c3a4194`「Disable auto-fetch for transcripts and prepare on-demand fetch for 月亮代表我的心」(모두 03-22 11:00–11:02) `<G>`

---

**#225 · 2026-03-22 11:16 · 사용자** `<V-msg#225>`(《甜蜜蜜》SRT 전체 25단락 첨부, 여기서는 앞뒤만 발췌)

> 把 가사 직접 입력하여 분석 和 가사 고치기 里可以分析带时间戳的歌词内容，一般格式为
> "1
> 00:00:08,480 --> 00:00:15,960
> 甜蜜蜜 你笑得甜蜜蜜
> 2
> 00:00:16,010 --> 00:00:21,120
> 好像花儿开在春风里
> […]
> 25
> 00:03:09,040 --> 00:03:16,730
> 啊 在梦里"。
> 一般由行号，时间戳，和歌词组成，可以根据这个内容进行分析，实现点击歌词就跳转到相应时间然后开始播放

**핵심 전환점 T2 — 제3단계 폴백의 탄생.** 사용자는 "더 강력한 스크래핑 서비스로 교체"를 더 이상 요구하지 않고, 자동화의 신뢰성 부족을 인정하며 **사람**을 폴백 체인 말단에 배치했다: 교사가 직접 SRT를 붙여넣고 시스템이 파싱을 담당한다. 또한 사용자는 **형식 명세**(행 번호 / 타임스탬프 / 텍스트 3단 구성)와 수용 기준(클릭 시 해당 시간으로 이동)을 직접 가져왔다.

인접 커밋군: `868f400e`/`5f8334ca`/`98324728`「Add SRT-style lyric parsing」/`634dcc5d`「Add SRT parsing for lyrics」(03-22 11:18) → `f3f4b474`「同步歌词跳转修正」(11:27) → `e2d6492e`「Fix SRT parsing for timed lyrics」(11:29) → `3dc7180f`「Improve SRT handling and mapping」(12:40) → `29113580`「Fix custom_timed SRT handling」(14:02) `<G>`

---

### 단계 B · 2026-05-02 ~ 05-04: 실패 인정, yt-dlp 외부화, 가사 데이터베이스 도입

---

**#1238 · 2026-05-02 13:50 · 사용자** `<V-msg#1238>`

> 我现在在노래 아카이브搜索 youtube 链接时，这个字幕下载时怎么实现的？我记得之前我试了很多种办法，都不能准确下载字幕，**当然你现在的办法同样也不能准确下载**。你把这些失败的内容一起告诉我，都是怎么实现的以及为什么无法下载。另外下面这几种办法你看能不能实现：我想在平台实现输入 YouTube 链接后，自动判断该视频是否有字幕，有字幕的自动获取并分析，没有字幕的走现有 AI 分析流程。
> 我目前用 Python + yt-dlp 在本地实现了这个功能，但在 Lovable 里试了几种方式都没成功。
> 请问你们支持以下哪种方案？
> 1. 我自己在 Railway/Render 部署一个 FastAPI 后端跑 yt-dlp，Lovable 前端调这个接口
> 2. 调用 youtube-transcript-api 的 REST 封装接口，前端直接 fetch
> 3. 用 Supabase Edge Functions 调字幕接口
> 4. 直接请求 YouTube timedtext API（前端方案）
> 你们现在用的是什么技术栈？前端框架是什么？后端支持写自定义 API 吗？

**핵심 전환점 T3.** 세 계층의 전문적 판단: (a) **41일간의 실제 사용**으로 단계 A 방안이 기준 미달임을 판정하고, AI에게 **모든 실패 방식의 복기**를 요구(AI에게 자신의 과거 방안에 대한 부정적 귀인을 요구하는 드문 사례); (b) 사용자는 이미 **로컬에서 Python+yt-dlp로 실현 가능성을 검증**한 상태로, "검증된 해답"을 가지고 요구를 제기했으며 AI의 과업은 "방법 고안"에서 "배선"으로 격하됨; (c) 4개 후보를 한꺼번에 제시하고 런타임 능력 경계를 추궁.

---

**#1240 · 2026-05-02 14:35 · 사용자** `<V-msg#1240>` — `我现在要实现这个搜索YouTube`(절단된 메시지, 실질 내용 없음)

---

**#1242 · 2026-05-02 14:36 · 사용자** `<V-msg#1242>`

> 我现在要实现这个搜索 YouTube 链接或者歌曲时自动调动字幕的内容，请参考这个提示词：我部署了一个 yt-dlp 字幕提取服务，地址是：
> `https://web-production-5712.up.railway.app`
> 接口是 POST /transcript，请求格式是：
> ```json
> { "url": "https://www.youtube.com/watch?v=xxxxx",
>   "languages": ["zh","zh-Hans","zh-CN","zh-Hant","zh-TW","ko","en"] }
> ```
> 返回格式是：
> ```json
> { "ok": true, "has_subtitles": true, "source": "manual_caption",
>   "language": "zh", "entries": [{"start": 4.0, "text": "字幕内容"}],
>   "plain_text": "完整字幕文本" }
> ```
> 请修改 get-youtube-transcript 后端函数，优先调用这个接口获取字幕。如果这个接口返回 `has_subtitles: false`，再走现有的 fallback 流程。提示词不一定完全正确，你按照你的实现方式试试，可以实现吗，另外我想让搜索 youtube 视频时出现是否有字幕的功能，怎么实现

**사용자가 전달한 것은 완전한 인터페이스 계약이다**: URL, 메서드, 요청 본문 필드, 응답 본문 필드, 그리고 **다운그레이드 트리거 조건**(`has_subtitles: false` → fallback 진행). 이 계약은 HEAD에서 필드 단위로 그대로 구현되어 있다:
- 요청 본문 `{ url, languages }` → `<C-supabase/functions/get-youtube-transcript/index.ts:56>`
- 응답 필드 `ok / has_subtitles / entries` 3중 판정 → `<C-supabase/functions/get-youtube-transcript/index.ts:200>`(`if (yt && yt.ok && yt.has_subtitles && Array.isArray(yt.entries) && yt.entries.length)`)
- 언어 우선순위 배열 기본값(중국어 7단계, 한국어 5단계) → `<C-supabase/functions/get-youtube-transcript/index.ts:9-14>`
- `source` 필드의 내부 라벨 매핑 → `<C-supabase/functions/get-youtube-transcript/index.ts:201-203>`
- 참고: 사용자가 준 `entries` 예시에는 `{start, text}`만 있고 `dur`이 없어, 구현 측에서 `dur`의 호환 값 처리를 처리함 `<C-supabase/functions/analyze-song/index.ts:707-711>`(`dur: Number(e.dur || e.duration) || 0`).

---

**#1244 · 2026-05-02 14:48 · 사용자** `<V-msg#1244>`(자격 증명 포함, 여기서는 아카이브 원문 구조를 유지하되 키를 부분 마스킹)

> 好的，请开始实施。调整可以参考：
> 1. Railway 调用**超时改为 30 秒**
> 2. API Key 已配置好，Railway 那边变量名是 `API_KEY`，值是 `ytdlp-subtitle-****`
> 3. 请在 Lovable 的 Supabase secrets 里也加上 `YT_DLP_SERVICE_URL` 和 `YT_DLP_SERVICE_KEY`，然后开始写 get-youtube-transcript 函数
> 调用时请求头加：`Authorization: Bearer ytdlp-subtitle-****`

> ⚠️ **아카이브 윤리 주석**: #1244 원문에는 평문 API Key가 포함되어 있다. 본 아카이브는 인용 시 마스킹했다. 논문 본문에서 이 항목을 인용할 경우에도 동일하게 마스킹해야 하며, "vibe coding 과정의 자격 증명 유출 리스크" 소재로 활용할 수 있다(사례 3 #1532의 GitHub 토큰 유출과 같은 유형의 표본으로, 총 2건).

**"30초"는 순수한 인간 파라미터이다.** HEAD에서도 여전히 30000ms이다: `<C-supabase/functions/get-youtube-transcript/index.ts:32>`(`timeoutMs = 30000`), `<C-supabase/functions/get-youtube-transcript/index.ts:199>`(`callYtDlpService(youtubeUrl, langPriority, 30000)`), `<C-supabase/functions/analyze-song/index.ts:690>`(`setTimeout(() => controller.abort(), 30000)`).
> **중요한 반례(확인됨, 생략 불가)**: #1261에서 AI는 이 타임아웃을 **30s에서 12s로 낮추어** 폴백을 가속하자고 제안했다. 이 제안은 **채택되지도 구현되지도 않았다** — HEAD의 세 곳 모두 여전히 30000이다. 이는 폴백 체인의 시간 예산이 최종적으로 사람에 의해 고정되었고, AI의 성능 최적화 제안이 부결되었음을 보여준다. `<G>`＋`<C>` 이중 증거.

---

**#1246 / #1248 / #1250 · 2026-05-02 14:53 / 14:56 / 14:57 · 사용자** `<V-msg#1246,1248,1250>`
`Lovable tool use: Approved. tool_submit:Adding secret(s)` / `Updating secret(s)` ×2

인접 커밋군: `5ecee350`/`f16f31e9`(05-02 14:54) → `79404ec3`/`a6a65976`/`53ef6569`/`4ca201ae`(05-02 15:00–15:01) `<G>`
`supabase/functions/get-youtube-transcript/index.ts`는 **전체 수명 주기에서 단 1회 커밋, 바로 이날**이다(262행이 한 번에 완성, 이후 3개월간 무수정) `<G>` — 이는 "계약 선행"이 가져온 일회성 정확성의 정량 표지이다.

---

**#1252 · 2026-05-03 07:57 · 사용자** `<V-msg#1252>`(제3항은 본 사례에 해당, 제1·2항은 UI이므로 제외 항목 참조)

> 3. 这个视频是否有字幕现在要点击才知道，怎么才能让他搜索的时候就直接显现呢？

---

**#1254 · 2026-05-03 08:03 · 사용자** — `Lovable tool use: Approved.`

인접 커밋군: `9c37d132`/`da9621d4`/`7214cbd0`/`1b41cebe`/`7f0b8254`(05-03 08:04) + `5de15a21`/`f539475a`/`dbfc7531`(08:05) `<G>`

---

**#1256 · 2026-05-03 08:09 · 사용자** `<V-msg#1256>`

> 1. 现在搜索歌词是用什么逻辑搜索的，很多我在 youtube 上看到的，在这里都搜索不到。
> 2. 是否有字幕是依据什么判定的，youtube api 没办法看到这个视频有没有字幕吗，现在显示是否有字幕的速度太慢了，有没有什么其他的办法

**핵심 전환점 T4 — "탐지"와 "취득"의 디커플링.** 사용자 지적: "자막이 있는지" 판단이 "다운로드 가능한지"를 기다려서는 안 된다. 제2문은 YouTube Data API `contentDetails.caption` + `captions.list`를 이용한 지연 없는 탐지를 직접 산출하여, 3–10초 사전 검사를 완전히 삭제했다(#1259).

인접 AI 응답(#1259)의 구현 내용 `<V-msg#1259>`: `videoCategoryId: "10"` 및 하드코딩된 "中文 歌词" 접미사 제거(재현율); 검색 결과 10 → 15; `caption` / `captionLanguages` 필드 신설; `autoCheckAllSubtitles`와 `checkSubtitles` 삭제.
인접 커밋군(05-03 08:12–08:13, 8회): `bb2f3786`/`8adf4f81`/`fde1fb5b`/`9cd7bf6a`/`4d878a3c`/`09e20cc3`/`34f76c7b`/`15ec3ee6`/`2be659d9` `<G>`

---

**#1258 · 2026-05-03 08:11 · 사용자** — `好的`

---

**#1260 · 2026-05-03 08:23 · 사용자** `<V-msg#1260>`

> 1. 我刚才搜索了一首"我不难过"，我确定他是有字幕的，而且搜索的时候也是显示有字幕的，为什么最后分析的时候还是 ai 推测的歌词呢？是哪里出了问题？
> 2. 在添加有字幕的歌曲时，点击添加后，那个添加中的时间会特别长，这个时候如果我把搜索的框关掉，没办法知道这个歌曲有没有添加，能不能设定为只要点击添加，就会生成卡片，只是如果没有分析完成的话显示正在分析中，然后里面的内容是空的

**사용자는 여기서 본 사례에서 가장 날카로운 실패 모드를 정의했다: 「탐지는 있다고 함 ≠ 가져올 수 있음」.** 이는 폴백 체인의 문제를 "단계 하나 추가"에서 "메타데이터와 콘텐츠가 서로 다른 채널에서 오므로 불일치가 필연적으로 발생하며, UI는 이 불일치를 정직하게 표현해야 한다"로 격상시켰다. 제2항은 폴백 체인의 **시간 비용**을 UX 문제로 외부화했다 — 폴백이 4단계를 모두 거쳐야 한다면 사용자를 마냥 기다리게 할 수 없으므로 반드시 비차단이어야 한다.

---

**#1262 · 2026-05-03 12:32 · 사용자** `<V-msg#1262>`

> Supadata 现在能成功获取字幕吗？成功率怎么样？我记得成功率非常不好，试过很多次都没办法成功获取到字幕

**핵심 전환점 T5 — 사용자가 기존 폴백 단계에 신뢰성 질의를 제기.** AI의 응답(#1263)은 드물게 자기부정을 하고 **데이터 부재를 인정**했다:

> `<V-msg#1263>` 刚查了 `fetch-transcript` 和 `get-youtube-transcript` 的 edge function 日志，**都没有日志记录**，所以我无法给您一个准确的成功率数据。
> …很多中文 MV 实际上是"嵌入式硬字幕"（直接烧录在视频画面里），YouTube API 检测不到，Supadata 也拿不到。

AI는 세 가지 선택지 (A) 로그 통계 추가 후 1주일 운영 (B) Supadata 우선순위 하향 (C) 다른 서비스 탐색을 제시했다. **사용자는 셋 중 어느 것도 선택하지 않았다** — #1264 참조.

---

**#1264 · 2026-05-03 14:29 · 사용자** `<V-msg#1264>`

> 我找到了一个更好的方案，你看怎么样，修改歌词获取流程，按以下优先级执行：
> **第一优先：LRCLIB**（免费歌词数据库，无需 API Key）
> 搜索接口：`https://lrclib.net/api/search?q=歌名+歌手名`
> 取返回结果中第一个 `instrumental: false` 且 `plainLyrics` 不为空的条目
> **只需要 plainLyrics 字段，不需要时间轴**
> 第一次用原始语言搜索，没找到再用 AI 翻译成英文搜索一次
> **第二优先：Supadata**（原有方案保留作为备用）和现在一样，LRCLIB 找不到时才调用
> **第三优先：AI 分析（兜底）** Supadata 也失败时走现有 AI 分析流程

**핵심 전환점 T6 — 본 사례에서 "인간의 전문적 판단" 밀도가 가장 높은 메시지이자 전체 사례의 핵심이다.**
사용자는 AI가 제시한 세 가지 선택지에서 벗어나 **문제 재정의**를 수행했다: "영상의 자막"을 얻을 수 없다면, 다른 소스에서 "이 노래의 가사"를 가져오면 된다. 여기에는 AI가 제안하지 않은 네 가지 판단이 포함된다:
1. **소스 유형 전환**: "영상 파생 데이터"(caption track)에서 "곡 메타데이터베이스"(LRCLIB)로 전환. 처음 세 폴백 단계는 모두 동일한 소스(YouTube)에 묶여 있어 YouTube가 차단하면 전선이 붕괴한다 — 이는 **상관 실패**이며, 사용자가 이를 식별해냈다.
2. **타임라인의 능동적 포기**: "只需要 plainLyrics 字段，不需要时间轴". 정확성을 동기화와 맞바꾸는 **명시적 트레이드오프**이며, #207 제4항의 "다운그레이드하되 사용 가능"으로 회귀한다.
3. **무의존 선호**: "免费歌词数据库，无需 API Key" — 운영 및 할당량 리스크 저감.
4. **2회 검색 전략**: 원어 → AI 영어 번역 후 재검색. CJK 곡명이 국제 데이터베이스에 수록되는 형태를 인간이 선험적으로 예측했다.

**그리고 이 판단은 옳았다**: 사흘 만에 LRCLIB은 "제3자 중 하나"에서 전체 체인의 가장 앞 단계가 되었고, HEAD에서도 여전히 선두에 있다.

---

**#1266 · 2026-05-03 14:31 · 사용자** — `Lovable tool use: Approved.`
인접 커밋: `ec218ee4`/`b9ec97a4`(05-03 14:32) `<G>`

---

**#1268 · 2026-05-04(아카이브 타임스탬프 05-03 14:44)· 사용자** `<V-msg#1268>`

> 我刚才添加了"我不难过"这首歌吗我在这个 lirlib 里搜到这首歌了，歌名一样但是歌手名这个网站显示的是"Stefanie Sun"，怎么能避免

**사용자가 직접 LRCLIB 공식 사이트에서 대조 실험을 수행하여, 불일치의 근본 원인이 "아티스트명 정규화 차이"(孙燕姿 ↔ Stefanie Sun)임을 특정했다.** 이는 버그 리포트가 아니라 근본 원인을 가지고 온 것이다.

---

**#1270 · 2026-05-04 12:48 · 사용자** — `确认`
인접 커밋: `b27f7f87`/`08021c99`(05-04 12:49) `<G>`; 5-pass 검색 + `pickBestLrclibMatch` + `extractArtistFromVideoTitle` 구현

---

**#1272 · 2026-05-04 13:43 · 사용자** `<V-msg#1272>`

> 我重新试了一下，还是没有拿到"我不难过"的正确歌词，按你的逻辑，在搜索"我不难过"的时候就应该能返回结果，或者**先搜索歌曲名，然后在出现的内容中选取一个和 youtube 中歌手名相同或最相近的那个**，这样能实现吗

**핵심 전환점 T7 — 사용자가 최종 구현 알고리즘을 구술했다.** 사용자가 제안한 "title-only로 먼저 검색하고, 후보 집합에서 아티스트명 유사도로 최적을 선택"이 바로 HEAD의 `fetchFromLRCLIB` **Pass 0**이다:
- `<C-supabase/functions/analyze-song/index.ts:559-563>` 주석에 사용자가 준 예시가 코드에 그대로 기록됨: `Many songs (e.g. "我不难过") are stored on LRCLIB under the romanized artist (Stefanie Sun) while we have the CJK artist (孫燕姿)`
- `<C-supabase/functions/analyze-song/index.ts:564>` 먼저 title-only 검색: `const r0 = await lrclibSearchRaw(title);`
- `<C-supabase/functions/analyze-song/index.ts:566-574>` `instrumental` 필터 + `plainLyrics.length < 100` + 제목 정규화 공차 ≤3자
- `<C-supabase/functions/analyze-song/index.ts:575-578>` **유일 정확 일치 시 즉시 채택**(사용자의 "就应该能返回结果"에 대응)
- `<C-supabase/functions/analyze-song/index.ts:579-589>` 복수 일치 시 hints(channel / 영상 제목에서 추출한 아티스트) 유사도로 최적 선택("最相近的那个"에 대응)
- `<C-supabase/functions/analyze-song/index.ts:590-601>` 그래도 불일치 시 AI로 아티스트명을 번역하여 재비교(#1264의 두 번째 검색에 대응)

---

**#1274 · 2026-05-04 13:50 · 사용자** — `Lovable tool use: Approved.`
인접 커밋: `b0428958`/`3bb4b034`/`69833ad9`(05-04 13:50–13:51) `<G>`
이번에 **아키텍처 위치 이전**도 함께 완료됨: LRCLIB 호출이 `fetchYouTubeSubtitles` 내부에서 메인 플로우의 `extractCleanTitle()` **이후**로 이동 — 코드에 그 이유가 주석으로 남아 있음 `<C-supabase/functions/analyze-song/index.ts:677-680>` 및 `<C-supabase/functions/analyze-song/index.ts:2619-2624>`, 후자는 실패 사례를 주석에 그대로 기록함: `raw title: "我不难过 孙燕姿 (歌词版)" / raw channel: "GM Lyric"`.

---

**#1276 · 2026-05-04 13:54 · 사용자** — `继续`(#1260 제2항 계승: 비차단 추가)
인접 커밋: `31fbb791`/`d537f3c0`/`97848834`/`62c9bae1`(05-04 13:55–13:56) `<G>`
구현: `add_only: true`로 먼저 행 생성 + 즉시 팝업 닫기 + `pendingAnalyses` 자리 표시 카드 + 백그라운드 fire-and-forget 전체 분석. `add_only` 분기는 HEAD의 `<C-supabase/functions/analyze-song/index.ts:2673-2700>`에 있으며, 그중 `has_subtitles: !!lyrics.trim() && !usedAiSearch` `<C-supabase/functions/analyze-song/index.ts:2684>`이 바로 "소스의 정직한 표현" 지점이다.

---

### 창구 내 제외된 사용자 메시지(일일이 명명하여 누락이 아님을 증명)

| 메시지 | 시간 | 주제 | 제외 사유 |
| --- | --- | --- | --- |
| #205 | 03-22 04:57 | 공용 / 내 곡 귀속 라벨 및 필터 | 소유권 체계(후보 사례 B)에 해당, 자막 수집이 아님 |
| #1252 제1항 | 05-03 07:57 | 팝업 내 스크롤 실패 | 순수 UI |
| #1252 제2항 | 05-03 07:57 | 노래 검색창 높이(최소 3곡 표시) | 순수 UI |
| #1278 | 05-04 14:00 | 카드 내 「공유」버튼 추가 | 공유 기능, 주제가 이미 이탈 |
| #1280 | 05-04 14:12 | 주제 분석 / 문화 테마 분포 / "자막" 버튼 클릭 불가 | 주체는 통계 분석; 그중 "자막 버튼 클릭 불가" 질문은 AI 응답이 취소됨(#1281에 `[Response was cancelled]`로 기록), 구현 증거가 없으므로 불산입 |
| #1282 | 05-04 15:04 | 가사 카드보내기(html2canvas) | 이미 별도 기능 라인 |

---

## 3. 코드 위치(`rg -n` / `code--view` 현장 검증, 기준 HEAD = `c897cc5d`)

### 3.0 HEAD의 폴백 체인 **실제** 계층(중요: 3단계가 아니라 "3소스 + 5단계")

논문 소절명은 "3단계 폴백"을 계승하되, 본문에 반드시 다음을 명시해야 한다: **"3단계"는 #1264 시점에 사용자가 정의한 3개 우선순위(LRCLIB / Supadata / AI)이다**; 41일의 진화를 거쳐 HEAD의 실제 실행 순서는 다음과 같으며 총 5단계, 3종 소스에 걸친다. 아래 순서는 `analyze-song` 메인 플로우와 `fetchYouTubeSubtitles`의 **실제 호출 순서**를 검증하여 얻은 것이다:

| 단계 | 소스 | 구현 위치 | 산출물 | 타임라인 |
| --- | --- | --- | --- | --- |
| L0 | 사용자 붙여넣기 SRT / 순수 텍스트(`custom_lyrics`) | `<C-analyze-song:2593-2602>`, 파서 `parseSRT` `<C-analyze-song:241>` / `parseSRTFormat` `<C-analyze-song:299>` | 정제 가사 + `timedEntries` | ✅ |
| L1 | Railway yt-dlp 서비스 | `<C-analyze-song:682-730>`; 독립 오케스트레이터 `<C-get-youtube-transcript:29-73,197-215>` | `timedEntries` | ✅ |
| L2 | YouTube 페이지 `captionTracks` 스크래핑 | `<C-analyze-song:732-770>`; 스크래퍼 `fetchCaptionTracksFromPage` `<C-analyze-song:358>`, 다운로더 `fetchSubtitleFromTrack` `<C-analyze-song:390>` | `timedEntries` | ✅ |
| L3 | 레거시 `timedtext` API | `<C-analyze-song:772-795>` | `timedEntries` | ✅ |
| L3.5 | Supadata | 메인 체인 외부 `<C-supabase/functions/fetch-transcript/index.ts:60-110>`; `get-youtube-transcript` 내부에서는 Strategy 3 `<C-get-youtube-transcript:141-168,235-248>` | `timedEntries` | ✅ |
| **L4** | **LRCLIB**(#1264의 "第一优先") | `<C-analyze-song:2619-2639>`, 구현 `fetchFromLRCLIB` `<C-analyze-song:546-657>` | **순수 텍스트 가사**, `timedEntries = []` | ❌ |
| L5 | AI 가사 검색(`searchLyricsWithAI`) | 제목 정제 `<C-analyze-song:2653-2661>` → 원본 제목 `<C-analyze-song:2663-2671>`; 구현 `<C-analyze-song:800,814>` | 순수 텍스트 가사 | ❌ |

> **독자에게 반드시 설명해야 할 "명실 불부합" 한 곳**: #1264에서 사용자는 LRCLIB을 **제1 우선순위**로 요구했으나, HEAD에서 LRCLIB의 트리거 조건은 `if (!hasCustomLyrics && !lyrics.trim() && cleanSearchTitle)` `<C-analyze-song:2625>`, 즉 **L1–L3이 모두 실패한 후에만 실행**된다. 이는 구현의 일탈이 아니라 AI가 #1275에서 명시적으로 인정하고 이유를 제시한 트레이드오프이다: LRCLIB은 `extractCleanTitle()`의 출력(추가 API 호출 1회)을 필요로 하며, `extractCleanTitle`은 영상 메타데이터에 의존하므로 자막 시도 이후에만 배치될 수 있다. `<V-msg#1275>`: "Acceptable for now: LRCLIB happens after subtitle attempts…". **논문에서 "LRCLIB이 제1 우선순위"를 인용한다면 반드시 이 한정을 보충해야 한다**: 이것은 "순수 텍스트 가사의 제1 우선순위"이지 전체 체인의 제1 우선순위가 아니다 — L1–L3은 타임라인이 있는 산출물로 가치가 더 높아 타임라인 없는 결과에 선점당해서는 안 되기 때문이다.

### 3.1 `supabase/functions/analyze-song/index.ts`(2903행; 전체 이력 **185회 커밋**)

| 블록 | 행 번호 | 본 사례와의 관계 |
| --- | --- | --- |
| `parseSRT` | 241–298 | 단계 A #225의 직접적 산물 |
| `parseSRTFormat` | 299–357 | SRT 변형(밀리초 구분자 `,` / `.`) 내결함 |
| `fetchCaptionTracksFromPage` | 358–389 | L2 |
| `fetchSubtitleFromTrack` | 390–421 | L2 |
| `lrclibSearchRaw` | 422–438 | #1270 구현 |
| `normalizeForMatch` | 439–448 | #1268「Stefanie Sun ↔ 孙燕姿」정규화 |
| `extractArtistFromVideoTitle` | 449–463 | #1269 계획의 Pass 3 |
| `pickBestLrclibMatch` | 464–524 | #1272「最相近的那个」 |
| `translateTitleArtistToEnglish` | 525–545 | #1264「AI 翻译成英文搜索一次」 |
| `fetchFromLRCLIB` | 546–657 | L4 본체, Pass 0 포함 |
| `fetchYouTubeSubtitles` | 659–798 | L1/L2/L3 오케스트레이션 |
| `searchLyricsWithAI` / `_searchLyricsAttempt` | 800–859 | L5 |
| `extractCleanTitle` | 860–898 | L4 선행 의존(L4의 위치를 결정) |
| `determineLyricsSource` | 2010–2015 | 소스 정직 표기: `ai_search` / `none` / `subtitle_<lang>` |
| `custom_lyrics` + SRT 파싱 | 2593–2602 | L0 |
| LRCLIB 재시도(clean title) | 2619–2639 | L4 트리거 지점 |
| 한국어 곡 자막 언어 검증 | 2642–2651 | 중국어 자막을 얻었으나 학습 대상이 한국어 → 능동 폐기 후 AI로 전환 |
| L5 2회(clean → raw) | 2653–2671 | 폴백의 폴백 |
| `add_only` 분기 | 2673–2700 | #1276 비차단 추가 |

> 설명: `analyze-song`의 185회 커밋은 **전체 수명 주기** 수치이며, 그 대부분은 사례 2(재개 가능한 처리)와 어휘표/문형 진화에 해당한다. 본 사례 창구 내 이 파일의 커밋은 **03-22: 25회**, **05-02~05-04: 8회**이다.

### 3.2 `supabase/functions/get-youtube-transcript/index.ts`(262행; **1회 커밋**, 2026-05-02)

| 블록 | 행 번호 |
| --- | --- |
| 언어 우선순위 상수(중국어 7단계 / 한국어 5단계) | 9–14 |
| `callYtDlpService`(30s AbortController) | 29–73 |
| Fallback 1: `fetchCaptionTracks` / `fetchTrack` / `fallbackCaptionTracks` | 76–138 |
| Fallback 2: `fallbackSupadata` | 141–168 |
| 3단계 오케스트레이션 + `check_only` 모드 | 170–253 |

**`check_only: true`(`<C-get-youtube-transcript:177,210-211,228,244>`)는 #1256에 의해 도태된 세대의 탐지 메커니즘의 화석이다**: "자막 유무만 판단하고 내용은 반환하지 않음"을 허용한다. #1259 이후 프런트엔드는 YouTube Data API 메타데이터로 전환하여 이 경로의 프런트엔드 호출자는 없지만 함수는 유지된다. 논문에서 "진화 잔재"의 실증으로 활용 가능하다.

### 3.3 `supabase/functions/fetch-transcript/index.ts`(124행; **11회 커밋**, 2026-03-22 → 2026-05-28)

단계 A #217의 산물(`189fb8bc` 등, 03-22 10:42)로 지금까지 유지된다. 주목할 점은 수명 주기 후반에 **인증**이 추가되었다는 것이다: `<C-fetch-transcript:16-32>` 주석에 `Require authenticated caller to prevent SUPADATA quota abuse`라고 명시 — 제3자 할당량 보호이며, #223「先做月亮代表我的心就好」의 동기와 같은 연원이다.

### 3.4 `src/hooks/useClientTranscript.ts`(175행; **6회 커밋**, 모두 2026-03-22 당일 집중)

| 블록 | 행 번호 | 관계 |
| --- | --- | --- |
| `autoFetch` 기본값 `false` | 65 | #223의 영구적 흔적 |
| auto-fetch 해제 시 즉시 return | 82–86 | 동일 |
| 디버그 로그(`startMs` / `durationMs` 샘플) | 41–50, 112–121 | #211 / #213이 지정한 중간량 |
| 적중 후 `song_analyses.full_analysis.timed_entries` + `songs.has_subtitles / lyrics_source` 회기 | 134–168 | 클라이언트 측 소스 표기 보완 |

**주의: 이 파일은 "아키텍처에서 도태되었으나 삭제되지 않은" 계층이다.** #215가 "stop trying the direct browser fetch approach"를 명령한 뒤, 파일 내 브라우저 직접 연결 로직은 **Edge Function 전달 호출**로 대체되었다(`fetchTranscriptViaEdgeFunction` `<C-useClientTranscript:13-57>`), 파일명 "client transcript"는 이미 명실이 부합하지 않는다. 이는 본 사례에서 가장 좋은 "설계 의도 vs 잔존 명명" 소재이다.

### 3.5 프런트엔드 탐지 계층

| 파일 | 행 수 | 전체 이력 커밋 | 창구 내 커밋 |
| --- | --- | --- | --- |
| `src/components/songs/YouTubeSearchDialog.tsx` | 353 | 32 | 03-22 기산 외, **05-02~05-04: 13회** |
| `src/components/lessons/SongPickerDialog.tsx` | 320 | 20 | **05-02~05-04: 10회** |
| `supabase/functions/youtube-search/index.ts` | 145 | 3 | **05-03: 1회** |
| `src/pages/Songs.tsx` | — | — | **05-04: 1회** |

`youtube-search`는 전체 수명 주기에 3회 커밋뿐이며, 그중 한 번이 #1256이 촉발한 "지연 없는 자막 탐지" 개조이다 — **탐지 계층의 변경량은 극소, 수익은 극대**(3–10초 대기 삭제). "올바른 채널 선택 > 알고리즘 최적화"의 논거로 사용 가능하다.

### 3.6 `src/components/songs/YouTubeConfirmDialog.tsx`(131행; 1회 커밋, 2026-03-27)

단계 A와 B 사이에 생겨난 수동 확인 관문(영상 및 언어 확인)으로, 폴백 체인에서 유일한 **인간 관문 전치** 위치이다. L0(수동 SRT 붙여넣기, 폴백 체인 말단의 인간)와 "한쪽 끝과 다른 쪽 끝의 두 인간 개입"이라는 대칭 구조를 이룬다.

---

## 4. 정량적 수정 라운드 통계

### 4.1 파일 수준(`git log --numstat`)

| 파일 | HEAD 행 수 | 전체 이력 커밋 | 최초 | 최종 |
| --- | --- | --- | --- | --- |
| `supabase/functions/analyze-song/index.ts` | 2903 | 185 | 2026-03-14 | 2026-05-27 |
| `src/components/songs/YouTubeSearchDialog.tsx` | 353 | 32 | 2026-03-15 | 2026-07-16 |
| `src/components/lessons/SongPickerDialog.tsx` | 320 | 20 | 2026-03-15 | 2026-05-19 |
| `supabase/functions/fetch-transcript/index.ts` | 124 | 11 | 2026-03-22 | 2026-05-28 |
| `src/components/songs/BulkRepairPanel.tsx` | 461 | 7 | 2026-04-25 | 2026-05-07 |
| `src/hooks/useClientTranscript.ts` | 175 | 6 | 2026-03-22 | 2026-03-22 |
| `supabase/functions/youtube-search/index.ts` | 145 | 3 | 2026-03-15 | 2026-05-28 |
| `supabase/functions/get-youtube-transcript/index.ts` | 262 | **1** | 2026-05-02 | 2026-05-02 |
| `supabase/functions/normalize-lyrics/index.ts` | 151 | 1 | 2026-04-17 | 2026-04-17 |
| `src/components/songs/YouTubeConfirmDialog.tsx` | 131 | 1 | 2026-03-27 | 2026-03-27 |

### 4.2 창구 내 변경량

| 창구 | 통계 범위 | 커밋 수(analyze-song) | 순증감 |
| --- | --- | --- | --- |
| 단계 A(03-22 종일) | `analyze-song` + `fetch-transcript` + `useClientTranscript.ts` | 25 | **+867 / −290** |
| 단계 B(05-02~05-04) | `analyze-song` + `get-youtube-transcript` + `youtube-search` + `YouTubeSearchDialog` + `SongPickerDialog` + `Songs.tsx` | 8 | **+821 / −143** |

**대비 해석(그래프로 바로 활용 가능)**: 양 단계의 코드 증가량은 유사하나(867 vs 821), **삭제량은 두 배 차이**(290 vs 143). 단계 A는 "시행착오형" — 쓰고 지우고 지우고 쓰기(브라우저 직접 연결 → CORS 벽 → 폐기); 단계 B는 "계약형" — 사용자가 검증된 방안과 인터페이스 계약을 가지고 들어와 삭제량이 현저히 감소. 이 수치는 **"인간의 선험 지식이 개입하는 위치가 앞설수록 재작업이 줄어든다"**는 정량 증거로 활용 가능하다.

### 4.3 일일 커밋 밀도

| 날짜 | 관련 파일 커밋 수 | 대응 사용자 메시지 |
| --- | --- | --- |
| 2026-03-22 | 25(analyze-song 기준) / 41(fetch-transcript, useClientTranscript 포함) | #207–#225(10건) |
| 2026-05-02 | 10 | #1238–#1250(7건) |
| 2026-05-03 | 13 | #1252–#1268(9건) |
| 2026-05-04 | 11 | #1270–#1276(4건) |

### 4.4 핵심 파라미터 진화표

| 파라미터 / 결정 | 초기값 | 최종값(HEAD) | 결정 주체 | 증거 |
| --- | --- | --- | --- | --- |
| Railway yt-dlp 타임아웃 | 30000ms | **30000ms(미변경)** | 인간(#1244); AI의 12000ms 제안 부결 | `<V-msg#1244>` `<V-msg#1261>` `<C-get-youtube-transcript:32,199>` `<C-analyze-song:690>` |
| 중국어 자막 언어 우선순위 | 사용자 지정 7단계 | 7단계(`zh, zh-Hans, zh-CN, zh-Hant, zh-TW, ko, en`) | 인간(#1242) | `<C-get-youtube-transcript:9-11>` |
| 한국어 자막 언어 우선순위 | — | 5단계(`ko, en, zh, zh-Hans, zh-CN`) | AI 보완 | `<C-get-youtube-transcript:12-14>` |
| Supadata 언어 우선순위 | — | 한국어: `ko,en`; 중국어: `zh, zh-Hans, zh-CN, zh-Hant, zh-TW, en`; 이후 auto 1회 | AI | `<C-fetch-transcript:51-54,91-110>` |
| 자막 유무 탐지 방식 | yt-dlp 사전 검사 호출(3–10s) | YouTube Data API `contentDetails.caption` + `captions.list`(0 대기) | 인간(#1256 제2문) | `<V-msg#1256>` `<V-msg#1259>` |
| 검색 결과 수 | 10 | 15 | AI(#1256 제1문에 응답) | `<V-msg#1259>` |
| 검색 키워드 구성 | 「中文 歌词」강제 결합 + `videoCategoryId=10` | 순수 사용자 키워드, 카테고리 제한 없음 | 인간(#1256 제1문) | `<V-msg#1259>` |
| LRCLIB 적중 최소 가사 길이 | — | **100자** | AI | `<C-analyze-song:569>` |
| LRCLIB 제목 일치 공차 | 엄격 일치 | 포함 관계이며 길이 차 **≤ 3** | AI(#1272에 응답) | `<C-analyze-song:571-573>` |
| LRCLIB 적중 아티스트명 최소 hint | — | **≥ 2**자 | AI | `<C-analyze-song:584>` |
| LRCLIB 검색 pass 수 | 1(title+artist) | 5 pass, Pass 0 = title-only | 인간(#1264가 2회 검색 정함 → #1272가 title-only 우선 정함) | `<V-msg#1264>` `<V-msg#1272>` `<C-analyze-song:559-657>` |
| 체인상 LRCLIB 위치 | 사용자 요구 "第一优先" | 실제로는 L1–L3 이후 | AI 트레이드오프 및 설명 | `<V-msg#1275>` `<C-analyze-song:2625>` |
| 타임라인 요구 | 필수(#207) | LRCLIB 분기에서 명시적 포기(`timedEntries: []`) | 인간(#1264) | `<V-msg#1264>` |
| 클라이언트 자동 자막 가져오기 | 기본 켜짐 | **기본 꺼짐**(`autoFetch = false`) | 인간(#223) | `<V-msg#223>` `<C-useClientTranscript:65>` |
| 곡 추가의 차단성 | 분석 완료까지 차단(30s+) | `add_only` 먼저 행 생성 + 백그라운드 분석 | 인간(#1260 제2항) | `<V-msg#1260>` `<C-analyze-song:2673-2700>` |
| Supadata 호출 인증 | 없음 | `Bearer` + `getClaims` 검증 요구 | — | `<C-fetch-transcript:16-32>` |

---

## 5. 핵심 전환점 표기

| 번호 | 메시지 / 커밋 | 시간 | 전환점인 이유 |
| --- | --- | --- | --- |
| **T1** | #215 | 03-22 10:18 | 사용자가 **이중 벽 귀인을 자력으로 완료**(클라우드 IP 봇 탐지 / 브라우저 CORS), AI가 추진 중이던 경로를 중단시키고 "2–3개 방안 + 4차원 평가 행렬"을 요구. 결정권이 명확히 인간 측에 남음. |
| **T2** | #225 | 03-22 11:16 | **인간을 폴백 체인 말단에 배치**. 자동화의 신뢰성 부족을 인정하고 "교사의 SRT 붙여넣기"를 궁극의 폴백으로 사용, 형식 명세와 수용 기준을 자체 지참. "3단계 폴백" 중 가장 반직관적이면서도 가장 내구성 있는 단계. |
| **T3** | #1238 | 05-02 13:50 | **41일간의 실사용으로 단계 A가 기준 미달임을 선언**, AI에게 모든 실패 방식의 복기를 요구; 동시에 "로컬 Python+yt-dlp 검증 완료"의 해답을 가지고 입장. |
| **T4** | #1256 제2문 | 05-03 08:09 | **"탐지"와 "취득"의 디커플링**. 자막 유무 판단을 메타데이터 채널로 전환하여 3–10초 사전 검사 삭제. |
| **T5** | #1262 + #1263 | 05-03 12:32 | 사용자가 기존 폴백 단계에 **신뢰성 질의**를 하여, AI의 자기부정과 "로그 없음"의솔직한 인정를 이끌어냄. 전체 프로젝트 아카이브에서 AI가 "데이터를 제시할 수 없다"고 인정한 소수의 기록 중 하나. |
| **T6** | #1264 | 05-03 14:29 | **전체 사례의 핵심: 소스 유형 전환**. 네 폴백 단계가 모두 동일 소스(YouTube)에 상관되어 동시에 실패함을 식별하고, 곡 데이터베이스에서 가사를 취하며 타임라인을 명시적으로 포기. 사용자는 **AI가 제시한 세 가지 선택지를 모두 부결**했다. |
| **T7** | #1272 | 05-04 13:43 | 사용자가 **최종 알고리즘을 구술**(title-only 우선 검색 + 아티스트명 유사도 최적 선택), 일대일로 `fetchFromLRCLIB` Pass 0에 구현됨. |

**일곱 전환점 중 여섯이 사용자에 의해 발기되었다(T5의 후반부를 제외한 T1–T7).** 이는 사례 3의 비율과 유사하지만, 본 사례가 더 특별한 점은: **T6가 "방안 선택"이 아니라 "문제 재정의"라는 것**이다 — AI가 제공한 세 가지 선택지는 모두 "YouTube 자막을 더 잘 스크래핑하는 방법"이라는 틀 안에 있었고, 사용자는 그 틀에서 벗어났다. 이것이 본 사례가 다른 세 사례와 구별되는, 논문에서 가장 전개할 가치가 있는 지점이다.

---

## 6. 변경 전 / 후 효과 증명: 검증 가능한 부분과 불가능한 부분

### 6.1 검증 가능한 "변경 전 상태"(모두 아카이브 내 실제 텍스트)

| 증상 | 출처 | 원문 |
| --- | --- | --- |
| 가사가 영상 자막에서 오지 않음 | #207 | "I suspect the current result may come from AI inference … rather than from the actual timestamped transcript" |
| 자동 스크롤 완전 불작동 | #211 / #213 | "auto-scroll still does not work"(2회) |
| 서버 측 3법 전멸 | #212(AI) | "服务端字幕抓取（page scraping + timedtext API + innertube API）**全部失败**…视频确实有 `zh-Hant` 字幕（通过 YouTube Data API `captions.list` 确认），但内容下载被阻断" |
| 이중 벽 귀인 | #215 | "1. server-side fetch is blocked by YouTube bot detection on cloud IPs 2. client-side fetch is blocked by YouTube CORS" |
| 하이라이트가 마지막 행에만 머묾 | #221 | "只显示最后一句是亮的。不管我点击哪一句都是从头播放" |
| 41일 후에도 기준 미달 | #1238 | "当然你现在的办法同样也不能准确下载" |
| 탐지는 있다고 하나 취득 불가 | #1260 | "我确定他是有字幕的，而且搜索的时候也是显示有字幕的，为什么最后分析的时候还是 ai 推测的歌词呢？" |
| 탐지 지나치게 느림 | #1256 | "现在显示是否有字幕的速度太慢了" |
| 추가 시 멈춤 | #1260 | "点击添加后，那个添加中的时间会特别长" |
| Supadata 장기 불신뢰 | #1262 | "我记得成功率非常不好，试过很多次都没办法成功获取到字幕" |
| 아티스트명 정규화 불일치 | #1268 | "歌名一样但是歌手名这个网站显示的是'Stefanie Sun'" |
| LRCLIB 개조 후에도 미적중 | #1272 | "我重新试了一下，还是没有拿到'我不难过'的正确歌词" |

### 6.2 검증 가능한 "변경 후 상태"

| 결론 | 근거 | 강도 |
| --- | --- | --- |
| LRCLIB이 「我不难过」에 적중하고 Stefanie Sun을 선택 | #1275 AI 보고: "LRCLIB now correctly hits with clean title and picks 'Stefanie Sun' as the matching artist" | 중(AI 자술 + 함수 로그 조회 주장, 로그 자체는 미보존) |
| 적중 후 「AI 추정」으로 표기되지 않음 | #1275: "`lyrics_source` set to `subtitle_lrclib` so the card won't show 'AI 추정'"; 코드 측 `<C-analyze-song:2010-2015>` `determineLyricsSource`와 `<C-analyze-song:2684>` `has_subtitles` 로직 일치 | 강(코드 검증 가능) |
| #1272 이후 "我不难过"에 대한 사용자 오류 보고 없음 | 아카이브 #1276은 "继续", #1278은 신주제로 전환 | 약(침묵이 성공을 의미하지 않음, 방증으로만) |
| yt-dlp 오케스트레이터 일회 완성 | `get-youtube-transcript/index.ts` 전체 수명 주기 1회 커밋, 262행 | 강(git 검증 가능) |
| 3단계 폴백이 HEAD에 온전히 생존 | 제3.0절의 5단계 표행별 검증 | 강(코드 검증 가능) |

### 6.3 확실한 근거를 찾을 수 없는 부분(있는 그대로 `<X>` 표기)

1. `<X>` **각 소스의 실제 성공률**. AI는 #1263에서 "두 edge function 모두 로그 기록이 없다"고 명확히 했고; #1263의 4행 성공률 표("高／中／低-中／中-低")는 AI의 **추정**이지 실측이 아니다. **논문에서 이 표를 데이터로 인용하는 것은 엄격히 금지되며**, "AI가 데이터 없이 정량적 외관의 표현을 제공한" 방법론 표본으로만 사용할 수 있다.
2. `<X>` **AI가 제안한 (A) 로그 통계 추가 후 1주일 운영**은 시행된 적이 없다 — 관련 커밋이 없고 `analyze-song` 내에 성공률 통계 코드가 없다.
3. `<X>` **스크린샷, 화면 녹화, 자동화 테스트**: 본 사례 창구 내에 테스트 파일 커밋이 없으며 `src/test/` 아래에 자막 관련 테스트 케이스가 없다.
4. `<X>` **Railway 서비스 측 기록**: `https://web-production-5712.up.railway.app`는 사용자 자체 구축 외부 서비스로, 배포 이력, 운영 로그, 현재 생존 상태가 모두 본 저장소에 없어 검증 불가. 논문에서 이 서비스를 기술해야 한다면 #1242의 인터페이스 계약 원문만 인용할 수 있다.
5. `<X>` **사용자의 명확한 "고쳐졌다" 회신**: 전체 창구에서 발견되지 않음. 가장 가까운 것은 #1276의 "继续"와 #1270의 "确认"이다.
6. `<X>` **#226–#1237(약 1012건)의조목별 정독**: 본 아카이브는 이 구간에서 키워드 검색만 수행했다(검색 적중 2건, 모두 해당 구간 외: #212와 #2234). 검색 도구는 모든 검색어의 동시 적중을 요구하여 재현율이 제한되므로, **이로써 해당 구간에 관련 메시지가 없다고 단언할 수 없다**. 답변(답변)에 "절대적 완전성"의 메시지 목록이 필요하다면 이 구간의조목별 정독을 보완해야 한다.
7. `<X>` **"3단계"라는 표현의 출처**: #1264에서 사용자가 3개 우선순위(LRCLIB / Supadata / AI)를 열거했고, #2234에서 사용자가 장 계획에 "字幕获取的三级兜底方案"로 기술했다. 그러나 제3.0절에서 보듯 HEAD 구현은 5단계이다. **논문 제목에 "3단계"를 유지해도 되지만 본문에 매핑 설명을 반드시 제공해야 한다** — 그렇지 않으면 독자가 제목에 따라 코드의 단계를 세다가 맞지 않게 된다.

---

## 7. 본 아카이브의 검증 체크리스트(답변에서조목별 재현용)

```bash
# 0. 기준
git log -1 --format='%h %ad' --date=format:'%Y-%m-%d %H:%M'   # 기대값 c897cc5d 2026-08-06 03:54

# 1. 단계 A 커밋군(03-22)
git log --format='%h|%ad|%s' --date=format:'%Y-%m-%d %H:%M' \
  --since=2026-03-22 --until=2026-03-23 \
  -- supabase/functions/fetch-transcript supabase/functions/analyze-song src/hooks/useClientTranscript.ts

# 2. 단계 B 커밋군(05-02~05-04)
git log --format='%h|%ad|%s' --date=format:'%Y-%m-%d %H:%M' \
  --since=2026-05-02 --until=2026-05-05 \
  -- supabase/functions/get-youtube-transcript supabase/functions/analyze-song \
     supabase/functions/youtube-search src/components/songs/YouTubeSearchDialog.tsx \
     src/components/lessons/SongPickerDialog.tsx src/pages/Songs.tsx

# 3. 양 단계 증감 대비(4.2절 수치)
git log --numstat --format='' --since=2026-03-22 --until=2026-03-23 \
  -- supabase/functions/analyze-song supabase/functions/fetch-transcript src/hooks/useClientTranscript.ts \
  | awk '{a+=$1;d+=$2} END{print "+"a" -"d}'      # 기대값 +867 -290
git log --numstat --format='' --since=2026-05-02 --until=2026-05-05 \
  -- supabase/functions/analyze-song supabase/functions/get-youtube-transcript \
     supabase/functions/youtube-search src/components/songs/YouTubeSearchDialog.tsx \
     src/components/lessons/SongPickerDialog.tsx src/pages/Songs.tsx \
  | awk '{a+=$1;d+=$2} END{print "+"a" -"d}'      # 기대값 +821 -143

# 4. get-youtube-transcript 일회 완성
git log --oneline -- supabase/functions/get-youtube-transcript/index.ts | wc -l   # 기대값 1

# 5. 30초 타임아웃 세 곳(AI 제안의 12s는 미구현)
rg -n "30000|12000" supabase/functions/get-youtube-transcript/index.ts supabase/functions/analyze-song/index.ts

# 6. 5단계 폴백의 코드 골격
rg -n "Strategy|LRCLIB|custom_lyrics|searchLyricsWithAI|parseSRT" supabase/functions/analyze-song/index.ts

# 7. #223의 영구적 흔적: autoFetch 기본 꺼짐
rg -n "autoFetch" src/hooks/useClientTranscript.ts

# 8. #1272 구술 알고리즘 → Pass 0
sed -n '559,601p' supabase/functions/analyze-song/index.ts
```

**아카이브 자체 점검 선언**: 본 문서의 모든 행 번호는 HEAD `c897cc5d`에서, 모든 커밋 해시와 시간은 `git log`에서, 모든 인용부호 내 중국어/영어 원문은 채팅 아카이브 제205–225, 1238–1284번에서축자 발췌했다. 검증 불가능한 부분은 제6.3절에 `<X>`로 명시적으로 표기했으며 보완 서술을 하지 않았다. 제1.1절에서 이미 선언했듯 본 아카이브는 "메시지 완전성"에서 사례 1~3보다 약한 범위를 가지므로, 논문에서 양자의 완전성을 동일하게 표현하지 말 것.
