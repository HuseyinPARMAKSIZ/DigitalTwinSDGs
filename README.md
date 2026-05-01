Mapping The Sustainable Development Goals in Digital Twin Research: Low Code Topic Modeling and LLM-Based Evaluation Framework başlıklı akademik çalışma kapsamında,

Digital Twin çalışmalarının SDG analizini belirlemek amacıyla, knime_5.5.0 kullanılarak, LDA ile elde edilmiş Topic-Terms'ler kullanılarak yerelde,
ollama ile mistral ve llama3   -  ollama run llama3 "$(cat subSDGprompt.txt)" -> tek seferde token problemi olunca yerelde aşağıdaki gibi tek tek işletildi.


ollama run mistral "$(cat prompt_sdg03.txt)" > mistral_sdg03.json
echo "SDG-03 tamamlandı. Sonuçlar: mistral_sdg03.json"

ollama run laguna-xs.2 "$(cat prompt_sdg09.txt)" > laguna-xs.2_sdg09.json


jq -s 'add' mistral_sdg09.json mistral_sdg11.json mistral_sdg12.json mistral_sdg03.json > mistral_local.json

subSDGprompt.txt prompt olarak belirlendikten sonra, 
openrouter ile nemotron 3 nano omni, Hy3 preview, OwlAlpha ve Gpt-oss-120b ve ling-2.6-1T modelleri koşturuldu.
web üzerinden deepseek-v4, kimi2.6, qwen3.6-plus modelleri koşturuldu.
yerelde ise llama3, mistral, phi4 ve laguna-xs.2 (ollama version is 0.22.1 güncellemeden çalışmam dedi) modelleri koşturuldu.

Analizlerin sonucu sub-SDG'ler netleştirildi.
