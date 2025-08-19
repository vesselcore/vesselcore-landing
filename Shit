import React, { useEffect, useMemo, useRef, useState } from "react";

/**

Impossible AI — Multi‑Model Chat (Frontend-Only Demo)


---

What this is:

A single-file React app that looks and feels like a modern AI website.


Lets users pick a model provider (DeepSeek, GPT, Claude, Gemini, Grok, Perplexity).


Collects user feedback (👍/👎) and opt‑in consent to "train" from chats.


Includes a Demo Mode that works fully client‑side (no API keys required).


Provides a clean hook (sendToProvider) where you can wire real provider APIs.


How to use this file:

Drop into any React project (Vite/Next/CRA). Export is a default React component.


TailwindCSS is assumed. (In ChatGPT canvas preview it's styled automatically.)


For production, build server routes that proxy to providers (see comments below).


Data/Privacy:

In Demo Mode, messages + feedback store in localStorage only.


In Live Mode, show a consent switch. Never send data server-side unless consented. */



// Basic message shape type Role = "user" | "assistant" | "system"; interface Msg { id: string; role: Role; text: string; liked?: boolean; disliked?: boolean; provider?: string; }

// Providers we'll surface in the UI const PROVIDERS = ["DeepSeek", "GPT", "Claude", "Gemini", "Grok", "Perplexity"] as const; export type Provider = typeof PROVIDERS[number];

// Simple utility const uid = () => Math.random().toString(36).slice(2, 10);

// Local "training" store (very light demo of learning-by-feedback). // We maintain a map from userPrompt → {bestResponse, score} // In Demo Mode we "improve" by replacing the canned response when a user thumbs‑up. const TRAINING_KEY = "impossibleai_training_v1"; function loadTraining() { try { return JSON.parse(localStorage.getItem(TRAINING_KEY) || "{}"); } catch { return {}; } } function saveTraining(data: any) { localStorage.setItem(TRAINING_KEY, JSON.stringify(data)); }

// Canned demo responses to simulate different providers (feel free to edit) const DEMO_RESPONSES: Record<Provider, string[]> = { DeepSeek: [ "Analyzing deeply… Here’s a concise plan with steps you can execute right now.", "Let's break the problem into 3 tactical chunks and ship a solution in 30 minutes.", ], GPT: [ "Here’s a friendly, thorough answer with clear steps and examples.", "I’ll give you the ‘why’, then the ‘how’, with a tiny checklist.", ], Claude: [ "Let’s reason this out carefully and consider safety, ethics, and clarity.", "I’ll summarize, critique, and propose an improved version for you.", ], Gemini: [ "Multimodal angle: imagine visuals, data, and code in one place.", "Here are quick options; pick one and I’ll expand it.", ], Grok: [ "Spicy take: here’s the brutally honest answer (with a wink).", "Cutting through noise—here’s the TL;DR and a plan.", ], Perplexity: [ "Answer with sources: key facts, citations, and next reads.", "Fast summary + links to dig deeper.", ], };

