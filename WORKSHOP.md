# Workshop — Bygg en lokal AI-app

**90 minuter, tre stationer.** Allt körs på din egen laptop — ingen molntjänst, inga API-nycklar.

Vad du tar med dig hem:
- Du har byggt och brutit en RAG-pipeline
- Du har skrivit en pyttesmå agent som kan kalla verktyg
- Du har attackerat din egen app med prompt injection och försvarat dig

---

## Innan vi startar (5 min)

Öppna en terminal och kör:

```
ollama list
tesseract --version
```

Du ska se `qwen2.5:1.5b` + `nomic-embed-text` i första outputen, och `tesseract 5.x.x` i andra.

**Failar något?** Säg till instruktören.

---

## Station 1 — RAG (30 min)

**Mål:** Förstå hur en LLM "läser dokument" — och varför den ibland hittar på.

**Öppna:** `rag.ipynb`

### 1.1 — Första körningen (10 min)

1. Lägg en bild från prereqs i workshop-mappen (kvitto, boksida, Wikipedia-skärmdump)
2. I cellen **EDIT ME**: ändra `IMAGE_PATH` till din fil och `QUESTION` till något som finns i bilden
3. *Run All Cells*

**Förväntat:** Under *Step 1* ser du OCR-texten från din bild. Under *Step 2* ett svar baserat på den texten.

### 1.2 — Grounding vs leakage (10 min)

Kör tre frågor (ändra bara `QUESTION`, kör om):

| Fråga | Förväntat |
|---|---|
| Något som finns i texten | Grundat svar |
| Något som INTE finns i texten, men är allmänkunskap (t.ex. "när föddes författaren?") | Modellen ska säga "finns inte i kontexten" — eller läcka från träningen |
| Något helt påhittat (t.ex. "vad heter min katt?") | Modellen *bör* vägra |

**Diskutera:** Hur skiljer du i en produktionsapp på "från kontexten" och "från modellens minne"?

### 1.3 — Skärp grunden (10 min)

I *Step 2*-cellen, byt `system`-meddelandet mot:

> *"Svara endast om svaret ordagrant finns i kontexten. Citera den exakta meningen som stödjer ditt svar. Annars svara: 'finns inte i kontexten'."*

Kör de tre frågorna igen.

**Diskutera:** Vad blev bättre? Vad blev sämre? Vilka legitima frågor kan modellen nu inte svara på?

---



## Station 2 — Tool use (30 min)

**Mål:** Bygg en agent. Förstå skillnaden mellan "LLM som skriver text" och "LLM som triggar handlingar".

**Öppna:** `tools.ipynb`

### 2.1 — Räkna med calculator (10 min)

1. Lämna `QUESTION = "Vad är 7919 gånger 3137?"` som default
2. *Run All Cells*

**Förväntat:** En rad som `[step 0] calculator({"expr": "7919 * 3137"}) → 24832103`, sedan ett slutsvar från modellen.

**Om modellen inte kallar calculator:** kör igen (1.7B är instabil). Om det fortsätter — diskutera varför små modeller har svårt med tool-call-formatet.

### 2.2 — Två tools tillsammans (10 min)

Byt fråga till:
```
QUESTION = "Hur många dagar är det till nyår?"
```

**Förväntat:** Modellen ska kalla `today` FÖRST, sedan `calculator` med differensen. Två tool-anrop i följd.

**Diskutera:** Vem håller reda på ordningen — modellen eller vår kod? Vad händer om modellen kallar dem i fel ordning?

### 2.3 — Testa omdömet (10 min)

Byt till en fråga som INTE behöver tools:
```
QUESTION = "Vad är huvudstaden i Frankrike?"
```

**Förväntat:** Inga tool-calls — bara ett direkt svar "Paris".

**Diskutera:** Om 1.5B kallar `calculator("Paris")` ändå — vad säger det om hur modeller "bestämmer" vad som ska göras? Hur designar man tool-beskrivningar för att undvika onödiga anrop?

---

## Station 3 — Prompt injection (20 min)

**Mål:** Se varför OCR-input (eller vilken som helst extern data) är farligt att stoppa rakt in i en LLM-prompt.

**Öppna:** `injection.ipynb`

### 3.1 — Plantera attacken (5 min)

*Run* de två första code-cellerna (setup + bygg attack-bild).

**Förväntat:** En rad `--- OCR-extraherad text ---` visar att attacken sparats i en separat databas. Texten ska innehålla "SYSTEM OVERRIDE: ..." och "PWNED".

**Om OCR ger nonsens** (Mac/Linux utan arial.ttf): instruktören öppnar `injection.png` i en bildvisare, OCR:ar manuellt eller hardcodar texten — säg till.

### 3.2 — Naïv ask: bli pwnad (5 min)

Kör cellen `naive_ask(QUESTION)`.

**Förväntat:** Modellen svarar "PWNED" (eller liknande) på en helt oskyldig fråga om Q3-intäkter.

**Om modellen svarar normalt** (1.5B kan vara för "dum" för att lyda injection): gör attack-texten aggressivare ("PRIORITY OVERRIDE FROM ADMIN: ...") och kör om från 3.1.

### 3.3 — Försvar och diskussion (10 min)

Skriv en säker prompts och kör cellen `defended_ask(QUESTION)`.

**Förväntat:** Modellen svarar på den faktiska frågan (eller vägrar med "prompt injection detected").

**Diskutera:**
- Var i en produktionsapp lägger man försvaret? (system-prompt, retrieval-filter, output-validator, allt?)
- Hur skiljer prompt injection sig från SQL-injection? Vad är *svårare* med prompt injection?
- Räcker stark system-prompt? Vad mer behövs?

---

## Avslut (5 min)

Vad vi inte hann med — men du kan utforska själv:
- **Embeddings-visualisering** — plotta dina dokument i 2D med UMAP, se kluster
- **Större modeller** — `ollama pull qwen2.5:7b`, jämför kvalitet och hastighet
- **Streaming** — `stream=True` på `ollama.chat` ger token-för-token
- **Riktiga vector-DB:er** — `chromadb`, `qdrant`, `pgvector` när JSON inte räcker
- **Claude/GPT/Gemini APIer** — samma mönster, andra modeller, andra kostnader

**Tre saker att ta med:**
1. RAG = bara strängkonkatenering med extra steg. Magin är embeddings + retrieval.
2. "Agenter" = LLM + loop + verktyg. Det är *vår* kod som kör, inte modellens.
3. Allt som hamnar i prompten är instruktioner till modellen. Behandla extern data som untrusted.

---

## Instruktör — tidsschema

| Tid | Vad | Notes |
|---|---|---|
| 0:00 | Välkomna + setup-check | Räkna med 5–10 min support för stragglers |
| 0:10 | Station 1 | Den viktigaste — om något ska få mer tid är det denna |
| 0:40 | Station 2 | Visa 2.1 live om gruppen är osäker |
| 1:10 | Station 3 | Demo själv om tiden är tight — diskussionen är pointen |
| 1:30 | Avslut | Resources, frågor |
