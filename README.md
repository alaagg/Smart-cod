import React, { useEffect, useMemo, useRef, useState } from "react";

// 🚸 حضانة AI عربية — نموذج أولي تفاعلي (MVP) // - تعليم مبسّط: الحروف / الأرقام / الألوان // - نطق عربي عبر Web Speech API // - تعرّف صوتي (إن توفر في المتصفح) // - تتبّع تقدّم الطفل محليًا عبر localStorage // ⓘ أفضل تجربة على Chrome/Edge بتمكين العربية في إعدادات الأصوات.

// ---------- بيانات المحتوى ---------- const LETTERS: { char: string; word: string; emoji: string }[] = [ { char: "ا", word: "أسد", emoji: "🦁" }, { char: "ب", word: "بطة", emoji: "🦆" }, { char: "ت", word: "تفاحة", emoji: "🍎" }, { char: "ث", word: "ثعلب", emoji: "🦊" }, { char: "ج", word: "جمل", emoji: "🐪" }, { char: "ح", word: "حصان", emoji: "🐴" }, { char: "خ", word: "خبز", emoji: "🍞" }, { char: "د", word: "دب", emoji: "🐻" }, { char: "ذ", word: "ذُرة", emoji: "🌽" }, { char: "ر", word: "رمان", emoji: "🍎" }, { char: "ز", word: "زرافة", emoji: "🦒" }, { char: "س", word: "سمكة", emoji: "🐟" }, { char: "ش", word: "شمس", emoji: "☀️" }, { char: "ص", word: "صقر", emoji: "🦅" }, { char: "ض", word: "ضفدع", emoji: "🐸" }, { char: "ط", word: "طائرة", emoji: "✈️" }, { char: "ظ", word: "ظرف", emoji: "✉️" }, { char: "ع", word: "عصفور", emoji: "🐦" }, { char: "غ", word: "غابة", emoji: "🌳" }, { char: "ف", word: "فراولة", emoji: "🍓" }, { char: "ق", word: "قلم", emoji: "✏️" }, { char: "ك", word: "كتاب", emoji: "📖" }, { char: "ل", word: "ليمون", emoji: "🍋" }, { char: "م", word: "موز", emoji: "🍌" }, { char: "ن", word: "نحلة", emoji: "🐝" }, { char: "ه", word: "هدهد", emoji: "🐦" }, { char: "و", word: "وردة", emoji: "🌹" }, { char: "ي", word: "يد", emoji: "🖐️" }, ];

const NUMBERS: { n: number; word: string; emoji: string }[] = [ { n: 1, word: "واحد", emoji: "1️⃣" }, { n: 2, word: "اثنان", emoji: "2️⃣" }, { n: 3, word: "ثلاثة", emoji: "3️⃣" }, { n: 4, word: "أربعة", emoji: "4️⃣" }, { n: 5, word: "خمسة", emoji: "5️⃣" }, { n: 6, word: "ستة", emoji: "6️⃣" }, { n: 7, word: "سبعة", emoji: "7️⃣" }, { n: 8, word: "ثمانية", emoji: "8️⃣" }, { n: 9, word: "تسعة", emoji: "9️⃣" }, { n: 10, word: "عشرة", emoji: "🔟" }, ];

const COLORS: { name: string; emoji: string }[] = [ { name: "أحمر", emoji: "🟥" }, { name: "أزرق", emoji: "🟦" }, { name: "أخضر", emoji: "🟩" }, { name: "أصفر", emoji: "🟨" }, { name: "بنّي", emoji: "🟫" }, { name: "أسود", emoji: "⬛" }, { name: "أبيض", emoji: "⬜" }, { name: "برتقالي", emoji: "🟧" }, { name: "زهري", emoji: "🌸" }, { name: "بنفسجي", emoji: "🔮" }, ];

// ---------- أدوات الصوت ---------- function useArabicVoices() { const [voices, setVoices] = useState<SpeechSynthesisVoice[]>([]);

