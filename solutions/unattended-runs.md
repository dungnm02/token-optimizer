# Chạy không người trông (CI, cron, hook) (Tiếng Việt)

**Giải quyết:** Nguyên nhân 6.1, 6.5, 3.3 và 1.4 trong [`../CAUSE.md`](../CAUSE.md)

**Ý tưởng:** Một agent chạy trong CI không có ai để dừng nó lại, không có
cache ấm để kế thừa, và không có ngữ cảnh từ hôm qua. Mọi giả định trong
[`../WORKFLOW.md`](../WORKFLOW.md) đều sụp đổ. Hãy trả tiền cho việc bàn giao
ngữ cảnh, và đặt trần cứng thay cho phán đoán của con người.

---

## Vì sao phần này khác hẳn phần còn lại của kho

Toàn bộ `WORKFLOW.md` xây trên một giả định: có một người ngồi đó, thấy agent
đi sai hướng và bấm dừng. Bỏ người đó ra thì kinh tế học đổi hoàn toàn.

| Trong phiên tương tác | Trong lần chạy không người trông |
| --- | --- |
| Cache ấm dần qua nhiều lượt | **Mỗi lần chạy đều bắt đầu nguội** |
| Người dùng dừng vòng lặp hỏng | Vòng lặp chạy tới khi hết trần |
| Người dùng chỉ đúng file | Agent phải tự khám phá lại từ đầu |
| Chi phí tỷ lệ với số lập trình viên | **Chi phí tỷ lệ với số commit / PR / lần cron** |
| Bạn thấy hóa đơn khi đang gõ | Hóa đơn về sau, không ai đang nhìn |

Dòng cuối là lý do các thiết lập kiểu này hay gây bất ngờ. Một agent gắn vào
mọi lần push, trên một repo có 40 người, chạy nhiều hơn bất kỳ ai trong số 40
người đó — và không ai trong số họ nhìn thấy nó chạy.

## Ba khoản chi phí kết cấu

**1. Mỗi lần chạy trả giá đầy đủ cho phần prefix.** TTL cache tính bằng phút.
Một job CI chạy mỗi giờ gần như chắc chắn cache miss ở mọi lần — nghĩa là
system prompt, file chỉ dẫn và toàn bộ schema MCP đều được tính giá đầy đủ,
mỗi lần. Đây là nguyên nhân 1.4 ở dạng thuần khiết nhất, và nó khiến
[`instruction-file-hygiene.md`](instruction-file-hygiene.md) quan trọng hơn
nhiều so với trong phiên tương tác.

**2. Không có ai dừng vòng lặp hỏng.** Nguyên nhân 3.3 nói về retry và
polling. Trong phiên tương tác, bạn thấy agent thử lại lần thứ tư và bấm
Escape. Trong CI, nó thử tiếp cho tới khi chạm trần nào đó — và nếu bạn chưa
đặt trần, "nào đó" nghĩa là hạn mức tài khoản của bạn.

**3. Không có ký ức giữa các lần chạy.** Nguyên nhân 6.5. Mỗi job bắt đầu
từ con số không và phải khám phá lại cấu trúc repo. Trong phiên tương tác,
chi phí khám phá được khấu hao qua nhiều giờ. Trong CI, bạn trả nó lại từ
đầu, mỗi lần.

## Cách áp dụng

1. **Đặt trần cứng, trước tiên.** Trước cả khi tối ưu gì. Giới hạn số lượt,
   token và thời gian chạy ở tầng runner (`timeout-minutes` trong GitHub
   Actions là mức tối thiểu). Đây không phải tối ưu hóa — đây là cầu chì.
2. **Thu hẹp điều kiện kích hoạt.** Đừng chạy agent trên mọi lần push. Lọc
   theo đường dẫn, theo nhãn, theo nhánh. Phần lớn khoản tiết kiệm ở đây đến
   từ việc **không chạy**, không phải từ việc chạy rẻ hơn.
3. **Bàn giao ngữ cảnh thay vì bắt agent tự tìm.** Đây là nguyên nhân 6.1.
   Đưa thẳng vào prompt những gì đã biết chắc: diff, log lỗi, đường dẫn file
   liên quan, output của bước build đã chạy. Đừng để agent tốn mười lượt để
   khám phá lại thứ mà job đã có trong tay.
4. **Viết prompt tất định, không mời gọi khám phá.** "Sửa lỗi lint trong các
   file thuộc diff này" chứ không phải "cải thiện chất lượng code". Một
   prompt mơ hồ trong CI không có ai để hỏi lại, nên nó sẽ quét.
5. **Đặt điều kiện hoàn thành kiểm chứng được bằng máy.** Test pass, lệnh trả
   về 0. Không có nó, agent không biết lúc nào nên dừng, và nó sẽ dừng ở chỗ
   đắt nhất.
