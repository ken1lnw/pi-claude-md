# 🌍 External Skills — ติดตั้งผ่าน Marketplace

Skills ชุดนี้ **ไม่ได้เก็บไฟล์ไว้ใน repo นี้** เพราะเป็นผลงานของทีมอื่น (โอเพนซอร์ส MIT)
เราเก็บแค่ **แค็ตตาล็อก + คำสั่งติดตั้ง** ซึ่งดีกว่าการ copy ไฟล์เข้ามา:

| copy ไฟล์เข้า repo | ใช้ marketplace (วิธีนี้) |
|---|---|
| ต้อง `git pull` เองถึงจะได้ของใหม่ | `/plugin marketplace update` ได้ของใหม่ทันที |
| repo บวมขึ้น ~700KB | repo เท่าเดิม |
| ต้องแนบ LICENSE + attribution เอง | license อยู่ที่ต้นทาง ไม่ต้องดูแล |
| แก้ไฟล์แล้วชนกับ upstream | ไม่มีปัญหา |

---

## ⚡ ติดตั้ง (ครั้งเดียว)

พิมพ์ใน Claude Code:

```
/plugin marketplace add Aaronontheweb/dotnet-skills
```

แล้วเลือกติดตั้ง plugin ที่ต้องการ:

```
/plugin install dotnet-skills
```

อัปเดตของใหม่ในภายหลัง:

```
/plugin marketplace update
```