useEffect(() => { const load = () => { const list = window.speechSynthesis.getVoices(); const ar = list.filter((v) => (v.lang || "").toLowerCase().startsWith("ar")); setVoices(ar.length ? ar : list); }; load(); window.speechSynthesis.onvoiceschanged = load; return () => { window.speechSynthesis.onvoiceschanged = null as any; }; }, []);

return voices; }

function speak(text: string, voice?: SpeechSynthesisVoice) { try { const u = new SpeechSynthesisUtterance(text); if (voice) u.voice = voice; u.lang = voice?.lang || "ar-SA"; u.rate = 0.95; window.speechSynthesis.cancel(); window.speechSynthesis.speak(u); } catch (e) { console.warn("Speech failed", e); } }

// تعرّف صوتي مبسّط (تجريبي) function useSpeechRecognition(lang = "ar-SA") { const [supported, setSupported] = useState(false); const recRef = useRef<any>(null);

useEffect(() => { const SR = (window as any).webkitSpeechRecognition || (window as any).SpeechRecognition; if (SR) { setSupported(true); const rec = new SR(); rec.lang = lang; rec.interimResults = false; rec.maxAlternatives = 1; recRef.current = rec; } }, [lang]);

const listen = () => new Promise<string>((resolve, reject) => { if (!recRef.current) return reject(new Error("no_rec")); const rec = recRef.current; const onResult = (e: any) => { const t = e.results?.[0]?.[0]?.transcript || ""; cleanup(); resolve(String(t)); }; const onEnd = () => { rec.stop(); }; const onError = (e: any) => { cleanup(); reject(e); }; const cleanup = () => { rec.removeEventListener("result", onResult); rec.removeEventListener("end", onEnd); rec.removeEventListener("error", onError); }; rec.addEventListener("result", onResult); rec.addEventListener("end", onEnd); rec.addEventListener("error", onError); try { rec.start(); } catch {} });

return { supported, listen }; }

// ---------- تخزين تقدّم الطفل ---------- const KEY = "hodana-ai-progress"; function useProgress() { const [progress, setProgress] = useState<{ letters: number; numbers: number; colors: number }>(() => { try { return JSON.parse(localStorage.getItem(KEY) || "") || { letters: 0, numbers: 0, colors: 0 }; } catch { return { letters: 0, numbers: 0, colors: 0 }; } }); const save = (p: typeof progress) => { setProgress(p); localStorage.setItem(KEY, JSON.stringify(p)); }; const reset = () => save({ letters: 0, numbers: 0, colors: 0 }); return { progress, save, reset }; }

// ---------- الواجهة ---------- export default function HodanaAI() { const voices = useArabicVoices(); const [voiceIndex, setVoiceIndex] = useState(0); const { supported, listen } = useSpeechRecognition("ar-SA"); const { progress, save, reset } = useProgress();

type Module = "letters" | "numbers" | "colors"; const [module, setModule] = useState<Module>("letters"); const [mode, setMode] = useState<"learn" | "quiz">("learn"); const [question, setQuestion] = useState<any>(null); const [result, setResult] = useState<string | null>(null); const [busy, setBusy] = useState(false);

const items = useMemo(() => { if (module === "letters") return LETTERS; if (module === "numbers") return NUMBERS; return COLORS; }, [module]);

// بدء سؤال عشوائي في وضع الاختبار const startQuiz = () => { const r = items[Math.floor(Math.random() * items.length)]; setQuestion(r); setResult(null); // نطق السؤال بالعربية const text = module === "letters" ? ما هو هذا الحرف؟ ${r.char}. قل: ${r.char} : module === "numbers" ? ما هو هذا الرقم؟ ${r.word} : ما هو هذا اللون؟ ${r.name}; speak(text, voices[voiceIndex]); };

