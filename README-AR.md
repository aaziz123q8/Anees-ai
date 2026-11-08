# مساعد أنيس (OpenClaw) — دليل التشغيل بالعربي

> المصدر الرسمي فقط: https://github.com/openclaw/openclaw — الوثائق: https://docs.openclaw.ai
> الإصدار الذي جُرّب: `2026.11.1`

**مهم:** البناء جُرّب في حاوية سحابية مؤقتة وانتهى بنجاح، لكن **التشغيل الفعلي يجب أن يكون على جهازك أو على VPS خاص بك**. لا تُدخل مفتاحك ولا تمسح QR في أي بيئة مؤقتة.

---

## ١) المتطلبات (على جهازك)
- **Node.js**: الإصدار `>=24.16 و <25` أو `>=26.1` (الوثائق توصي بـ 24.22.1 أو أحدث).
- **pnpm 12.15.0**: يُفعَّل عبر Corepack:
  ```bash
  corepack enable
  ```
- **git**.
- (للمتصفح) Chrome أو Chromium.

## ٢) التثبيت من المصدر (الترتيب الرسمي)
```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
corepack enable
pnpm install && pnpm build && pnpm ui:build
```
> المستودع كبير جداً بتاريخه الكامل. إن انقطع التنزيل استعمل `git clone --depth 1 ...`.

للتشغيل من داخل المجلد استعمل `pnpm openclaw ...` بدل `openclaw ...`.

## ٣) الإعداد الأولي
```bash
pnpm openclaw onboard --install-daemon
```
- اختر **Quick start**.
- عند سؤال المزوّد اختر **Anthropic API key** والصق مفتاحك **في المعالج مباشرة**. هو يحفظه في مكانه الصحيح.
- إن لم يتوفر systemd ستُتخطى خطوة الخدمة الخلفية، فشغّل البوابة يدوياً (القسم ٦).

### أين المفاتيح؟
| الشيء | المكان | يُرفع إلى Git؟ |
|---|---|---|
| إعدادات OpenClaw | `~/.openclaw/openclaw.json` | لا |
| متغيرات البيئة (اختياري) | `~/.openclaw/.env` | لا |
| مفتاح Anthropic | يحفظه المعالج (`onboard`) | لا |
| جلسة واتساب | `~/.openclaw/credentials/whatsapp/<الحساب>/creds.json` | **أبداً** |
| رمز دخول اللوحة | `pnpm openclaw gateway auth-token` | لا |

كل هذه خارج مجلد المشروع في مجلدك الشخصي، وملف `.gitignore` في هذا المستودع يمنع رفع `.env` و`.openclaw/` و`creds.json`.

## ٤) إعدادات الأمان الجاهزة
انسخ القالب `config/openclaw.template.json5` إلى `~/.openclaw/openclaw.json` (أو ادمج محتواه)، ثم تحقق:
```bash
pnpm openclaw config validate
```
ما يضبطه القالب:
- البوابة على `loopback` فقط (من جهازك فقط) مع رمز دخول.
- النموذج الافتراضي: `anthropic/claude-sonnet-5-5`.
- واتساب: `dmPolicy: "allowlist"` و`allowFrom` فارغة (ضع رقمك فقط) والمجموعات معطّلة. **لا تستعمل `open` أبداً.**
- طلبات الموافقة على الأوامر الخطرة تُرسل إلى محادثتك.
- أدوات ممنوعة: `gateway` و`cron` و`sessions_spawn` و`sessions_send`.

### سياسة الأوامر (مهم)
الوثائق تقول إن الافتراضي قد يسمح بتنفيذ الأوامر بلا سؤال، لذلك فعّل الوضع الحذر:
```bash
pnpm openclaw exec-policy preset cautious
```
يعني: قائمة سماح، ويسأل عن أي أمر غير مدرج، والرفض هو الافتراضي عند عدم الرد.
للرد من واتساب على طلب الموافقة: `/approve <id> allow-once` أو `/approve <id> deny`.

## ٥) المرحلة الأخيرة: التحقق
```bash
pnpm openclaw doctor
pnpm openclaw security audit
```

## ٦) التشغيل وإعادة التشغيل
```bash
pnpm openclaw gateway run        # تشغيل في المقدمة (يوقفه Ctrl+C)
pnpm openclaw gateway start      # تشغيل الخدمة الخلفية
pnpm openclaw gateway restart    # إعادة التشغيل
pnpm openclaw gateway status     # الحالة
pnpm openclaw dashboard          # فتح اللوحة http://127.0.0.1:18789/
```

## ٧) واتساب
1. **استعمل رقماً مخصصاً للمساعد** وليس رقمك الشخصي. الربط عبر واتساب ويب غير رسمي وفيه خطر الحظر.
2. ثبّت الإضافة إن طُلبت: `pnpm openclaw plugins install @openclaw/whatsapp`
3. اربط بالـQR:
   ```bash
   pnpm openclaw channels login --channel whatsapp
   ```
4. ضع **رقمك أنت فقط** في `channels.whatsapp.allowFrom` بالصيغة الدولية، مثل `["+965XXXXXXXX"]`.