> **ที่มา:** [`Aaronontheweb/dotnet-skills`](https://github.com/Aaronontheweb/dotnet-skills) — *"Claude Code skills and sub-agents for .NET Developers"* — **License: MIT**

---

## 📦 มีอะไรบ้าง (35 skills)

### C# — ภาษาและการออกแบบ

| Skill | ใช้ทำอะไร |
|---|---|
| `csharp-coding-standards` | เขียน C# สมัยใหม่ — records, pattern matching, `Span<T>`, C# 12+ |
| `csharp-api-design` | ออกแบบ public API ให้ compatible ระยะยาว (extend-only, versioning, NuGet) |
| `csharp-concurrency-patterns` | เลือกให้ถูก — `async/await` สำหรับ I/O, Channels สำหรับ producer/consumer, Akka สำหรับ state |
| `csharp-type-design-performance` | `sealed` class, `readonly struct`, static pure function, เลือก collection ให้เร็ว |
| `project-structure` | `.slnx`, `Directory.Build.props`, Central Package Management, SourceLink, `global.json` |
| `package-management` | จัดการ NuGet ด้วย CPM + `dotnet` CLI — ห้ามแก้ XML ตรง ๆ |
| `local-tools` | `dotnet-tools.json` ให้ tooling ตรงกันทั้งทีมและ CI |
| `serialization` | เลือก format ให้ถูก — Protobuf/MessagePack > Newtonsoft; System.Text.Json + AOT |

### Akka.NET

| Skill | ใช้ทำอะไร |
|---|---|
| `akka-best-practices` | EventStream vs DistributedPubSub, supervision vs try-catch, Props |
| `akka-hosting-actor-patterns` | entity actor, cluster sharding, message extractor, reminders |
| `akka-aspire-configuration` | ต่อ Akka.NET เข้ากับ .NET Aspire (clustering, persistence, management) |
| `akka-management` | cluster bootstrap, service discovery (K8s/Azure), health check |
| `akka-testing-patterns` | เทสต์ actor ด้วย Akka.Hosting.TestKit, TestProbe, persistence |

### .NET Aspire

| Skill | ใช้ทำอะไร |
|---|---|
| `aspire-configuration` | ให้ AppHost ส่ง config ผ่าน env var — โค้ดแอปไม่ต้องรู้จัก Aspire |
| `aspire-service-defaults` | ServiceDefaults ร่วม — OpenTelemetry, health check, resilience |
| `aspire-integration-testing` | integration test ด้วย Aspire + xUnit (ของจริง ไม่ mock) |
| `aspire-mailpit-integration` | เทสต์อีเมลด้วย Mailpit — ดู HTML ที่ render จริงโดยไม่ส่งออกไปข้างนอก |

### Database / Performance

| Skill | ใช้ทำอะไร |
|---|---|
| `database-performance` | แยก read/write model, กัน N+1, `AsNoTracking`, จำกัด row, ห้าม join ฝั่งแอป |
| `efcore-patterns` | EF Core — NoTracking เป็น default, query splitting, จัดการ migration |

### Test และคุณภาพโค้ด

| Skill | ใช้ทำอะไร |
|---|---|
| `testcontainers` | integration test กับ DB/queue/cache **ตัวจริงใน Docker** แทน mock |
| `snapshot-testing` | Verify — approve API surface, HTTP response, email ที่ render แล้ว |
| `verify-email-snapshots` | snapshot test email template โดยเฉพาะ (คู่กับ MJML) |
| `crap-analysis` | หา code เสี่ยง จาก CRAP score (ซับซ้อนสูง × ไม่มีเทสต์) |
| `slopwatch` | **จับ LLM reward hacking** — ปิดเทสต์ทิ้ง, `catch {}` เปล่า, suppress warning |
| `playwright-blazor` | UI test แอป Blazor (Server/WASM) ด้วย Playwright |
| `playwright-ci-caching` | cache Playwright browser ใน CI — ประหยัด 1–2 นาทีทุก build |

### DI / Config / Observability

| Skill | ใช้ทำอะไร |
|---|---|
| `microsoft-extensions-dependency-injection` | จัด DI เป็น `Add*` extension method — `Program.cs` สะอาด, reuse ใน test ได้ |
| `microsoft-extensions-configuration` | Options pattern — `IValidateOptions`, strongly-typed settings, validate ตอน startup |
| `opentelementry-dotnet-instrumentation` | ใส่ OpenTelemetry — tracing (Activity/Span), metrics, naming convention |

### เครื่องมือและอื่น ๆ

| Skill | ใช้ทำอะไร |
|---|---|
| `ilspy-decompile` | decompile .NET assembly ดู implementation จริงของ framework/NuGet |
| `dotnet-devcert-trust` | แก้ปัญหา HTTPS dev certificate บน Linux/WSL2 |
| `mjml-email-templates` | เขียน email template ด้วย MJML → HTML ที่ใช้ได้ทุก client (Outlook/Gmail) |
| `r3-reactive-extensions` | R3 — Reactive Extensions รุ่นใหม่สำหรับ .NET |
| `marketplace-publishing` | วิธี publish skill/agent เข้า marketplace ตัวนี้ |
| `skills-index-snippets` | ทำ index ใน `AGENTS.md`/`CLAUDE.md` ให้ route งานไปหา skill ที่ถูกตัว |

---

## ⚠️ Skills ที่ยัง **ไม่แนะนำให้แชร์** (ที่มายังไม่ยืนยัน)

5 ตัวนี้เจอกระจายอยู่ในหลาย community repo และ **ยังยืนยัน license ต้นทางไม่ได้**
ใครอยากใช้ให้ไปหาต้นทางเองก่อน อย่าเพิ่งเอาเข้า repo นี้:

| Skill | ทำอะไร | สถานะที่มา |
|---|---|---|
| `web-design-guidelines` | ตรวจ UI ตาม Web Interface Guidelines + accessibility | frontmatter ระบุ `author: vercel` แต่หา repo ต้นทางไม่เจอ |
| `fastapi-clean-architecture` | FastAPI + Clean Architecture (Domain/Infra/API, repository pattern) | เจอในหลาย community repo — ไม่มี canonical upstream |
| `async-testing-expert` | pytest สำหรับ async Python (mocking, fixtures) | เดียวกัน |
| `multi-system-sso-authentication` | SSO หลาย IdP, JWT RS256, session verification | เดียวกัน |
| `brazilian-financial-integration` | Boleto / PIX / CPF-CNPJ (ระบบการเงินบราซิล) | เดียวกัน — และไม่เกี่ยวกับงานทีมเรา |

---

## 📌 หมายเหตุสำหรับคนที่จะเพิ่มของใหม่เข้าหน้านี้

ก่อนเพิ่ม skill จากภายนอกเข้าแค็ตตาล็อก ต้องตอบให้ได้ 3 ข้อ:

1. **ต้นทางคือ repo ไหน** — ต้องมี URL
2. **License อะไร** — ถ้าไม่มี LICENSE ไฟล์ที่ต้นทาง = อย่าเอาเข้า
3. **ติดตั้งยังไง** — marketplace ดีที่สุด; ถ้าไม่มี marketplace ให้ลิงก์ไป repo ต้นทาง **อย่า copy ไฟล์เข้ามา**

repo นี้เป็น **public** — การ copy โค้ดคนอื่นเข้ามาโดยไม่มี license คือการ redistribute ที่ไม่ถูกต้อง
