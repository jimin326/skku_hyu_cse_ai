# 제출본 — `사자먹는은행_v1.js`

대회 사이트에 `사자먹는은행_v1.js` 로 업로드한 봇(2026-09-05 오후, 스킬 공개 뒤). `Lion_Eating_Bank_v13_1.js`(v13 + 스킬 어댑터 v2)가 기반이다.

봇 로직(§1 Thunder 서브 · §2 ACCore 랠리 · §3 오케스트레이터)은 v13_1 과 같다. 바뀐 곳은 파일 앞부분의 **스킬 어댑터 구역**(`SK` 노브 · `SK_ST` 상태 · `applySkill`)뿐이며, 대회 당일 공개된 발톱(claw) 스킬에 맞춰 아래와 같이 확장했다.

## v13_1 → 제출본 변경 내용

### 1. 어댑터를 발톱 규칙에 맞춰 켬 (`SK` 노브)

v13_1 의 어댑터는 스킬이 어떤 형태일지 몰라 자리만 잡아 둔 상태(`on: false`, `key: 'skill'`, `full: 100`)였다. 공개된 규칙 코드(`skill/claw.js` · `skill/gauge.js` · `bot/botContract.js`)에 맞춰 값을 확정했다.

| 노브 | v13_1 | 제출본 | 근거 |
|---|---|---|---|
| `on` | false | **true** | 어댑터 활성 |
| `key` | `'skill'` | **`'skillX'`** | 발동은 반환 객체의 `skillX`(조준 x 좌표, 0~432). `fire` 는 0 으로 두어 좌표 대신 1 이 나가는 일을 막음 |
| `full` | 100 | **55** | 발톱 비용. 런타임엔 `config.claw.cost` 를 우선 읽음 |
| `claw` | — | **1** | 발톱 시전 켬(§2) |

`SK_ST` 에는 `casts` · `errors` 카운터를 추가했다.

### 2. 시전 정책 (`skCastX`, 새 함수)

발톱은 시전 25프레임 뒤 `centerX ± 62` 안에 서 있는 상대를 기절시킨다. `skCastX(snapshot)` 는 내 게이지가 비용(`config.claw.cost`) 이상이고, 내 발톱이 비행 중이 아니고, 내가 누워 있지 않으면(state < 4) 상대의 현재 x 를 조준점으로 돌려준다. 조건이 하나라도 안 맞으면 `null`(시전 안 함).

### 3. `applySkill` 배선

최종 출력 직전에 한 번 불리는 `applySkill` 안에서, `config.claw` 가 있는 엔진일 때만 `skCastX` 를 호출해 반환 객체에 `skillX` 를 붙인다. `config.claw` 가 없는 엔진(구 저장소, `shadow_diff`, `sk_v2_test`)에서는 아무것도 하지 않아 v13 과 출력이 같다.

### 4. 당일 노브

| 노브 | v13_1 | 제출본 | 효과 |
|---|---|---|---|
| `DEBUG` | true | **false** | F12 로그 끔 |

## 검증 (새 엔진 + 실제 발톱 코드, `tools/`)

- `rule_check`: throws 0, invalid 0, >120ms 0, p99 15ms. 로드 · 크기 정상.
- 게이트의 `shadow_diff` 는 v13 대비 불일치가 있다 — 발톱 시전(`skillX`)에 의한 의도된 차이. `--allow-shadow-diff` 로 승인해 실행한다.

```bash
ENGINE_ROOT=../engine node --no-warnings tools/dayof/gates.mjs "bot/submitted/사자먹는은행_v1.js" --base Lion_Eating_Bank_v13 --allow-shadow-diff "제출본"
```
