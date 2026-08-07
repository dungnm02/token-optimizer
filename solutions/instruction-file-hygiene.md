# Vệ sinh file chỉ dẫn (CLAUDE.md, AGENTS.md, .clinerules) (Tiếng Việt)

**Giải quyết:** Nguyên nhân 6.4, 2.3 và 1.3 trong [`../CAUSE.md`](../CAUSE.md)

**Ý tưởng:** File chỉ dẫn của bạn nằm trong **prefix cache** và chiếm chỗ
trong context window suốt cả phiên, dù nội dung đó có liên quan tới việc bạn
đang làm hay không. Hãy giữ nó chỉ gồm những sự thật *luôn đúng*, và đẩy mọi
thứ theo tình huống sang skill nạp-theo-yêu-cầu.

---

Đây là bề mặt chi phí mà bạn đụng vào nhiều nhất và ít khi đo nhất. Mỗi agent
có một file:

| Agent | File | Nạp khi nào |
| --- | --- | --- |
| Claude Code | `CLAUDE.md` | Đầu phiên |
| Codex CLI | `AGENTS.md` | Đầu phiên |
| Gemini CLI | `GEMINI.md` | Đầu phiên |
| Cline | `.clinerules` (file hoặc thư mục) | Nối vào system prompt |

## Vì sao nó tốn kém hơn vẻ ngoài

Một file 400 dòng nghe có vẻ nhỏ. Ba lý do khiến nó không nhỏ:

1. **Nó chiếm chỗ vĩnh viễn.** Không như một kết quả tool có thể bị cắt tỉa,
   file chỉ dẫn ở lại trong context window từ đầu tới cuối phiên. Nó lấy đi
   dư địa mà lẽ ra dành cho code thật.
2. **Bạn trả giá đầy đủ cho nó ở mỗi lần cache miss.** Nhờ prompt caching,
   phần lớn thời gian nó được tính ở mức cache read rẻ. Nhưng mỗi lần cache
   nguội — nghỉ giữa giờ, đổi ngày, hết TTL — bạn trả giá đầy đủ để nạp lại
   toàn bộ, và bạn trả cho **cả file**, kể cả 380 dòng không liên quan.
3. **Nó nhân lên theo số agent.** Trên Claude Code, mỗi thành viên trong một
   đội agent tự nạp `CLAUDE.md`, MCP server và skill. Một file phình to không
   tốn thêm một lần — nó tốn thêm *n* lần.

Và cái bẫy khó chịu nhất: **sửa file này giữa phiên sẽ phá cache prefix**
(nguyên nhân 1.3). Sửa một dòng ở dòng 12 và mọi thứ phía sau nó phải xây lại
từ đầu. Đó là lý do một quy ước trong [`../WORKFLOW.md`](../WORKFLOW.md) nói
hãy đóng băng file chỉ dẫn giữa task.

## Sự thật khó chịu: model không bắt buộc phải nghe theo

Trước khi tối ưu kích thước, hãy đối diện với điều này. File chỉ dẫn là
**gợi ý, không phải cưỡng chế**. Model đọc chúng và có thể bỏ qua.

[`../PROOF.md`](../PROOF.md) ghi lại một kết quả đo trực tiếp về chuyện này:
tầng rules-file thụ động của ponytail đã kích hoạt **0 lần trong 10 phiên**.
Cùng bộ hướng dẫn đó, khi gắn qua hook tất định, đo được **−10,3%**
(p = 0,004). Cơ chế gắn kết quyết định tất cả — chứ không phải nội dung.

Suy ra hai điều:

- Đừng viết một luật vào file chỉ dẫn rồi coi như xong. Hãy **kiểm chứng nó
  có kích hoạt không**: giao một tác vụ mời gọi vi phạm luật đó và xem agent
  có tuân không.
- Nếu một luật quan trọng đến mức *phải* được tuân thủ, hãy chuyển nó xuống
  tầng cưỡng chế được — hook, lệnh CI, linter — thay vì thêm dòng in đậm.

## Cách áp dụng

1. **Đặt ngân sách dòng, rồi giữ nó.** Tài liệu Claude Code khuyên giữ
   `CLAUDE.md` **dưới 200 dòng, chỉ gồm phần thiết yếu**. Hãy dùng đó làm
   trần cho cả bốn agent cho tới khi bạn có số đo của riêng mình.
2. **Áp dụng phép thử "luôn đúng".** Với mỗi khối trong file, hỏi: *nội dung
   này có liên quan tới hơn 80% số tác vụ tôi giao trong repo này không?*
   Nếu không, nó không thuộc về đây. Quy trình review PR, các bước migration
   database, cách phát hành — tất cả đều trượt bài kiểm tra này.
