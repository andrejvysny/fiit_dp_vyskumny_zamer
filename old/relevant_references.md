Ako Váš Asistent pre magisterskú prácu som na základe poslednej podrobnej odpovede vygeneroval kompletný zoznam všetkých citovaných referencií a dokumentov. Tieto referencie sú rozdelené podľa kľúčových oblastí, na ktoré sa Váš návrh práce zameriava, s explicitným uvedením kľúčových interných citácií, ktoré boli použité.

**Použité a relevantné zdroje pre tému LLM Agent assisted Focused Crawling a RAG:**

### I. Metodické a teoretické rámce (Web Crawling a Sémantické Zameranie)

| Citácie | Popis a Kľúčové Zistenia | Dokumenty |
| :--- | :--- | :--- |
| **,,,,,** | **Focused Crawling:** Klasické definície Focused Crawlingu, odkazy na PageRank, indegree, a CTR prediktory ako tradičné mechanizmy navádzania. | `1406.5690v1.pdf` |
| **,,,,** | **Paralelné/Doménové Crawlovanie (WEB-SAILOR/BUbiNG):** Princípy distribuovanej architektúry (Seed-server a Crawl-klienti) a doménovo-špecifické crawlovanie (DSet). | `jcssp.2015.120.126.pdf` |
| **,,,** | **Sémantický Crawler (NER):** Metóda, ktorá využíva Named Entity Recognition (NER) na validáciu sémantického zmyslu kľúčového slova (WSD) a dramatické zníženie irelevantných dokumentov. | `Semantic_crawling_An_approach_based_on_Named_Entity_Recognition.pdf` |

### II. Neurálne Scoring, Validácia a Filtrovanie Kvality (LLM-Assisted Prioritizácia)

| Citácie | Popis a Kľúčové Zistenia | Dokumenty |
| :--- | :--- | :--- |
| **,,,,,,** | **Neural Prioritisation:** LLM-založené modely (ako $Q_{\theta}$) na odhad **sémantickej kvality** webových stránok pre prioritizáciu. Využitie heuristiky susedstva pre **aproximáciu kvality** URL. | `2506.16146v2.pdf` |
| **,,,,** | **LLM-Guided Data Selection (LMDS) / Knowledge Distillation:** Použitie silného LLM ($LM_{large}$) na generovanie labelov **kvality/vzdelávacej hodnoty** ("Yes"/"No"), ktoré sú destilované do menšieho modelu ($LM_{small}$) pre škálovateľné filtrovanie korpusu. | `2406.04638v1.pdf` |
| **,,,,** | **Query-Agnostic Scoring:** Modely ako **Neural Quality Estimator** (napr. QT5-small) odhadujúce log-pravdepodobnosť, že dokument bude relevantný *k aspoň jednému* dotazu. | `2407.12170v1.pdf` |
| **** | **Vektorová Podobnosť:** Scoring dokumentov na základe **relatívnej vzdialenosti** kontextualizovaného embeddingu dokumentu od ground truth vektora (napr. ThreatCrawl). | `2304.11960v4.pdf` |

### III. Multi-Agentné Architektúry a Reinforcement Learning (RL)

| Citácie | Popis a Kľúčové Zistenia | Dokumenty |
| :--- | :--- | :--- |
| **,,,,,,** | **LLM ako Reviewer (ResearchAgent):** Multi-agentný systém, kde LLM inštancie (ReviewingAgents) poskytujú **kritiku a spätnú väzbu** na základe kritérií odvodených od ľudských úsudkov. |
| **,,,,,,** | **End-to-end RL pre Agenty (DeepResearcher):** Aplikácia RL tréningu v **reálnom webovom prostredí** (nie statickom RAG korpuse). RL sa aplikuje po SFT. | `2509.13309v2.pdf` |
| **,,,** | **Akcie a Stav:** Agenti generujú akcie pomocou **Tool Calls** (`web search`, `web browse`). Stav je definovaný kontextom a históriou akcií (ReAct). | `2507.02592v1.pdf` |
| **,,,,** | **RL a Validácia Odmeny:** RL sa používa na maximalizáciu **správnosti finálnej odpovede**. **F1 Score** sa používa ako pravidlo pre **Reward**. **Rejection Sampling Fine-Tuning (RFT)** zamietne celé trajektórie, ktoré nevedú k správnej odpovedi. LLM (GPT-4o-mini) funguje ako **LLM-as-a-Judge** pre scoring. | `2507.02592v1.pdf`, `2504.03160v4.pdf` |

### IV. Hodnotiace Metriky

| Citácie | Popis a Kľúčové Zistenia | Dokumenty |
| :--- | :--- | :--- |
| **,,** | **Crawling Metriky:** **Harvest Rate (HR)** ako miera efektivity a **MaxNDCG** ako horná hranica potenciálnej vyhľadávacej efektivity na prehľadanom korpuse. | `3637528.3671620.pdf` |
| **,,,** | **Rankovacie Metriky (IR):** **nDCG** (@10), **MRR**, a **MAP** na hodnotenie kvality poradia získaných dokumentov. | `3637528.3671620.pdf` |

### V. Konceptuálne Rámce

*   **RAG (Retrieval-Augmented Generation):** Použitie retrieve-augmented architektúry na získavanie aktuálnych dát a zmiernenie **hallucinácií**.
*   **Špecializované dátové sady:** Použitie dátových sád ako **REDSTONE** pre získavanie doménovo-špecifických dát (napr. kód, matematika, QA), ktoré zlepšujú výkon modelu.