const handleListen = async () => { if (!supported) { setResult("التعرّف الصوتي غير مدعوم في هذا المتصفح 🙁"); return; } try { setBusy(true); const heard = (await listen()).trim(); let ok = false; if (module === "letters") { ok = heard.includes(question.char) || heard.includes(question.word); } else if (module === "numbers") { ok = heard.includes(question.word) || heard.includes(String(question.n)); } else { ok = heard.includes(question.name); } setResult(ok ? أحسنت! سمعت: ${heard} : قريب! أنت قلت: ${heard}); // تحديث التقدّم const p = { ...progress } as any; if (ok) p[module] = Math.min(100, p[module] + 5); save(p); } catch (e) { setResult("حاول مرة أخرى، لم أسمع بوضوح."); } finally { setBusy(false); } };

return ( <div dir="rtl" className="min-h-screen bg-gradient-to-b from-amber-50 to-orange-50 text-zinc-800"> <div className="max-w-5xl mx-auto p-4 sm:p-8"> {/* العنوان */} <header className="flex flex-col sm:flex-row items-start sm:items-center justify-between gap-4 mb-6"> <div> <h1 className="text-3xl sm:text-4xl font-extrabold tracking-tight">حضانة <span className="text-orange-600">AI</span> العربية</h1> <p className="text-sm sm:text-base text-zinc-600 mt-1">معلّم تفاعلي للأطفال (٢–٦ سنوات) — حروف، أرقام، ألوان، نطق، واختبارات صوتية</p> </div> <div className="flex items-center gap-2"> <select className="rounded-xl border px-3 py-2 bg-white shadow-sm" value={voiceIndex} onChange={(e) => setVoiceIndex(Number(e.target.value))} title="اختيار صوت عربي" > {voices.map((v, i) => ( <option key={v.name + i} value={i}> {v.name} ({v.lang}) </option> ))} </select> <button onClick={() => speak("مرحبًا! أنا معلّمك الذكي.", voices[voiceIndex])} className="rounded-xl px-4 py-2 bg-orange-600 text-white shadow hover:bg-orange-700" > تجربة الصوت </button> </div> </header>

{/* مؤشرات التقدم */}
    <section className="grid grid-cols-1 sm:grid-cols-3 gap-3 mb-6">
      {([
        { key: "letters", label: "الحروف", val: progress.letters },
        { key: "numbers", label: "الأرقام", val: progress.numbers },
        { key: "colors", label: "الألوان", val: progress.colors },
      ] as any[]).map((row) => (
        <div key={row.key} className="bg-white rounded-2xl p-4 shadow">
          <div className="flex items-center justify-between mb-2">
            <span className="font-semibold">{row.label}</span>
            <span className="text-sm text-zinc-500">{row.val}%</span>
          </div>
          <div className="w-full h-3 bg-zinc-100 rounded-full overflow-hidden">
            <div className="h-full bg-orange-500" style={{ width: `${row.val}%` }} />
          </div>
        </div>
      ))}
    </section>

    {/* اختيار الوحدة والوضع */}
    <section className="bg-white rounded-2xl p-4 sm:p-6 shadow mb-6">
      <div className="flex flex-wrap items-center gap-3 mb-4">
        <label className="font-semibold">الوحدة:</label>
        <div className="flex gap-2">
          {(
            [
              { id: "letters", name: "الحروف" },
              { id: "numbers", name: "الأرقام" },
              { id: "colors", name: "الألوان" },
            ] as { id: Module; name: string }[]
          ).map((m) => (
            <button
              key={m.id}
              onClick={() => setModule(m.id)}
              className={`px-4 py-2 rounded-xl border shadow-sm ${
                module === m.id ? "bg-orange-600 text-white" : "bg-zinc-50 hover:bg-zinc-100"
              }`}
            >
              {m.name}
            </button>
          ))}
        </div>

        <span className="mx-2" />

        <label className="font-semibold">الوضع:</label>
        <div className="flex gap-2">
          {(["learn", "quiz"] as const).map((m) => (
            <button
              key={m}
              onClick={() => setMode(m)}
              className={`px-4 py-2 rounded-xl border shadow-sm ${
                mode === m ? "bg-emerald-600 text-white" : "bg-zinc-50 hover:bg-zinc-100"
              }`}
            >
              {m === "learn" ? "تعلّم" : "اختبار"}
            </button>
          ))}
        </div>

        <div className="ms-auto flex gap-2">
          <button onClick={reset} className="px-3 py-2 rounded-xl border bg-zinc-50 hover:bg-zinc-100">
            تصفير التقدّم
          </button>
        </div>
      </div>

      {mode === "learn" ? (
        <LearnGrid module={module} voices={voices} voiceIndex={voiceIndex} />
      ) : (
        <QuizPanel
          module={module}
          question={question}
          onStart={startQuiz}
          onListen={handleListen}
          busy={busy}
          result={result}
        />
      )}
    </section>

    {/* تلميحات للأهل */}
    <section className="text-sm text-zinc-600 px-2">
      <ul className="list-disc ps-5 space-y-1">
        <li>يفضّل استخدام Chrome/Edge وإضافة صوت عربي من إعدادات النظام لتحسين النطق.</li>
        <li>إن لم يعمل التعرّف الصوتي، يمكن للطفل الضغط على البطاقات لسماع النطق والتعلّم بدون اختبار صوتي.</li>
        <li>لا يتم جمع أي بيانات خارج جهازكم — التقدّم محفوظ محليًا فقط.</li>
      </ul>
    </section>
  </div>
</div>

); }

function LearnGrid({ module, voices, voiceIndex, }: { module: "letters" | "numbers" | "colors"; voices: SpeechSynthesisVoice[]; voiceIndex: number; }) { const data: any[] = module === "letters" ? LETTERS : module === "numbers" ? NUMBERS : COLORS; return ( <div> <h2 className="font-bold mb-3">اختر بطاقة لسماع النطق والتعلّم</h2> <div className="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 gap-3"> {data.map((it, i) => ( <button key={i} onClick={() => { const text = module === "letters" ? ${it.emoji}، هذا حرف ${it.char}، مثل كلمة ${it.word} : module === "numbers" ? ${it.emoji}، هذا رقم ${it.word} : ${it.emoji}، هذا لون ${it.name}; speak(text, voices[voiceIndex]); }} className="bg-zinc-50 hover:bg-zinc-100 rounded-2xl p-4 text-center shadow border" > <div className="text-4xl mb-2">{it.emoji}</div> <div className="text-xl font-bold"> {module === "letters" ? it.char : module === "numbers" ? it.word : it.name} </div> {module === "letters" && ( <div className="text-sm text-zinc-500">مثال: {it.word}</div> )} </button> ))} </div> </div> ); }

function QuizPanel({ module, question, onStart, onListen, busy, result, }: { module: "letters" | "numbers" | "colors"; question: any; onStart: () => void; onListen: () => void; busy: boolean; result: string | null; }) { return ( <div> <h2 className="font-bold mb-3">اختبار صوتي ممتع</h2> <div className="flex flex-wrap items-center gap-3 mb-4"> <button onClick={onStart} className="px-4 py-2 rounded-xl bg-indigo-600 text-white shadow hover:bg-indigo-700"> ابدأ سؤالًا عشوائيًا </button> <button onClick={onListen} disabled={!question || busy} className={px-4 py-2 rounded-xl shadow border ${ !question || busy ? "bg-zinc-200 text-zinc-500" : "bg-emerald-50 hover:bg-emerald-100" }} > 🎤 قل الإجابة </button> </div>

{question ? (
    <div className="bg-zinc-50 rounded-2xl p-4 border shadow-sm">
      <div className="text-6xl mb-3 text-center">
        {module === "letters" ? question.char : module === "numbers" ? question.emoji : question.emoji}
      </div>
      <p className="text-center text-zinc-600 mb-2">
        {module === "letters"
          ? `قل: ${question.char} — مثال: ${question.word}`
          : module === "numbers"
          ? `قل: ${question.word}`
          : `قل: ${question.name}`}
      </p>
      {result && (
        <div className="text-center font-semibold mt-2">
          {result.includes("أحسنت") ? (
            <span className="text-emerald-700">{result} ✅</span>
          ) : (
            <span className="text-amber-700">{result} 🙂</span>
          )}
        </div>
      )}
    </div>
  ) : (
    <p className="text-zinc-600">اضغط «ابدأ سؤالًا عشوائيًا» ثم «قل الإجابة» للتجربة.</p>
  )}
</div>

); }

