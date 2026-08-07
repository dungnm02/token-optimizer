# Hóa đơn: tiết kiệm token thực ra mua được gì (Tiếng Việt)

Mọi con số phần trăm trong kho này đều ngầm giả định một điều: bạn **trả tiền
theo token**. Với phần lớn người đọc, giả định đó sai.

Nếu bạn chạy Claude Code bằng gói Max, Codex bằng ChatGPT Plus, hay Gemini
CLI ở tier Code Assist, thì tiết kiệm 30% token **không** làm hóa đơn giảm
30%. Hóa đơn của bạn là một con số cố định hàng tháng. Cái bạn vừa mua được
là thứ khác: thêm dư địa trước khi chạm trần, thêm phiên làm việc trước khi
hết hạn mức tuần.

Tài liệu này nói rõ bạn đang tiêu **đơn vị gì**, vì đơn vị đó quyết định
nguyên nhân nào trong [`CAUSE.md`](CAUSE.md) thực sự đắt với bạn — và câu
trả lời khác nhau tùy thiết lập.

> **Trạng thái bằng chứng:** ⚪ Các sự kiện về gói cước, cửa sổ hạn mức và
> hạn ngạch bên dưới đều dẫn nguồn từ tài liệu nhà cung cấp (xem
> [Nguồn](#nguồn)). Phần diễn giải — nguyên nhân nào đắt hơn dưới đơn vị
> nào — là suy luận từ chính các sự kiện đó, **chưa được đo**. Nhà cung cấp
> đổi giá và đổi hạn mức thường xuyên; hãy kiểm lại nguồn trước khi lập kế
> hoạch ngân sách.

## Bạn đang tiêu đơn vị gì?

| Thiết lập | Đơn vị khan hiếm | Tiết kiệm token đổi được gì |
| --- | --- | --- |
| Claude Code + API key / Bedrock / Vertex / Foundry | **Đô la**, theo token | Tiền thật, tỷ lệ 1:1 |
| Claude Code + gói Pro / Max / Team / Enterprise | **Hạn mức phiên & tuần** | Dư địa: làm được lâu hơn trước khi bị chặn |
| Codex + API key | **Đô la**, theo token | Tiền thật, tỷ lệ 1:1 |
| Codex + gói ChatGPT (Go/Plus/Pro/Business) | **Credit trong cửa sổ 5 giờ** | Dư địa, rồi mới tới tiền khi mua thêm credit |
| Gemini CLI + Code Assist | **Số request mỗi ngày** | Gần như *không gì cả* — xem mục dưới |
| Gemini CLI + API key trả phí | **Đô la**, theo token | Tiền thật, tỷ lệ 1:1 |
| Cline (BYOK) | **Đô la**, theo token | Tiền thật, tỷ lệ 1:1 |

Cline là agent duy nhất trong bốn agent mà thiết lập mặc định luôn là tính
tiền theo token: bản mã nguồn mở không có phí thuê bao hay phí ghế, bạn trả
đúng giá nhà cung cấp cho model mình chọn. Với Cline, mọi phần trăm trong kho
này quy đổi thẳng thành đô la.

## Ba đơn vị, ba thứ tự ưu tiên khác nhau

Đây là phần quan trọng nhất của tài liệu. Cùng một nguyên nhân, mức độ đau
khác nhau hoàn toàn tùy đơn vị bạn đang tiêu.

| Nguyên nhân | Trả theo token (đô la) | Gói thuê bao (hạn mức) | Hạn ngạch request (Gemini) |
| --- | --- | --- | --- |
| 1.1/1.4 Hỏng caching | Rất đắt — cache read rẻ hơn nhiều lần | Rất đắt — cache miss bị gắn cờ trong `/usage` | Không đáng kể |
| 2.1 Lịch sử không giới hạn | Đắt, tăng theo bậc hai | Đắt — lý do số một khiến hạn mức tuần bốc hơi | Không đáng kể |
| 3.1 Output tool quá lớn | Đắt | Đắt | **Miễn phí** |
| 3.2 Round-trip vụn vặt | Vừa phải | Vừa phải | **Đắt nhất** — mỗi round-trip là một request |
| 3.3 Vòng lặp retry/polling | Vừa phải | Vừa phải | **Đắt nhất** — cùng lý do |
| 5.1 Token reasoning | Đắt — tính như output token | Đắt | Không đáng kể |
| 5.2 Output dài dòng | Đắt | Đắt | **Miễn phí** |
| 6.2 Model quá cỡ | Đắt — chênh lệch giá theo bậc | Đắt — model cao cấp rút hạn mức nhanh hơn | Không đáng kể |

Đọc bảng này theo cột của bạn, đừng đọc ngang.

### Nghịch lý Gemini: khi output rẻ mà lượt gọi thì đắt

Hạn ngạch Gemini Code Assist tính bằng **request**, không phải token:
1.500 request/ngày cho mỗi người dùng ở bản Standard, 2.000 ở bản Enterprise,
và trần 2 request/giây.

Điều đó đảo ngược gần hết cẩm nang này. Nếu đơn vị khan hiếm là request thì:

- Siết `truncateToolOutputThreshold` **không** giúp bạn tiết kiệm hạn ngạch.
- Một câu trả lời dài dòng tốn đúng bằng một câu trả lời cô đọng: một request.
- Nhưng một agent gọi tool 40 lần thay vì 8 lần thì tốn **gấp năm** hạn ngạch.

Trên tier đó, hãy đọc [`solutions/tool-composition.md`](solutions/tool-composition.md)
và [`solutions/event-driven-waiting.md`](solutions/event-driven-waiting.md)
trước; bỏ qua toàn bộ nhánh nén output cho tới khi bạn chuyển sang API key
trả phí. Lưu ý hạn ngạch này có thể thay đổi và Google có kênh xin nâng —
kiểm lại nguồn.

## Sự thật đắt giá nhất: TTL cache thay đổi theo cách bạn trả tiền

Đây là chi tiết mà gần như không ai biết, và nó nằm đúng vào nguyên nhân 1.4.

Trên Claude Code, **thời gian sống của cache phụ thuộc vào việc bạn trả tiền
kiểu gì**:

| Cách xác thực | TTL cache |
| --- | --- |
| Gói thuê bao (Pro/Max/Team/Enterprise) | **1 giờ** |
| Gói thuê bao nhưng đang dùng usage credit | **5 phút** |
| API key hoặc cloud provider | **5 phút** (mặc định) |

Hệ quả rất cụ thể: nếu bạn đang ở gói thuê bao, nghỉ ăn trưa 45 phút rồi quay
lại phiên cũ, cache vẫn còn ấm. Cũng thao tác đó trên API key thì bạn trả
giá đầy đủ để nạp lại toàn bộ context.

Nghĩa là lời khuyên "đừng để phiên nguội quá lâu" trong
[`WORKFLOW.md`](WORKFLOW.md) **cấp thiết gấp mười hai lần** khi bạn dùng API
key. Và nó cũng có nghĩa: khoảnh khắc bạn vượt hạn mức gói và bắt đầu tiêu
usage credit, TTL của bạn rơi từ 1 giờ xuống 5 phút — bạn vừa bị chuyển sang
chế độ đắt hơn đúng lúc đang tiêu tiền thật.

## Trên gói thuê bao, con số đô la trên màn hình không phải hóa đơn của bạn

Claude Code hiển thị một con số chi phí cho mỗi phiên. Tài liệu chính thức
nói thẳng rằng khối Session trong `/usage` "dành cho người dùng API", và với
người đăng ký Max hay Pro thì "con số chi phí phiên không liên quan tới việc
tính tiền".

Con số đó được tính **cục bộ** từ số token nhân với giá niêm yết. Nó không
biết bạn đang ở gói nào, không phản ánh giá khuyến mãi hay chiết khấu hợp
đồng.

Vậy nên nếu bạn ở gói thuê bao, hãy dùng nó như một **chỉ số tương đối** —
tốt để so sánh A với B trong [`MEASURE.md`](MEASURE.md), vô nghĩa nếu coi là
số tiền. Thứ thực sự phản ánh tình trạng của bạn nằm ở cùng màn hình đó:

- **Thanh hạn mức gói** — bạn còn lại bao nhiêu.
- **Phân bổ (attribution)** — phần trăm mức dùng gần đây thuộc về từng skill,
  subagent, plugin và từng MCP server. Đây chính là công cụ kiểm toán cho
  [`solutions/mcp-server-audit.md`](solutions/mcp-server-audit.md).
- **Cờ hành vi (behavior flags)** — Claude Code tự gắn cờ những hành vi chiếm
  từ 10% mức dùng gần đây trở lên, ví dụ "long context" hay "cache misses".
  Đó là `CAUSE.md` được chẩn đoán tự động, ngay trên máy bạn.

Nhấn `d` hoặc `w` để xem 24 giờ hoặc 7 ngày. (`/cost` là bí danh của
`/usage`.) Số liệu tính từ lịch sử phiên trên **máy này**, nên không gồm mức
dùng từ thiết bị khác.

## Cửa sổ hạn mức: cái gì reset, và khi nào

| Thiết lập | Cửa sổ |
| --- | --- |
| Claude Pro / Max | Hạn mức phiên + hạn mức tuần |
| Claude Team / Enterprise | Hạn mức ghế, reset theo cửa sổ trượt 5 giờ **và** theo tuần |
| Codex trên gói ChatGPT | Cửa sổ 5 giờ dùng chung cho message cục bộ và cloud chat, có thể kèm hạn mức tuần |

Hai điều dễ khiến người ta mất thời gian:

1. **Đổi model không cứu được bạn.** Trên Claude, các cửa sổ hạn mức dùng
   chung cho mọi model. Gặp "You've hit your weekly limit" thì `/model` không
   mở lại được. Nhưng nếu thông báo là hạn mức riêng của Opus thì đổi sang
   model thấp hơn *có* giúp bạn làm tiếp.
2. **Hạn mức dùng chung với sản phẩm khác.** Trên Claude Team/Enterprise, hạn
   ngạch ghế chia chung với Claude chat và Cowork. Trên ChatGPT, Codex chia
   chung hạn mức với các bề mặt ChatGPT khác. Buổi sáng chat nhiều có thể
   khiến buổi chiều code bị chặn.

Cả hai đều củng cố cùng một điểm trong [`WORKFLOW.md`](WORKFLOW.md): một
phiên bị bỏ mở cả ngày vẫn rút hạn mức mỗi lần bạn gõ, vì toàn bộ lịch sử
được gửi lại theo từng request.

## Cảnh báo ngược: đừng mua công cụ để tiết kiệm khoản tiền bạn không tiêu

Đây là cái bẫy mà kho này có thể vô tình đẩy bạn vào.

Nếu bạn ở gói thuê bao cố định và **chưa bao giờ chạm trần hạn mức**, thì
giá trị tiền mặt của mọi thứ trong kho này bằng **không**. Bạn vẫn có thể
muốn áp dụng — phiên gọn thì agent làm tốt hơn, và
[`PROOF.md`](PROOF.md) cho thấy ponytail cải thiện chất lượng đầu ra chứ
không chỉ chi phí — nhưng hãy gọi đúng tên lý do. Đừng bỏ một buổi chiều cài
CodeGraph để "tiết kiệm 60% chi phí" khi chi phí của bạn là 200 đô cố định
dù có làm gì đi nữa.

Thứ tự đúng để quyết định:

1. **Bạn có chạm trần không?** Nếu không, dừng lại. Áp dụng
   [`WORKFLOW.md`](WORKFLOW.md) vì lý do chất lượng, bỏ qua phần còn lại.
2. **Có chạm trần?** Vậy đơn vị của bạn là dư địa. Ưu tiên theo cột "gói thuê
   bao" trong bảng ở trên — caching và tích lũy context, không phải nén output.
3. **Trả theo token?** Toàn bộ kho này áp dụng nguyên vẹn, theo thứ tự trong
   [`solutions/README.md`](solutions/README.md).
4. **Trả theo request?** Chỉ tối ưu số lượt gọi. Bỏ qua phần còn lại.

## Cách xác định bạn đang ở chế độ nào

| Agent | Kiểm tra bằng |
| --- | --- |
| Claude Code | `/usage` — có thanh hạn mức gói nghĩa là thuê bao; chỉ có khối Session nghĩa là API |
| Claude Code | `/login` cho biết bạn xác thực bằng tài khoản claude.ai hay API key |
| Codex CLI | `/status` — hiển thị gói và cửa sổ hạn mức đang áp dụng |
| Gemini CLI | Kiểm tra bạn đăng nhập bằng tài khoản Google (Code Assist, tính theo request) hay `GEMINI_API_KEY` (tính theo token) |
| Cline | Luôn tính theo token; xem provider đang chọn trong phần cài đặt |

Nếu tổ chức của bạn trộn nhiều cách đăng nhập, mỗi lập trình viên được tính
theo đúng cách họ đã xác thực — nên hai người ngồi cạnh nhau có thể đang tiêu
hai đơn vị hoàn toàn khác nhau.

## Điểm tham chiếu ngân sách

Cho đội đang trả theo token, tài liệu Claude Code công bố các con số triển
khai thực tế sau: trung bình khoảng **13 đô/lập trình viên/ngày có hoạt
động**, **150–250 đô/lập trình viên/tháng**, và dưới **30 đô/ngày hoạt động**
với 90% người dùng.

Dùng chúng làm mốc kiểm tra độ hợp lý, không phải mục tiêu. Nếu bạn đang ở
mức 400 đô/người/tháng thì có gì đó sai về cấu trúc — nhiều khả năng là phiên
không bao giờ được xóa hoặc model cao cấp để mặc định, đúng hai thủ phạm mà
tài liệu nhà cung cấp cũng nêu tên.

Vài khoản nhỏ hay bị bỏ sót:

- **Chi phí nền.** Claude Code tiêu một ít token cả khi bạn không gõ gì
  (tóm tắt hội thoại phục vụ `--resume`, xử lý lệnh) — thường dưới 0,04 đô
  mỗi phiên.
- **Đội agent.** Chạy nhiều instance song song tiêu token gấp khoảng **7 lần**
  một phiên thường khi các thành viên chạy ở chế độ plan. Đây là nguyên nhân
  6.1 và 6.3 gộp lại, ở quy mô lớn nhất mà bạn có thể tự tay tạo ra.
- **Tác vụ hẹn giờ.** Một tác vụ theo lịch vẫn kích hoạt dù phiên đang rảnh,
  và mỗi lần đều gửi lại toàn bộ context.

## Nguồn

- Claude Code — quản lý chi phí, `/usage`, TTL cache theo cách xác thực, cửa
  sổ hạn mức, chi phí nền, chi phí đội agent:
  <https://code.claude.com/docs/en/costs>
- Claude Code — tham chiếu lệnh (`/cost` là bí danh của `/usage`):
  <https://code.claude.com/docs/en/commands>
- Codex — giá và hạn mức theo gói ChatGPT, cửa sổ 5 giờ, cơ chế credit:
  <https://learn.chatgpt.com/docs/pricing>
- Gemini Code Assist — hạn ngạch request mỗi ngày và mỗi giây:
  <https://docs.cloud.google.com/gemini/docs/quotas>
- Cline — mô hình BYOK, không phí thuê bao ở bản mã nguồn mở:
  <https://cline.bot/pricing>

Giá cả và hạn mức thay đổi thường xuyên. Mọi con số ở đây được ghi lại vào
**tháng 8 năm 2026**.

---

# Billing: what saving tokens actually buys you

Every percentage in this repo quietly assumes one thing: that you **pay per
token**. For most readers, that assumption is wrong.

If you run Claude Code on a Max plan, Codex on ChatGPT Plus, or Gemini CLI on
a Code Assist tier, then cutting 30% of your tokens does **not** cut your bill
by 30%. Your bill is a fixed monthly number. What you just bought is something
else: more headroom before you hit a wall, more sessions before the weekly
limit runs out.

This doc makes explicit which **unit** you are spending, because that unit
decides which causes in [`CAUSE.md`](CAUSE.md) are actually expensive for you
— and the answer differs by setup.

> **Evidence status:** ⚪ The plan, limit-window, and quota facts below are
> sourced from vendor documentation (see [Sources](#sources)). The
> interpretation — which causes hurt more under which unit — is reasoned from
> those facts and is **unmeasured**. Vendors change pricing and limits often;
> re-check the sources before you budget against them.

## Which unit are you spending?

| Setup | Scarce unit | What saved tokens buy |
| --- | --- | --- |
| Claude Code + API key / Bedrock / Vertex / Foundry | **Dollars**, per token | Real money, 1:1 |
| Claude Code + Pro / Max / Team / Enterprise plan | **Session & weekly limits** | Headroom: more work before you're blocked |
| Codex + API key | **Dollars**, per token | Real money, 1:1 |
| Codex + a ChatGPT plan (Go/Plus/Pro/Business) | **Credits inside a 5-hour window** | Headroom first, money only once you buy credits |
| Gemini CLI + Code Assist | **Requests per day** | Almost *nothing* — see below |
| Gemini CLI + paid API key | **Dollars**, per token | Real money, 1:1 |
| Cline (BYOK) | **Dollars**, per token | Real money, 1:1 |

Cline is the only one of the four that is metered by default: the open-source
build has no subscription or seat fee, and you pay your chosen provider's
exact price. On Cline, every percentage in this repo converts straight into
dollars.

## Three units, three different priority orders

This is the most important part of the doc. The same cause hurts completely
differently depending on the unit you spend.

| Cause | Metered (dollars) | Subscription (limits) | Request quota (Gemini) |
| --- | --- | --- | --- |
| 1.1/1.4 Broken caching | Very expensive — cache reads cost a fraction | Very expensive — cache misses get flagged in `/usage` | Negligible |
| 2.1 Unbounded history | Expensive, grows quadratically | Expensive — the number-one reason weekly limits evaporate | Negligible |
| 3.1 Oversized tool output | Expensive | Expensive | **Free** |
| 3.2 Chatty round-trips | Moderate | Moderate | **The most expensive thing you can do** |
| 3.3 Retry/polling loops | Moderate | Moderate | **Same** |
| 5.1 Reasoning tokens | Expensive — billed as output tokens | Expensive | Negligible |
| 5.2 Verbose output | Expensive | Expensive | **Free** |
| 6.2 Oversized model | Expensive — tier price gaps | Expensive — premium models drain limits faster | Negligible |

Read your column. Don't read across.

### The Gemini inversion: output is free, calls are not

Gemini Code Assist quota is counted in **requests**, not tokens: 1,500
requests/day per user on Standard, 2,000 on Enterprise, with a ceiling of 2
requests/second.

That inverts most of this guide. If your scarce unit is requests, then:

- Tightening `truncateToolOutputThreshold` saves you **no quota at all**.
- A rambling answer costs exactly what a terse one costs: one request.
- But an agent that makes 40 tool calls instead of 8 burns **five times** the
  quota.

On that tier, read [`solutions/tool-composition.md`](solutions/tool-composition.md)
and [`solutions/event-driven-waiting.md`](solutions/event-driven-waiting.md)
first, and skip the entire output-compression branch until you move to a paid
API key. Note these quotas can change and Google has a quota-increase
process — re-check the source.

## The most expensive fact here: cache TTL depends on how you pay

This is the detail almost nobody knows, and it lands squarely on cause 1.4.

On Claude Code, **cache lifetime depends on your billing mode**:

| How you authenticate | Cache TTL |
| --- | --- |
| Subscription (Pro/Max/Team/Enterprise) | **1 hour** |
| Subscription, but drawing on usage credits | **5 minutes** |
| API key or cloud provider | **5 minutes** (default) |

The consequence is concrete: on a subscription, you can take a 45-minute lunch,
come back to the same session, and the cache is still warm. Do the same on an
API key and you pay full price to reload your entire context.

Which means the "don't let a session go cold" advice in
[`WORKFLOW.md`](WORKFLOW.md) is **twelve times more urgent** on an API key.
And it means that the moment you exceed your plan limit and start drawing on
usage credits, your TTL drops from an hour to five minutes — you get switched
into the more expensive mode exactly when you start spending real money.

## On a subscription, the dollar figure on screen is not your bill

Claude Code shows a cost figure per session. The official documentation says
outright that the Session block in `/usage` is "intended for API users," and
that for Max and Pro subscribers "the session cost figure isn't relevant for
billing purposes."

That number is computed **locally** from token counts at list prices. It
doesn't know what plan you're on, and it doesn't reflect promotional pricing
or contracted discounts.

So on a subscription, treat it as a **relative** metric — good for comparing
A against B in [`MEASURE.md`](MEASURE.md), meaningless as an amount of money.
What actually reflects your situation is on the same screen:

- **Plan usage bars** — how much you have left.
- **Attribution** — the share of recent usage belonging to each skill,
  subagent, plugin, and individual MCP server. This is the audit instrument
  for [`solutions/mcp-server-audit.md`](solutions/mcp-server-audit.md).
- **Behavior flags** — Claude Code flags behaviors accounting for 10% or more
  of recent usage, such as "long context" or "cache misses." That is
  `CAUSE.md` diagnosed automatically, on your own machine.

Press `d` or `w` for 24 hours or 7 days. (`/cost` is an alias for `/usage`.)
The figures come from session history on **this machine**, so usage from other
devices isn't included.

## Limit windows: what resets, and when

| Setup | Window |
| --- | --- |
| Claude Pro / Max | A session limit plus a weekly limit |
| Claude Team / Enterprise | Seat allowance, resetting on a rolling 5-hour window **and** a weekly one |
| Codex on a ChatGPT plan | A 5-hour window shared between local messages and cloud chats, with weekly limits on top |

Two things reliably waste people's time:

1. **Switching models won't save you.** On Claude, the limit windows are
   shared across all models. If you hit "You've hit your weekly limit,"
   `/model` will not restore access. But if the message is an Opus-specific
   limit, dropping to a lower model *does* let you keep working.
2. **Limits are shared with other products.** On Claude Team/Enterprise, the
   seat allowance is shared with Claude chat and Cowork. On ChatGPT, Codex
   shares limits with other ChatGPT surfaces. A chat-heavy morning can block
   your afternoon of coding.

Both reinforce the same point from [`WORKFLOW.md`](WORKFLOW.md): a session
left open all day still draws on your limits every time you type, because the
whole history is re-sent with each request.

## The inverse warning: don't buy tools to save money you aren't spending

This is the trap this repo could walk you into.

If you're on a flat subscription and you **never hit your limits**, then the
cash value of everything in this repo is **zero**. You may still want to apply
it — a leaner session produces better agent output, and [`PROOF.md`](PROOF.md)
shows ponytail improving output quality rather than just cost — but name the
reason correctly. Don't spend an afternoon installing CodeGraph to "save 60%
on cost" when your cost is a fixed $200 no matter what you do.

The right order to decide:

1. **Do you hit your limits?** If not, stop. Apply [`WORKFLOW.md`](WORKFLOW.md)
   for quality reasons and skip the rest.
2. **You do hit them?** Then your unit is headroom. Prioritize by the
   "subscription" column above — caching and context accumulation, not output
   compression.
3. **Metered?** The whole repo applies as written, in the order given in
   [`solutions/README.md`](solutions/README.md).
4. **Request-quota?** Optimize call count only. Ignore the rest.

## How to tell which mode you're in

| Agent | Check with |
| --- | --- |
| Claude Code | `/usage` — plan usage bars mean subscription; a Session block alone means API |
| Claude Code | `/login` shows whether you authenticated with a claude.ai account or an API key |
| Codex CLI | `/status` — shows your plan and the limit window in force |
| Gemini CLI | Check whether you signed in with a Google account (Code Assist, request-metered) or `GEMINI_API_KEY` (token-metered) |
| Cline | Always token-metered; check the selected provider in settings |

If your organization mixes sign-in methods, each developer is metered
according to the one they authenticated with — so two people sitting next to
each other can be spending entirely different units.

## Budget reference points

For teams paying per token, the Claude Code documentation publishes these
real-deployment figures: roughly **$13 per developer per active day**,
**$150–250 per developer per month**, and under **$30 per active day** for 90%
of users.

Use them as a sanity check, not a target. If you're at $400/developer/month,
something is structurally wrong — most likely sessions that are never cleared
or a premium model left as the default, which are the same two culprits the
vendor documentation names.

A few line items people miss:

- **Background spend.** Claude Code uses some tokens even when you aren't
  typing (conversation summarization for `--resume`, command processing) —
  typically under $0.04 per session.
- **Agent teams.** Running multiple instances in parallel uses roughly **7x**
  the tokens of a standard session when teammates run in plan mode. That's
  causes 6.1 and 6.3 combined, at the largest scale you can create by hand.
- **Scheduled tasks.** A scheduled task fires on its interval even while the
  session is idle, sending your full context each time.

## Sources

- Claude Code — cost management, `/usage`, cache TTL by authentication mode,
  limit windows, background spend, agent-team cost:
  <https://code.claude.com/docs/en/costs>
- Claude Code — commands reference (`/cost` is an alias for `/usage`):
  <https://code.claude.com/docs/en/commands>
- Codex — pricing and limits per ChatGPT plan, the 5-hour window, how credits
  work: <https://learn.chatgpt.com/docs/pricing>
- Gemini Code Assist — per-day and per-second request quotas:
  <https://docs.cloud.google.com/gemini/docs/quotas>
- Cline — BYOK model, no subscription fee on the open-source build:
  <https://cline.bot/pricing>

Pricing and limits change often. Every figure here was recorded in
**August 2026**.