export default function ImpossibleAIWebsite() { const [provider, setProvider] = useState<Provider>("GPT"); const [demoMode, setDemoMode] = useState<boolean>(true); const [consent, setConsent] = useState<boolean>(false); const [input, setInput] = useState(""); const [messages, setMessages] = useState<Msg[]>([{ id: uid(), role: "system", text: "Welcome to Impossible AI — multi‑model chat. Pick a provider and ask anything.", }]); const [loading, setLoading] = useState(false); const [trainMap, setTrainMap] = useState<any>(loadTraining()); const listRef = useRef<HTMLDivElement>(null);

useEffect(() => { listRef.current?.scrollTo({ top: listRef.current.scrollHeight, behavior: "smooth" }); }, [messages]); useEffect(() => { saveTraining(trainMap); }, [trainMap]);

const canSend = input.trim().length > 0 && !loading;

async function onSend() { const text = input.trim(); if (!text) return; setInput(""); const userMsg: Msg = { id: uid(), role: "user", text }; setMessages(m => [...m, userMsg]); setLoading(true);

try {
  let reply = "";

  if (demoMode) {
    // Demo: simple deterministic "improvement" using feedback memory
    const learned = trainMap[text]?.bestResponse;
    if (learned) {
      reply = learned;
    } else {
      const pool = DEMO_RESPONSES[provider];
      reply = pool[(text.length + provider.length) % pool.length];
    }
    await new Promise(r => setTimeout(r, 400)); // tiny delay for realism
  } else {
    // LIVE MODE: Replace with real server call (see sendToProvider below)
    reply = await sendToProvider({ provider, prompt: text, consent });
  }

  const aiMsg: Msg = { id: uid(), role: "assistant", text: reply, provider };
  setMessages(m => [...m, aiMsg]);
} catch (e: any) {
  const err: Msg = { id: uid(), role: "assistant", text: `Error: ${e?.message || "Something went wrong."}` };
  setMessages(m => [...m, err]);
} finally {
  setLoading(false);
}

}

function onFeedback(msgId: string, type: "up" | "down") { setMessages(m => m.map(msg => { if (msg.id !== msgId) return msg; const updated = { ...msg, liked: type === "up", disliked: type === "down" }; // Light "training": if this was an assistant reply, store it as preferred for the last user prompt if (updated.role === "assistant") { const lastUser = [...m].reverse().find(x => x.role === "user"); if (lastUser && type === "up") { const prior = trainMap[lastUser.text] || { bestResponse: updated.text, score: 0 }; const improved = { bestResponse: updated.text, score: (prior.score || 0) + 1 }; const next = { ...trainMap, [lastUser.text]: improved }; setTrainMap(next); } } return updated; })); }

return ( <div className="min-h-screen w-full bg-slate-950 text-slate-100 flex flex-col"> {/* Header */} <header className="w-full border-b border-white/10 sticky top-0 backdrop-blur bg-slate-950/70 z-10"> <div className="max-w-5xl mx-auto px-4 py-4 flex items-center justify-between"> <div className="flex items-center gap-3"> <div className="h-9 w-9 rounded-full bg-gradient-to-tr from-cyan-400 to-yellow-300 shadow-lg shadow-yellow-400/20 animate-pulse" /> <div> <h1 className="text-lg font-semibold">Impossible AI</h1> <p className="text-xs text-slate-400">Multi‑Model Chat · Demo</p> </div> </div> <div className="flex items-center gap-3"> <select value={provider} onChange={e=>setProvider(e.target.value as Provider)} className="bg-slate-900 border border-white/10 rounded-xl px-3 py-2 text-sm"> {PROVIDERS.map(p => <option key={p} value={p}>{p}</option>)} </select> <label className="flex items-center gap-2 text-xs"> <input type="checkbox" checked={demoMode} onChange={e=>setDemoMode(e.target.checked)} /> Demo Mode </label> <label className="flex items-center gap-2 text-xs opacity-90"> <input type="checkbox" checked={consent} onChange={e=>setConsent(e.target.checked)} /> I consent to training from my chats (Live Mode only) </label> </div> </div> </header>

{/* Chat Area */}
  <main className="max-w-5xl mx-auto w-full grow px-4">
    <div ref={listRef} className="w-full h-[68vh] overflow-y-auto pt-6 pb-24 space-y-3">
      {messages.map(m => (
        <div key={m.id} className={`flex ${m.role === "user" ? "justify-end" : "justify-start"}`}>
          <div className={`max-w-[80%] rounded-2xl px-4 py-3 text-sm shadow-md ${m.role === "user" ? "bg-cyan-600/20 border border-cyan-400/20" : m.role === "system" ? "bg-slate-800/60 border border-white/10" : "bg-yellow-400/10 border border-yellow-300/20"}`}>
            {m.provider && (
              <div className="text-[10px] uppercase tracking-wide text-slate-400 mb-1">{m.provider}</div>
            )}
            <div className="whitespace-pre-wrap leading-relaxed">{m.text}</div>
            {m.role === "assistant" && (
              <div className="mt-2 flex items-center gap-2 text-xs text-slate-400">
                <button onClick={()=>onFeedback(m.id, "up")} className={`px-2 py-1 rounded-lg border border-white/10 ${m.liked ? "bg-green-400/20" : "hover:bg-white/5"}`}>👍</button>
                <button onClick={()=>onFeedback(m.id, "down")} className={`px-2 py-1 rounded-lg border border-white/10 ${m.disliked ? "bg-red-400/20" : "hover:bg-white/5"}`}>👎</button>
                <span className="ml-2">Feedback trains better replies over time.</span>
              </div>
            )}
          </div>
        </div>
      ))}
    </div>
  </main>

  {/* Composer */}
  <footer className="max-w-5xl mx-auto w-full px-4 pb-6 fixed bottom-0 left-0 right-0">
    <div className="bg-slate-900/80 backdrop-blur rounded-2xl border border-white/10 p-3 shadow-xl">
      <div className="flex gap-2 items-end">
        <textarea
          rows={2}
          value={input}
          onChange={e=>setInput(e.target.value)}
          onKeyDown={(e)=>{ if (e.key === 'Enter' && !e.shiftKey) { e.preventDefault(); canSend && onSend(); } }}
          placeholder="Ask anything… (Enter to send)"
          className="grow resize-none bg-transparent outline-none text-slate-100 placeholder:text-slate-500"
        />
        <button
          disabled={!canSend}
          onClick={onSend}
          className={`px-4 py-2 rounded-xl text-sm font-medium border ${canSend?"border-yellow-300/50 hover:bg-yellow-300/10":"border-white/10 opacity-40"}`}
        >Send</button>
      </div>
      <div className="mt-2 text-[11px] text-slate-500">
        {demoMode ? (
          <span>Demo Mode: responses are simulated locally. Toggle off to use real providers (requires server setup).</span>
        ) : (
          <span>Live Mode: your prompts may be sent to the selected provider via your server. Only sent if consent is checked.</span>
        )}
      </div>
    </div>
  </footer>

  {/* Developer Notes */}
  <section className="max-w-5xl mx-auto w-full px-4 py-6 text-xs text-slate-400">
    <h2 className="text-slate-200 font-semibold mb-2">Developer wiring (swap Demo for Live)</h2>
    <ol className="list-decimal ml-5 space-y-1">
      <li>Create server routes that proxy to providers. Example endpoints:
        <ul className="list-disc ml-5">
          <li><code>POST /api/chat/openai</code> → calls OpenAI (GPT) with your API key.</li>
          <li><code>POST /api/chat/anthropic</code> → calls Anthropics (Claude).</li>
          <li><code>POST /api/chat/google</code> → calls Gemini.</li>
          <li><code>POST /api/chat/xai</code> → calls Grok.</li>
          <li><code>POST /api/chat/deepseek</code> → calls DeepSeek.</li>
          <li><code>POST /api/chat/perplexity</code> → calls Perplexity.</li>
        </ul>
      </li>
      <li>Each route should accept JSON: <code>{`{ prompt, system, history, temperature }`}</code> and return <code>{`{ text }`}</code>.</li>
      <li>Store chat data/feedback only if <b>consent</b> is true. Use a DB like Postgres/Supabase/Firebase with a schema like:
        <pre className="bg-slate-900/70 p-2 rounded-lg overflow-x-auto">{`tables:

conversations(id, user_id, provider, created_at) messages(id, conversation_id, role, text, created_at) feedback(id, message_id, type, created_at) learned(id, user_prompt_hash, best_response, score, updated_at)`}</pre> </li> <li>To "train from users" safely: aggregate prompts/responses + feedback, then fine‑tune or build a retrieval/rerank layer. Start simple: rerank canned or previous replies with highest score.</li> </ol> </section> </div> ); }

// ------- Provider wiring (client) ------- async function sendToProvider({ provider, prompt, consent }: { provider: Provider; prompt: string; consent: boolean; }): Promise<string> { // IMPORTANT: For safety, never call provider APIs directly from the browser. // Instead, POST to your own server route. Below is a tiny router that expects // those routes to exist. Replace URLs as needed. const body = { prompt, consent, history: [], temperature: 0.7 }; const route = (() => { switch (provider) { case "GPT": return "/api/chat/openai"; case "Claude": return "/api/chat/anthropic"; case "Gemini": return "/api/chat/google"; case "Grok": return "/api/chat/xai"; case "DeepSeek": return "/api/chat/deepseek"; case "Perplexity": return "/api/chat/perplexity"; default: return "/api/chat/openai"; } })();

const res = await fetch(route, { method: "POST", headers: { "Content-Type": "application/json" }, body: JSON.stringify(body) }); if (!res.ok) throw new Error(Provider route failed (${res.status})); const data = await res.json(); return data.text || "(No text returned)"; }

