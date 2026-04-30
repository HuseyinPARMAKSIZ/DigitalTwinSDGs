Mapping The Sustainable Development Goals in Digital Twin Research: Low Code Topic Modeling and LLM-Based Evaluation Framework başlıklı akademik çalışma kapsamında,

Digital Twin çalışmalarının SDG analizini belirlemek amacıyla, knime_5.5.0 kullanılarak, LDA ile elde edilmiş Topic-Terms'ler kullanılarak yerelde,
ollama ile mistral ve llama3   -  ollama run llama3 "$(cat subSDGprompt.txt)"


# Tüm sonuçları tek dosyada birleştir
jq -s 'add' results_sdg09.json results_sdg11.json results_sdg12.json results_sdg03.json > all_results.json


openrouter ile nemotron 3 nano omni, Hy3 preview, ...

Analizlerin sonucu sub-SDG'ler netleştirildi.
