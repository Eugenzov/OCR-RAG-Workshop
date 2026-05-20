# Förutsättningar för workshop

Gör **allt** detta **innan** workshoppen. Nedladdningarna är ~2–3 GB totalt och blir långsamma över konferens-WiFi — gör det hemma över en bra uppkoppling.

Räkna med ~30 minuter om allt går smidigt.

## 1. Hårdvara

- **RAM:** 4 GB ledigt räcker bra (modellen använder ~1,3 GB)
- **Disk:** ~3 GB ledigt utrymme
- **Bara CPU duger** — ingen GPU krävs
- **OS:** Windows 10/11, macOS eller Linux

## 2. Python 3.10 eller nyare

- **Windows:** Installera från https://www.python.org/downloads/ — **kryssa i "Add Python to PATH"** på första installationsskärmen
- **macOS:** `brew install python` (eller använd installatorn från python.org)
- **Linux:** Oftast förinstallerat; annars `sudo apt install python3 python3-pip`

**Verifiera** i en ny terminal:
```
python --version
```
Ska visa `Python 3.10.x` eller högre. (På Windows fungerar även `py --version`.)

## 3. Tesseract OCR (binären, inte Python-paketet)

Detta är själva OCR-motorn. Python-paketet `pytesseract` är bara ett omslag runt den.

- **Windows:** Ladda ner installatorn från https://github.com/UB-Mannheim/tesseract/wiki (välj senaste 64-bitars `.exe`). Notera installationssökvägen — standard är antingen:
  - `C:\Program Files\Tesseract-OCR\` (installera för alla användare), eller
  - `C:\Users\<du>\AppData\Local\Programs\Tesseract-OCR\` (installera bara för dig)
- **macOS:** `brew install tesseract`
- **Linux:** `sudo apt install tesseract-ocr`

**Verifiera:**
```
tesseract --version
```
Ska visa `tesseract 5.x.x`. Om det säger "command not found" på Windows behöver du peka notebooken mot exe-filen — instruktioner finns i notebooken.

## 4. Ollama (lokal LLM-körmotor)

- Ladda ner och installera: https://ollama.com/download
- Efter installationen kör Ollama automatiskt i bakgrunden (leta efter llama-ikonen i aktivitetsfältet / menyraden)

**Verifiera:**
```
ollama --version
```

## 5. Hämta de två modellerna (~1,3 GB nedladdning)

I en terminal:
```
ollama pull qwen2.5:1.5b
ollama pull nomic-embed-text
```

**Verifiera:**
```
ollama list
```
Ska visa båda modellerna.

## 6. Python-paket

Från workshop-mappen:
```
pip install -r requirements.txt
```

Om `pip` inte känns igen på Windows, använd:
```
py -m pip install -r requirements.txt
```

## 7. Notebook-miljö

Välj **en**:
- **VS Code** + tillägget **Jupyter** (rekommenderas — enklast)
- **JupyterLab:** `pip install jupyterlab`, kör sedan `jupyter lab` i workshop-mappen

## Vanliga problem

| Symptom | Lösning |
|---|---|
| `pip not recognized` (Windows) | Använd `py -m pip` eller installera om Python med "Add to PATH" ikryssat |
| `TesseractNotFoundError` | Tesseract-binären är inte installerad eller finns inte i PATH — se notebooken för `tesseract_cmd`-överstyrningen |
| `ConnectionError: Failed to connect to Ollama` | Ollama körs inte — starta från aktivitetsfältet, eller kör `ollama serve` |
| `model 'qwen2.5:1.5b' not found` | Du hoppade över steg 5 — kör `ollama pull qwen2.5:1.5b` |