3. **Chuyển phần trượt bài kiểm tra sang skill.** Thân của một skill chỉ nạp
   khi được dùng, nên tài liệu tham chiếu dài gần như không tốn gì cho tới
   lúc bạn cần. Đây là cách đổi một chi phí *thường trực* lấy một chi phí
   *thỉnh thoảng*.
4. **Chỉ đường, đừng nhúng.** Đừng dán nội dung schema, ví dụ config hay
   danh sách API vào file chỉ dẫn. Ghi đường dẫn. Agent đọc được file, và
   khi đó nó chỉ trả tiền cho những file thực sự cần.
5. **Cắt phần agent tự suy ra được.** "Đây là dự án TypeScript dùng React"
   là lãng phí — agent thấy `package.json`. Hãy giữ những gì nó **không**
   đoán được: quy ước nội bộ, những cái bẫy đã biết, lý do vì sao thứ trông
   như sai lại đang đúng.
6. **Loại bỏ mâu thuẫn và trùng lặp.** Chỉ dẫn ở cấp người dùng, cấp dự án
   và cấp thư mục cộng dồn với nhau. Cùng một luật viết ba nơi vẫn bị tính
   tiền ba lần (nguyên nhân 2.3), và các luật đá nhau khiến agent tốn token
   reasoning để phân xử.
7. **Loại các cây thư mục không nên quét.** Trên Claude Code,
   `claudeMdExcludes` ngăn việc gom các file memory nằm trong `node_modules`,
   `vendor` và các thư mục tương tự. Cline có `.clineignore` với vai trò
   tương tự cho việc đọc file.
8. **Đóng băng giữa task.** Cần sửa? Sửa lúc bắt đầu phiên, chưa phải giữa
   chừng (nguyên nhân 1.3).
9. **Tỉa nó như tỉa bất kỳ prompt nào.** Xóa thử một khối, chạy lại bộ tác
   vụ đại diện của bạn, và nếu chất lượng không giảm thì giữ nguyên việc xóa.
   Đó chính là quy trình trong [`prompt-de-scaffolding.md`](prompt-de-scaffolding.md).

## Cái gì nên ở đâu

| Nội dung | Đặt vào |
| --- | --- |
| Lệnh build/test/lint | File chỉ dẫn — dùng ở gần như mọi tác vụ |
| Quy ước code trái với mặc định | File chỉ dẫn — ngắn gọn |
| Cạm bẫy đã biết ("đừng sửa file này, nó tự sinh") | File chỉ dẫn |
| Bản đồ kiến trúc | File chỉ dẫn nếu ngắn; nếu không thì một skill hoặc [`code-maps.md`](code-maps.md) |
| Quy trình nhiều bước (release, migration, review) | Skill — nạp theo yêu cầu |
| Tài liệu tham chiếu dài, bảng tra cứu | Skill hoặc file trên đĩa, có ghi đường dẫn |
| Nội dung schema, đặc tả API | File trên đĩa, chỉ ghi đường dẫn |
| Luật bắt buộc phải tuân thủ | Hook / CI / linter — **không phải** file chỉ dẫn |

## Công cụ hiện đại nhất (SOTA)

### Có sẵn — coding agent & API của nhà cung cấp

| Nhà cung cấp / agent | Tính năng | Ghi chú |
| --- | --- | --- |
| Claude Code | Skill (`SKILL.md`) | Thân skill chỉ nạp khi được dùng — cách chính thức để chuyển chi phí thường trực thành chi phí theo yêu cầu |
| Claude Code | `/context` | Cho thấy chính xác file memory đang chiếm bao nhiêu chỗ; nó gắn cờ cả tình trạng phình memory |
| Claude Code | `/usage` (phân bổ) | Quy phần trăm mức dùng gần đây về từng skill, subagent và plugin |
| Claude Code | `claudeMdExcludes` | Chặn việc gom file memory từ các cây thư mục dependency |
| Claude Code | Hook | Tầng cưỡng chế cho những luật thực sự phải được tuân thủ |
| Cline | `.clineignore` | Giới hạn phạm vi file mà agent được đọc |

### Bên thứ ba — không phụ thuộc agent (ưu tiên mã nguồn mở)

