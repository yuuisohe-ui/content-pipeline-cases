# content-pipeline-cases

박사학위논문 「한·중 노래기반 AI 바이브코딩 언어 교육 플랫폼 설계 및 구축」 5장 5.1절 **「대표적인 협업 사례와 사례 간 공통 규칙」**에서 다루는 다섯 가지 협업 사례의 **원자료 아카이브**입니다. 논문 본문의 서술이 어떤 대화, 커밋, 코드 위치에 근거하는지 그 기록을 함께 제시하기 위해 공개합니다.

이 저장소에는 실행 가능한 애플리케이션 코드가 없습니다. 각 사례가 다루는 코드는 플랫폼 저장소의 특정 시점(HEAD) 기준 위치로 문서 안에 기록되어 있습니다.

- 플랫폼: https://sino-song-learn.lovable.app
- 모든 모듈 최종 프롬프트: [thesis-prompts](https://github.com/yuuisohe-ui/thesis-prompts)
- GitHub 기반 Suno 동영상 생성 자동화: [melody-video-renderer](https://github.com/yuuisohe-ui/melody-video-renderer)

## 이 자료에서 말하는 협업

논문에서 협업은 프로그래밍 경험이 없는 연구자가 Lovable의 AI와 자연어 대화로 플랫폼을 개발하는 과정을 가리킵니다. 각 문서는 그 대화의 원문과 그에 따른 코드 변경 기록을 근거로 사례를 재구성합니다.

## 사례 목록

| 사례 | 논문 절 | 사례명 | 문서 |
|---|---|---|---|
| 사례 1 | 5.1.1 | 교육 콘텐츠 제시의 일관성 | [보기](docs/05-case1-timeline-alignment-evidence.ko.md) |
| 사례 2 | 5.1.2 | 교육 콘텐츠 주석의 완전성 | [보기](docs/05-case2-resumable-analysis-evidence.ko.md) |
| 사례 3 | 5.1.3 | 교육 콘텐츠 생성의 신뢰성 | [보기](docs/05-case3-async-suno-state-evidence.ko.md) |
| 사례 4 | 5.1.4 | 교육 콘텐츠 확보의 복원력 | [보기](docs/05-case4-subtitle-fallback-evidence.ko.md) |
| 사례 5 | 5.1.5 | 교육 콘텐츠 귀속의 경계 | [보기](docs/05-case5-ownership-tristate-evidence.ko.md) |

## 문서의 공통 구성

다섯 문서는 같은 골격을 따릅니다.

1. **시간 창 경계**: 분석 대상 기간의 시작과 끝
2. **사용자 메시지 축자 원문**: 해당 기간에 연구자가 AI에게 보낸 메시지 원문
3. **코드 위치**: 기준 HEAD를 명시하고, 줄 번호를 검색으로 대조한 코드 위치
4. **정량 수정 라운드 통계**
5. **핵심 전환점**
6. **효과 검증**: 검증 가능한 것과 검증 불가능한 것의 구분
7. **검증 체크리스트** (사례에 따라 절이 추가되거나 구성이 일부 다릅니다)

### 증거 등급 표기

문서 안의 주장에는 근거의 종류를 표시하는 기호가 붙어 있습니다.

| 기호 | 의미 |
|---|---|
| 〈V〉 | 축자 원문 (대화 원문을 그대로 인용) |
| 〈G〉 | 커밋 기록 |
| 〈C〉 | 코드 위치 |
| 〈X〉 | 검증 불가 |

### 포함된 원자료의 종류

대화 원문 메시지, 커밋 해시와 커밋 묶음, 줄 번호가 붙은 코드 위치, 소수의 코드 조각, 오류 메시지 인용, 정량 통계표.

## 사례별 관련 코드 위치

아래는 각 문서가 기록한 주요 코드 위치입니다. 경로는 플랫폼 저장소 기준이며, 기록 시점 이후 변경되었을 수 있습니다.

| 사례 | 주요 코드 위치 |
|---|---|
| 사례 1 | `src/features/song-player/lines.ts`, `LyricsHighlight.tsx`, Edge Function `sg-suno-lyrics` · `sg-repair-aligned`, `songs.aligned_words` 필드 |
| 사례 2 | Edge Function `analyze-song`, `src/lib/fetchWithRetry.ts`, `Songs.tsx` · `CsvImportDialog.tsx`, `song_analyses` 테이블 |
| 사례 3 | Edge Function `sg-suno-generate` · `sg-suno-poll` · `sg-suno-lyrics` · `sg-finalize-song` · `video-trigger` · `video-callback`, `src/lib/song-generator/suno-client.ts`, `SongGenerator.tsx`, `YoutubeVideoGenerateDialog.tsx` |
| 사례 4 | Edge Function `get-youtube-transcript` · `fetch-transcript`, `useClientTranscript.ts`, `YouTubeSearchDialog.tsx` · `YouTubeConfirmDialog.tsx` · `SongPickerDialog.tsx`, `BulkRepairPanel.tsx` |
| 사례 5 | `src/lib/ownership.ts`, `Songs.tsx` · `Lessons.tsx` · `Courses.tsx` · `Students.tsx`, `workspace/*Section.tsx`, `hidden_public_items` 테이블, `fork_public_item` · `move_to_trash` RPC |

## 이 저장소의 관리 방식

이 저장소는 논문 자료 원본 저장소에서 자동으로 동기화되는 **읽기 전용 미러**입니다. 이 저장소에서 직접 수정한 내용은 다음 동기화 때 덮어써집니다. 문서는 `docs/` 아래에 저장됩니다.