## ٨) المتصفح
```bash
pnpm openclaw browser --browser-profile openclaw doctor
pnpm openclaw browser --browser-profile openclaw start
pnpm openclaw browser --browser-profile openclaw open https://example.com
pnpm openclaw browser --browser-profile openclaw snapshot
```
- يستعمل ملف Chromium مخصصاً (`openclaw`) منفصلاً عن متصفحك الشخصي. **لا تفعّل** ملف `user` (متصفحك المسجّل دخوله) إلا بقرار واعٍ.
- عند طلب موقع OTP أو CAPTCHA: الوثائق تقول إن المساعد يبلغ بأنه يحتاج تدخلاً يدوياً بدل التخمين.
- الإجراءات التي لا رجعة فيها (دفع، حذف، حجز) تمر عبر سياسة الموافقة في القسم ٤.

## ٩) المكالمات الهاتفية (Twilio)
```bash
pnpm openclaw plugins install @openclaw/voice-call
pnpm openclaw gateway restart
```
الإعداد تحت `plugins.entries.voice-call.config` (لا تضع `authToken` في ملف يُرفع إلى Git):
```json5
{
  provider: "twilio",
  fromNumber: "+رقم_Twilio",
  toNumber: "+رقمك",
  twilio: { accountSid: "AC...", authToken: "..." },
  inboundPolicy: "allowlist",   // الوارد: رقمك فقط
  allowFrom: ["+رقمك"],
  outbound: { defaultMode: "notify" },
  serve: { port: 3334, path: "/voice/webhook" },
}
```
اختبار (التجربة الافتراضية لا تتصل فعلياً):
```bash
pnpm openclaw voicecall setup
pnpm openclaw voicecall smoke --to "+رقمك"          # تجربة بدون اتصال
pnpm openclaw voicecall smoke --to "+رقمك" --yes    # اتصال حقيقي واحد (يكلّف مالاً)
```
**تنبيهات:**
- `allowFrom` هنا فحص ضعيف لرقم المتصل (يمكن تزييفه)، وليس إثبات هوية.
- **العربية في الصوت (TTS): غير مؤكدة** في الوثائق التي قرأتها. جرّب موديلات OpenAI أو ElevenLabs متعددة اللغات (مثل `eleven_multilingual_v2`) واختبر بمكالمة واحدة. لا تعتمد عليها قبل التجربة.
- لا يوجد مفتاح موثّق لـ«تأكيد قبل كل مكالمة صادرة» سوى `--yes` في الأمر. لذلك اجعل `tools.deny` يمنع الاتصال الصادر التلقائي، أو اطلب تأكيدك يدوياً.

### ماذا يحتاج الاتصال العلني بالإنترنت؟
Twilio يجب أن يصل إلى بوابتك عبر **عنوان عام (HTTPS)**، والوثائق تقول إن الإعداد يفشل مع العناوين المحلية. خياراتك (لا تفتح شيئاً إلا بقرارك):
| الخيار | المفتاح | الملاحظة |
|---|---|---|
| ngrok | `tunnel: { provider: "ngrok" }` | سهل للتجربة، والرابط المجاني يتغير فيكسر التوقيع |
| Tailscale Funnel | `tailscale: { mode: "funnel", port: 8443, path: "/voice/webhook" }` | المنافذ المسموحة 443/8443/10000، الأفضل للإنتاج |
| دومين ثابت | `publicUrl: "https://دومينك/voice/webhook"` | يحتاج شهادة HTTPS |

يفتح هذا **منفذ الـWebhook فقط**، وليس لوحة التحكم. تحقق دائماً من التوقيع ولا تفعّل `skipSignatureVerification`.

## ١٠) الإيقاف الطارئ (فوري)
من الأسرع إلى الأشمل:
1. **إيقاف البوابة:** `pnpm openclaw gateway stop` (أو `Ctrl+C` إن كانت في المقدمة).
2. **قطع واتساب:** `pnpm openclaw channels logout --channel whatsapp` أو من واتساب: الأجهزة المرتبطة ← تسجيل الخروج.
3. **منع تنفيذ الأوامر:** `pnpm openclaw exec-policy preset deny-all`
4. **سحب الصلاحيات من مصدرها:** احذف مفتاح Anthropic من console.anthropic.com، وعطّل رقم/مفتاح Twilio من لوحته.
5. **حذف الخدمة نهائياً:** `pnpm openclaw gateway uninstall`

## ١١) ما عن «heartbeat»
في هذا الإصدار **لم يعد هناك إعداد heartbeat بقيمة `"0m"`**؛ حلّت محلّه المهام المجدولة. بعد الإعداد الأولي اعرض المهام وعطّل مهمة الفحص الدوري:
```bash
pnpm openclaw automations list
pnpm openclaw automations disable <job-id>
```

## ١٢) قائمة الصلاحيات الممنوحة للمساعد
| الصلاحية | الحالة |
|---|---|
| الرد على رقمك فقط في واتساب | مفعّلة بعد إضافة الرقم |
| المجموعات في واتساب | معطّلة |
| الأوامر على الجهاز | بإذن (`cautious`)، يسأل عن غير المدرج |
| متصفح Chromium مخصص | عند التفعيل |
| متصفحك الشخصي (`user`) | **غير مفعّل** |
| إدارة البوابة والمهام والجلسات | ممنوعة (`tools.deny`) |
| الوصول من الشبكة/الإنترنت إلى اللوحة | **مغلق** (`loopback`) |
| مكالمات واردة | رقمك فقط، وبعد تشغيل الـWebhook العام |
| مكالمات صادرة | بتأكيدك |