6. **Dùng model rẻ nhất còn qua được.** Không có người dùng nào đang chờ, nên
   không có áp lực về độ trễ. Đây là nơi lý tưởng nhất cho
   [`model-routing.md`](model-routing.md) và cho các bậc model thấp.
7. **Hạ mức reasoning effort.** Tác vụ CI thường hẹp và lặp lại. Đây gần như
   luôn là nút đúng để vặn xuống — xem [`reasoning-effort-tuning.md`](reasoning-effort-tuning.md).
8. **Tách khỏi cấu hình cá nhân.** Codex có `--ignore-user-config` và
   `--ignore-rules`. Một môi trường tự động nên tất định — và như một tác
   dụng phụ, nó không kéo theo các file chỉ dẫn cá nhân vào mọi lần chạy.
9. **Giới hạn mức song song.** Bung 20 job cùng lúc trên cache nguội là
   nguyên nhân 6.3 nguyên bản. Chạy một job làm ấm trước rồi mới bung, hoặc
   hạ trần concurrency — xem [`fan-out-warming.md`](fan-out-warming.md).
10. **Thử lại phần tất định, đừng thử lại agent.** Nếu job hỏng vì mạng, hãy
    retry bước mạng. Retry cả lượt gọi agent là trả tiền hai lần cho cùng một
    suy luận.
11. **Đừng dùng agent cho việc mà linter làm được.** Đây là khoản tiết kiệm
    lớn nhất trong tài liệu này và cũng dễ bị bỏ qua nhất. Định dạng, sắp xếp
    import, quy tắc đặt tên — đó là việc của công cụ tất định, giá bằng không.

## Cờ và cơ chế theo từng agent

| Agent | Chế độ không tương tác | Ghi chú |
| --- | --- | --- |
| Claude Code | `claude -p "<tác vụ>"` | `/mcp` cũng dùng được ở chế độ `-p`, in ra tóm tắt trạng thái dạng văn bản |
| Codex CLI | `codex exec "<tác vụ>"` | Tiến trình ra stderr, chỉ message cuối ra stdout; `--sandbox workspace-write`, `--ignore-user-config`, `--ignore-rules`; GitHub Action chính thức là `openai/codex-action` |
| Gemini CLI | `gemini -p "<tác vụ>"` | Cũng tự vào chế độ headless khi không phải TTY; `--output-format json` để ghép với `jq` |
| Cline | — | Là extension của IDE; không có đường chạy headless chính thức |

Việc "tiến trình ra stderr, kết quả ra stdout" của Codex đáng chú ý về mặt chi
phí: nó cho phép bạn ghép output vào bước tiếp theo mà không cần thêm một
lượt gọi LLM nữa chỉ để trích xuất kết quả.

## Cạm bẫy: tác vụ theo lịch trên phiên đang mở

Một cạm bẫy riêng, dễ tốn tiền âm thầm: **tác vụ hẹn giờ vẫn kích hoạt dù
phiên đang rảnh, và mỗi lần đều gửi lại toàn bộ context**.

Nghĩa là một tác vụ theo lịch gắn vào một phiên đã chạy cả ngày sẽ tính tiền
cho toàn bộ lịch sử của phiên đó, mỗi lần nó chạy — kể cả khi công việc thực
sự chỉ là kiểm tra một file. Hãy đặt tác vụ định kỳ trong phiên riêng, gọn,
chứ đừng gắn vào phiên bạn đang làm việc.

## Công cụ hiện đại nhất (SOTA)

### Có sẵn — coding agent & API của nhà cung cấp

| Nhà cung cấp / agent | Tính năng | Ghi chú |
| --- | --- | --- |
| Claude Code | `-p` (không tương tác) | Đường chạy headless chính thức |
| Codex CLI | `codex exec` | Kèm cờ sandbox và cờ bỏ qua cấu hình cho môi trường tự động |
| Codex | `openai/codex-action` | Action chính thức; chạy proxy an toàn cho API key thay vì đưa key vào bước shell |
| Gemini CLI | `-p` / stdin không phải TTY | `--output-format json` để ghép với công cụ khác |
| Batch API của nhà cung cấp | Bậc giá rẻ hơn cho khối lượng không tương tác | Xem [`batch-processing.md`](batch-processing.md) |

### Bên thứ ba — không phụ thuộc agent (ưu tiên mã nguồn mở)

| Công cụ | Giấy phép | Ghi chú |
| --- | --- | --- |
| Linter và formatter tất định | Khác nhau | Luôn rẻ hơn agent cho các luật máy kiểm được |
| Trần concurrency của runner CI | — | Cách rẻ nhất để chặn fan-out cache lạnh |
| Gateway LLM (LiteLLM, Helicone, Portkey) | Khác nhau | Trần chi tiêu theo key — cần thiết khi không ai đang nhìn |