| Công cụ | Giấy phép | Ghi chú |
| --- | --- | --- |
| promptfoo | MIT | Tỉa file chỉ dẫn theo lối thực nghiệm: xóa một khối, chạy bộ eval, giữ việc xóa nếu điểm không giảm |
| Bộ đếm token của nhà cung cấp | — | Đo file chỉ dẫn trực tiếp thay vì đếm dòng; xem [`token-counting.md`](token-counting.md) |

## Đánh đổi

- **Cắt quá tay thì agent hỏi lại nhiều hơn**, và mỗi câu hỏi là một
  round-trip mang theo toàn bộ context. Một file chỉ dẫn quá gầy có thể đắt
  hơn một file vừa đủ. Đây là cùng một đường cong hình chữ U đã mô tả trong
  [`../CONFIG.md`](../CONFIG.md).
- **Chuyển sang skill không miễn phí.** Metadata của skill vẫn tốn một ít
  chỗ để agent biết là skill đó tồn tại. Lợi ích đến từ việc phần thân dài
  không được nạp — nên chỉ đáng làm với nội dung thực sự dài.
- **Chia theo thư mục làm tăng chi phí bảo trì.** Chỉ dẫn theo từng thư mục
  giữ context gọn, nhưng lại tạo thêm nơi để luật lệ trôi dạt khỏi nhau.

## Tác động dự kiến

⚪ **Chưa đo.** Phần tiết kiệm hoàn toàn phụ thuộc file hiện tại của bạn to
cỡ nào — cắt một file 150 dòng vốn đã gọn thì gần như không được gì.

Cách tự ước lượng, và là cách duy nhất đáng tin: chạy `/context` (hoặc bộ đếm
token tương đương), ghi lại phần chỗ mà file chỉ dẫn đang chiếm, rồi nhân với
số lần bạn khởi động phiên mới trong tuần. Nếu con số đó không đáng kể, hãy
bỏ qua giải pháp này và dành công sức cho chỗ khác. Xem
[`../MEASURE.md`](../MEASURE.md) để biết cách kiểm chứng một lần cắt thực sự
có tác dụng.

---

# Instruction-file hygiene (CLAUDE.md, AGENTS.md, .clinerules)

**Addresses:** Causes 6.4, 2.3 and 1.3 in [`../CAUSE.md`](../CAUSE.md)

**Idea:** Your instruction file lives in the **cache prefix** and occupies
context-window space for the entire session, whether or not its contents
relate to what you're doing. Keep only facts that are *always true*, and push
everything situational into on-demand skills.

---

This is the cost surface you touch most often and measure least. Every agent
has one:

| Agent | File | Loaded when |
| --- | --- | --- |
| Claude Code | `CLAUDE.md` | Session start |
| Codex CLI | `AGENTS.md` | Session start |
| Gemini CLI | `GEMINI.md` | Session start |
| Cline | `.clinerules` (file or directory) | Concatenated into the system prompt |

## Why it costs more than it looks

A 400-line file sounds small. Three reasons it isn't:

1. **It occupies space permanently.** Unlike a tool result, which can be
   pruned, the instruction file stays in the context window from the first
   turn to the last. It takes headroom that should have gone to actual code.
2. **You pay full price for it on every cache miss.** Thanks to prompt
   caching, most of the time it's billed at the cheap cache-read rate. But
   every time the cache goes cold — a break, a new day, TTL expiry — you pay
   full price to reload all of it, including the 380 lines that were
   irrelevant.
3. **It multiplies by agent count.** On Claude Code, every member of an agent
   team loads `CLAUDE.md`, MCP servers, and skills on its own. A bloated file
   doesn't cost once more — it costs *n* times more.

And the nastiest trap: **editing it mid-session breaks the cache prefix**
(cause 1.3). Change one word on line 12 and everything after it rebuilds from
scratch. That's why one of the conventions in
[`../WORKFLOW.md`](../WORKFLOW.md) is to freeze instruction files mid-task.

## The uncomfortable fact: the model doesn't have to obey

Before optimizing size, face this. Instruction files are **advisory, not
enforced**. The model reads them and can ignore them.

[`../PROOF.md`](../PROOF.md) records a direct measurement of exactly this:
ponytail's passive rules-file tier fired **0 times in 10 sessions**. The same
ruleset, attached through a deterministic hook, measured **−10.3%**
(p = 0.004). The attachment mechanism decided the outcome — not the content.

Two things follow:

- Don't write a rule into an instruction file and call it done. **Verify that
  it fires**: give the agent a task that invites breaking the rule and see
  whether it complies.
- If a rule matters enough that it *must* hold, move it to a layer that can
  enforce it — a hook, a CI check, a linter — instead of adding another bold
  line.

## How to apply

