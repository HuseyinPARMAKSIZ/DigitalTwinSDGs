Mapping The Sustainable Development Goals in Digital Twin Research: Low Code Topic Modeling and LLM-Based Evaluation Framework başlıklı akademik çalışma kapsamında,

Digital Twin çalışmalarının SDG analizini belirlemek amacıyla, knime_5.5.0 kullanılarak, LDA ile elde edilmiş Topic-Terms'ler kullanılarak yerelde,
ollama ile mistral ve llama3   -  ollama run llama3 "$(cat subSDGprompt.txt)" -> tek seferde token problemi olunca yerelde aşağıdaki gibi tek tek işletildi.


ollama run mistral "$(cat prompt_sdg03.txt)" > mistral_sdg03.json
echo "SDG-03 tamamlandı. Sonuçlar: mistral_sdg03.json"

ollama run laguna-xs.2 "$(cat prompt_sdg09.txt)" > laguna-xs.2_sdg09.json


jq -s 'add' mistral_sdg09.json mistral_sdg11.json mistral_sdg12.json mistral_sdg03.json > mistral_local.json


openrouter ile nemotron 3 nano omni, Hy3 preview, ...

Analizlerin sonucu sub-SDG'ler netleştirildi.