## Đánh đổi

- **Trần cứng sẽ cắt ngang công việc thật.** Một job bị giết ở giữa vẫn tính
  tiền cho phần đã làm mà không cho bạn kết quả. Hãy đặt trần đủ rộng để tác
  vụ hợp lệ hoàn thành, đủ hẹp để vòng lặp hỏng chết sớm.
- **Bàn giao ngữ cảnh đắt lên phía trước.** Nhồi diff và log vào prompt làm
  request đầu tiên to hơn. Nó gần như luôn có lãi so với việc để agent tự
  khám phá, nhưng không phải luôn luôn — nếu diff dài mười nghìn dòng thì hãy
  tóm tắt hoặc lọc trước.
- **Model rẻ hơn thì tỷ lệ thất bại cao hơn.** Một job hỏng phải chạy lại
  bằng model đắt hơn sẽ tốn hơn là dùng thẳng model đúng ngay từ đầu.

## Tác động dự kiến

⚪ **Chưa đo.** Kho này chưa có phép đo A/B nào cho các lần chạy không người
trông.

Điều nói được về mặt cấu trúc: khoản tiết kiệm lớn nhất trong tài liệu này
gần như chắc chắn đến từ việc **thu hẹp điều kiện kích hoạt** và **không dùng
agent cho việc linter làm được** — cả hai đều loại bỏ toàn bộ lần chạy, chứ
không phải làm cho nó rẻ hơn một phần trăm nào đó. Đó là dạng tiết kiệm duy
nhất không có rủi ro đánh đổi chất lượng.

Trước khi tin bất kỳ con số nào bạn tự tính ra ở đây, hãy đọc
[`../MEASURE.md`](../MEASURE.md) — các lần chạy CI khác nhau rất nhiều về chi
phí tùy kích thước diff, nên chênh lệch giữa hai tuần thường là nhiễu chứ
không phải tín hiệu.

---

# Unattended runs (CI, cron, hooks)

**Addresses:** Causes 6.1, 6.5, 3.3 and 1.4 in [`../CAUSE.md`](../CAUSE.md)

**Idea:** An agent running in CI has nobody to stop it, no warm cache to
inherit, and no context from yesterday. Every assumption in
[`../WORKFLOW.md`](../WORKFLOW.md) breaks. Pay for context handoff, and
replace human judgment with hard ceilings.

---

## Why this differs from the rest of the repo

All of `WORKFLOW.md` rests on one assumption: a human is sitting there, sees
the agent going the wrong way, and stops it. Remove that human and the
economics change completely.

| In an interactive session | In an unattended run |
| --- | --- |
| The cache warms across turns | **Every run starts cold** |
| The user kills a broken loop | The loop runs until it hits a ceiling |
| The user points at the right file | The agent rediscovers everything |
| Cost scales with developer count | **Cost scales with commits / PRs / cron ticks** |
| You see the bill while typing | The bill arrives later, with nobody watching |

That last row is why these setups surprise people. An agent wired to every
push, on a repo with 40 contributors, runs more than any one of those 40
people — and none of them ever see it run.

## Three structural costs

**1. Every run pays full price for the prefix.** Cache TTL is measured in
minutes. A CI job that runs hourly will almost certainly miss the cache every
time — meaning the system prompt, the instruction files, and every MCP schema
are billed at full price, every run. This is cause 1.4 in its purest form, and
it makes [`instruction-file-hygiene.md`](instruction-file-hygiene.md) far more
important here than it is interactively.

**2. Nobody stops a broken loop.** Cause 3.3 covers retries and polling.
Interactively, you watch the agent try a fourth time and hit Escape. In CI it
keeps trying until it hits some ceiling — and if you haven't set one, "some
ceiling" means your account limit.

**3. No memory between runs.** Cause 6.5. Every job starts from zero and
rediscovers the repo structure. Interactively, discovery cost is amortized
across hours. In CI you pay it again, every time.

## How to apply

1. **Set hard ceilings first.** Before optimizing anything. Limit turns,
   tokens and wall time at the runner level (`timeout-minutes` in GitHub
   Actions is the bare minimum). This isn't an optimization — it's a fuse.
2. **Narrow the trigger.** Don't run the agent on every push. Filter by path,
   by label, by branch. Most of the saving available here comes from **not
   running**, not from running more cheaply.
3. **Hand over context instead of making the agent find it.** This is cause
   6.1. Put what you already know straight into the prompt: the diff, the
   error log, the relevant file paths, the output of the build step you
   already ran. Don't spend ten turns rediscovering what the job already had
   in hand.