1. **Set a line budget and hold it.** The Claude Code documentation
   recommends keeping `CLAUDE.md` **under 200 lines, essentials only**. Use
   that as the ceiling on all four agents until you have your own numbers.
2. **Apply the "always true" test.** For each block in the file, ask: *does
   this apply to more than 80% of the tasks I run in this repo?* If not, it
   doesn't belong. PR review procedures, database migration steps, release
   processes — they all fail this test.
3. **Move what fails into skills.** A skill's body loads only when it's used,
   so long reference material costs almost nothing until you need it. This is
   how you trade a *standing* cost for an *occasional* one.
4. **Point, don't embed.** Never paste schema contents, config examples, or
   API listings into the instruction file. Write the path instead. The agent
   can read files, and then it pays only for the ones it actually needs.
5. **Cut whatever the agent can infer.** "This is a TypeScript project using
   React" is wasted — the agent can see `package.json`. Keep what it
   **can't** guess: internal conventions, known traps, why the thing that
   looks wrong is actually correct.
6. **Remove contradictions and duplicates.** User-level, project-level and
   directory-level instructions stack. The same rule in three places is
   billed three times (cause 2.3), and rules that fight each other spend
   reasoning tokens getting adjudicated.
7. **Exclude trees that shouldn't be scanned.** On Claude Code,
   `claudeMdExcludes` stops memory files inside `node_modules`, `vendor` and
   similar directories from being collected. Cline's `.clineignore` plays the
   same role for file reads.
8. **Freeze it mid-task.** Need to change it? Change it at the start of a
   session, not halfway through (cause 1.3).
9. **Prune it like any other prompt.** Delete a block, re-run your
   representative task set, and if quality holds, keep the deletion. That's
   the procedure in [`prompt-de-scaffolding.md`](prompt-de-scaffolding.md).

## What belongs where

| Content | Put it in |
| --- | --- |
| Build/test/lint commands | Instruction file — used on nearly every task |
| Code conventions that contradict the defaults | Instruction file — briefly |
| Known traps ("don't edit this file, it's generated") | Instruction file |
| Architecture map | Instruction file if short; otherwise a skill or [`code-maps.md`](code-maps.md) |
| Multi-step procedures (release, migration, review) | A skill — loaded on demand |
| Long reference material, lookup tables | A skill, or a file on disk with the path noted |
| Schema contents, API specs | A file on disk; write the path only |
| Rules that must be obeyed | A hook / CI / linter — **not** the instruction file |

## SOTA tools

### Native — coding agents & provider APIs

| Provider / agent | Feature | Notes |
| --- | --- | --- |
| Claude Code | Skills (`SKILL.md`) | Skill bodies load only when used — the sanctioned way to convert a standing cost into an on-demand one |
| Claude Code | `/context` | Shows exactly how much space memory files occupy; it also flags memory bloat |
| Claude Code | `/usage` (attribution) | Assigns a share of recent usage to each skill, subagent and plugin |
| Claude Code | `claudeMdExcludes` | Stops memory-file collection from dependency trees |
| Claude Code | Hooks | The enforcement layer for rules that genuinely must hold |
| Cline | `.clineignore` | Bounds which files the agent may read |

### Third-party — agent-agnostic (open source preferred)

| Tool | License | Notes |
| --- | --- | --- |
| promptfoo | MIT | Prune instruction files empirically: delete a block, run the eval set, keep the deletion if scores hold |
| Provider token counters | — | Measure the instruction file directly instead of counting lines; see [`token-counting.md`](token-counting.md) |

## Trade-offs

- **Cut too far and the agent asks more questions**, and each question is a
  round-trip carrying the whole context. An instruction file that is too thin
  can cost more than one that is right-sized. This is the same U-curve
  described in [`../CONFIG.md`](../CONFIG.md).
- **Moving to skills isn't free.** Skill metadata still occupies a little
  space so the agent knows the skill exists. The win comes from the long body
  not being loaded — so it only pays off for genuinely long content.
- **Directory-scoped splitting raises maintenance cost.** Per-directory
  instructions keep context lean, but they create more places for rules to
  drift apart.

## Expected impact

⚪ **Unmeasured.** The saving depends entirely on how big your current file
is — trimming an already-lean 150-line file gains essentially nothing.

How to estimate it yourself, and the only trustworthy way: run `/context` (or
the equivalent token counter), record the share the instruction file
occupies, and multiply by how many fresh sessions you start per week. If that
number is negligible, skip this solution and spend the effort elsewhere. See
[`../MEASURE.md`](../MEASURE.md) for how to verify that a cut actually helped.
