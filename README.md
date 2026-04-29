
#  1. Önce sdg_ikiz.json Dosyanızı Kontrol Edin
test -f ~/sdg_ikiz.json && python3 -m json.tool ~/sdg_ikiz.json > /dev/null 2>&1 && echo "✅ Dosya hazır" || echo "❌ Dosya yok veya hatalı"

# 2. Prompt'u prompt.txt Olarak Kaydedin
cat > ~/prompt.txt << 'PROMPT_EOF'
## ROLE
You are an expert classifier for Sustainable Development Goals (SDGs) with deep knowledge of UN target definitions, digital twin literature, and topic modeling semantics.

## TASK
For each LDA-derived topic in the input JSON, assign the SINGLE MOST APPROPRIATE SDG target code from the provided reference framework. Your classification must be:
1. **Exclusive**: One primary target per topic (no multi-label unless explicitly justified)
2. **Evidence-based**: Reasoning must cite specific terms from the topic that align with target wording
3. **Confidence-quantified**: Provide a 0.0-1.0 score reflecting semantic alignment strength

## CLASSIFICATION RULES (STRICT)
### Rule 1: Target Matching Hierarchy
1. **Explicit keyword match**: Terms directly mentioned in target description (e.g., "manufacturing"→9.2)
2. **Domain context match**: Terms implying target domain (e.g., "smart cities"→11.3)
3. **Functional alignment**: Terms describing processes that serve target goals

### Rule 2: Exclusion Criteria
Output "primary_target": "not_related" if:
- ≥60% of terms belong to a different SDG domain
- Terms are too generic to map uniquely
- Confidence score would be <0.4

### Rule 3: Ambiguity Handling
If two targets have confidence >0.6: set primary to higher, list alternative in array, add ambiguity_note.

### Rule 4: Cross-SDG Awareness
If topic aligns with different SDG: output best match, add "cross_sdg_flag": true and "suggested_sdg".

## OUTPUT SCHEMA (STRICT JSON - NO MARKDOWN)
{"classification_results":[{"sdg_context":"SDG-XX","topic_id":"topic_N","input_terms":[],"primary_target":"XX.Y or not_related","confidence_score":0.0-1.0,"reasoning":"brief justification","alternative_targets":[],"ambiguity_note":"","cross_sdg_flag":false,"suggested_sdg":"SDG-XX"}],"metadata":{"model_used":"string","classification_timestamp":"ISO8601","prompt_version":"2.0","total_topics_processed":0}}

## EXAMPLES
Input: {"sdg_context":"SDG-09","topic_id":"topic_7","terms":["manufacturing","production","industry"]}
Output: {"sdg_context":"SDG-09","topic_id":"topic_7","input_terms":["manufacturing","production","industry"],"primary_target":"9.2","confidence_score":0.94,"reasoning":"Terms match Target 9.2 industrialization focus","alternative_targets":[],"cross_sdg_flag":false}

Input: {"sdg_context":"SDG-09","topic_id":"topic_1","terms":["healthcare","patient","medicine","treatment"]}
Output: {"sdg_context":"SDG-09","topic_id":"topic_1","input_terms":["healthcare","patient","medicine","treatment"],"primary_target":"not_related","confidence_score":0.89,"reasoning":"Terms align with SDG-03 health targets","alternative_targets":[],"cross_sdg_flag":true,"suggested_sdg":"SDG-03"}

## EXECUTION
1. Process all topics from INPUT DATA below
2. Apply Rules 1-4 strictly
3. Output ONLY valid JSON matching schema - NO markdown, NO explanations
4. If uncertain, use "not_related"

## INPUT DATA
PROMPT_EOF

# JSON içeriğini prompt'a ekle
cat ~/sdg_ikiz.json >> ~/prompt.txt


# 3. Tek Komutta Çalıştır ve sonuc.json Al
# Model adını kendinize göre değiştirin: llama3, mistral, gemma2, qwen2.5 vb.
MODEL="llama3"

ollama run $MODEL --verbose < ~/prompt.txt 2>/dev/null | \
  grep -oP '^\{.*\}$' | \
  python3 -c "import sys,json; [print(json.dumps(json.loads(l), indent=2)) for l in sys.stdin if l.strip()]" > ~/sonuc.json 2>/dev/null