4. **Write deterministic prompts that don't invite exploration.** "Fix the
   lint errors in the files in this diff," not "improve code quality." A
   vague prompt in CI has nobody to ask for clarification, so it scans.
5. **Give it a machine-checkable done-condition.** Tests pass, a command
   exits 0. Without one the agent doesn't know when to stop, and it will stop
   at the most expensive possible point.
6. **Use the cheapest model that passes.** No user is waiting, so there's no
   latency pressure. This is the single best place for
   [`model-routing.md`](model-routing.md) and lower model tiers.
7. **Lower the reasoning effort.** CI tasks are usually narrow and repetitive.
   This is almost always the right dial to turn down — see
   [`reasoning-effort-tuning.md`](reasoning-effort-tuning.md).
8. **Isolate from personal config.** Codex has `--ignore-user-config` and
   `--ignore-rules`. An automated environment should be deterministic — and
   as a side effect, it stops dragging personal instruction files into every
   run.
9. **Cap concurrency.** Fanning out 20 jobs at once against a cold cache is
   cause 6.3 verbatim. Warm with one job before fanning out, or lower the
   concurrency ceiling — see [`fan-out-warming.md`](fan-out-warming.md).
10. **Retry the deterministic part, not the agent.** If the job failed on a
    network blip, retry the network step. Retrying the whole agent call pays
    twice for the same reasoning.
11. **Don't use an agent for what a linter does.** This is the largest saving
    in this doc and the easiest to overlook. Formatting, import ordering,
    naming rules — that's deterministic tooling, and it costs zero.

## Per-agent flags and mechanisms

| Agent | Non-interactive mode | Notes |
| --- | --- | --- |
| Claude Code | `claude -p "<task>"` | `/mcp` also works in `-p` mode, printing a text status summary |
| Codex CLI | `codex exec "<task>"` | Progress to stderr, final message only to stdout; `--sandbox workspace-write`, `--ignore-user-config`, `--ignore-rules`; the official GitHub Action is `openai/codex-action` |
| Gemini CLI | `gemini -p "<task>"` | Also enters headless mode automatically in a non-TTY environment; `--output-format json` to pipe into `jq` |
| Cline | — | An IDE extension; no official headless path |

Codex's "progress to stderr, result to stdout" split is worth noting as a cost
property: it lets you pipe the output into the next step without a second LLM
call just to extract the result.

## A trap: scheduled tasks on an open session

One trap that quietly costs money: **a scheduled task fires on its interval
even while the session is idle, sending your full context each time**.

That means a scheduled task attached to a session that's been open all day is
billed for that session's entire history, every time it fires — even if the
actual work is checking one file. Put recurring tasks in their own lean
session rather than attaching them to the one you work in.

## SOTA tools

### Native — coding agents & provider APIs

| Provider / agent | Feature | Notes |
| --- | --- | --- |
| Claude Code | `-p` (non-interactive) | The official headless path |
| Codex CLI | `codex exec` | With sandbox and config-bypass flags for automated environments |
| Codex | `openai/codex-action` | Official action; runs a secure proxy for the API key instead of passing it into a shell step |
| Gemini CLI | `-p` / non-TTY stdin | `--output-format json` for piping into other tools |
| Provider batch APIs | Discounted tier for non-interactive volume | See [`batch-processing.md`](batch-processing.md) |

### Third-party — agent-agnostic (open source preferred)

| Tool | License | Notes |
| --- | --- | --- |
| Deterministic linters and formatters | Various | Always cheaper than an agent for machine-checkable rules |
| CI runner concurrency limits | — | The cheapest way to prevent cold-cache fan-out |
| LLM gateways (LiteLLM, Helicone, Portkey) | Various | Per-key spend ceilings — essential when nobody is watching |

## Trade-offs

- **Hard ceilings will cut off real work.** A job killed mid-run still bills
  for what it did without giving you a result. Set ceilings wide enough for
  legitimate tasks to finish and narrow enough that a broken loop dies early.
- **Context handoff costs more upfront.** Stuffing the diff and logs into the
  prompt makes the first request bigger. It's almost always a net win against
  letting the agent explore, but not always — if the diff is ten thousand
  lines, summarize or filter it first.
- **Cheaper models fail more often.** A job that fails and has to be re-run on
  a more expensive model costs more than using the right model up front.

## Expected impact

⚪ **Unmeasured.** This repo has no A/B measurement for unattended runs.

What can be said structurally: the largest savings in this doc almost
certainly come from **narrowing the trigger** and **not using an agent for
what a linter does** — both of which eliminate entire runs rather than making
a run some percentage cheaper. That's the only kind of saving with no quality
trade-off to worry about.

Before trusting any number you compute here, read
[`../MEASURE.md`](../MEASURE.md) — CI runs vary enormously in cost with diff
size, so a difference between two weeks is usually noise rather than signal.