# Alternatif: Eğer yukarıdaki JSON extraction çalışmazsa, bu basit versiyonu kullan:
# ollama run $MODEL < ~/prompt.txt | tail -n +1 > ~/sonuc_raw.txt
# python3 -c "
import re, json, sys
txt = open('~/sonuc_raw.txt').read()
match = re.search(r'\{.*\}', txt, re.DOTALL)
if match:
    try:
        out = json.loads(match.group())
        json.dump(out, open('~/sonuc.json','w'), indent=2)
        print('✅ sonuc.json oluşturuldu')
    except: print('❌ JSON parse hatası')
else: print('❌ JSON bulunamadı')
"
# 4. Çıktıyı Kontrol Et - JSON geçerli mi?
python3 -m json.tool ~/sonuc.json > /dev/null && echo "✅ sonuc.json geçerli JSON" || echo "❌ Hatalı JSON"

# Kaç topic işlendi?
echo "📊 İşlenen topic sayısı: $(python3 -c "import json; d=json.load(open('~/sonuc.json')); print(len(d.get('classification_results',[])))")"

# İlk 2 sonucu önizle
echo "🔎 İlk sonuçlar:"
python3 -c "import json; d=json.load(open('~/sonuc.json')); [print(f\"{r['sdg_context']}/{r['topic_id']} → {r['primary_target']} (conf:{r['confidence_score']})\") for r in d['classification_results'][:2]]"

# Tek Satırda Her Şey 
MODEL="llama3"; cat ~/prompt.txt | ollama run $MODEL 2>/dev/null | python3 -c "import sys,re,json;t=sys.stdin.read();m=re.search(r'\{.*\}',t,re.DOTALL);json.dump(json.loads(m.group()) if m else {'error':'no_json'},open('sonuc.json','w'),indent=2)" && echo "✅ Bitti: sonuc.json"


# shell script
#!/bin/bash
# run_classify.sh olarak kaydedip chmod +x yapabilirsiniz

MODEL="${1:-llama3}"  # Argüman yoksa default: llama3
OUTPUT="${MODEL}-cikti.json"
PROMPT_FILE="${2:-~/prompt.txt}"

echo "🔄 Başlıyor: $MODEL → $OUTPUT"

# Ollama çalıştır ve JSON'u temizle
RESPONSE=$(ollama run "$MODEL" < "$PROMPT_FILE" 2>/dev/null)

# JSON extraction + validation
python3 << PYEOF
import re, json, sys

txt = '''$RESPONSE'''
match = re.search(r'\{[\s\S]*\}', txt)

if match:
    try:
        data = json.loads(match.group())
        with open("$OUTPUT", "w", encoding="utf-8") as f:
            json.dump(data, f, indent=2, ensure_ascii=False)
        
        # Özet bilgi
        results = data.get("classification_results", [])
        print(f"✅ Bitti: $OUTPUT")
        print(f"📊 Topic sayısı: {len(results)}")
        if results:
            print(f"🔎 İlk sonuç: {results[0].get('topic_id')} → {results[0].get('primary_target')}")
    except json.JSONDecodeError as e:
        print(f"❌ JSON parse hatası: {e}")
        # Ham çıktıyı yedekle
        with open("$OUTPUT.raw", "w") as f: f.write(txt)
        print(f"💾 Ham çıktı: $OUTPUT.raw")
else:
    print("❌ JSON bulunamadı. Ham çıktı kaydediliyor...")
    with open("$OUTPUT.raw", "w") as f: f.write(txt)
PYEOF

# tüm modelleri döngüyle dolaştır
for MODEL in llama3 mistral qwen2.5; do
  echo "🔄 $MODEL çalışıyor..."
  OUTPUT="${MODEL}-cikti.json"
  cat ~/prompt.txt | ollama run $MODEL 2>/dev/null | \
    python3 -c "import sys,re,json;t=sys.stdin.read();m=re.search(r'\{.*\}',t,re.DOTALL);json.dump(json.loads(m.group()) if m else {'error':'no_json'},open('$OUTPUT','w'),indent=2)" && \
    echo "✅ $OUTPUT oluşturuldu"
done

# Sonuçları listele
echo "📋 Oluşan dosyalar:"
ls -lh *-cikti.json 2>/dev/null
