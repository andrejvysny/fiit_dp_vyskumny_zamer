# Analysis and research created by Google NotebookLM
---STARTanalysis---
Dobrý deň. Ako Váš Asistent pre magisterskú prácu som sformuloval diskusiu k Vášmu predbežnému zámeru práce s cieľom **objasniť a zamerať** Váš výskum na základe predložených zdrojov.

Vaša téma "Multi-agent, LLM-assisted semantic web/document crawler, ktorý používa reinforcement learning a LLM-based validáciu a scoring na zlepšenie cieleného získavania a rankovania dokumentov, s možnou integráciou do RAG systému" je mimoriadne komplexná. Nižšie uvádzam analýzu jednotlivých komponentov, pričom zdôrazňujem podporu v dokumentoch a **výskumné medzery**.

---

## 1. Navádzanie Crawlerov k Relevantnému Obsahu

Existujúce systémy využívajú rôzne stratégie, ktoré prešli od štatistických metód k neurálnemu, sémantickému prioritizovaniu.

### A. Tradičné a Zamerané Crawlovanie (Focused Crawling)

*   **Štatistická relevancia:** Crawlery boli tradične navádzané na základe **PageRank**, **indegree**, alebo **Click-Through Rate (CTR)** prediktorov. Tieto prístupy sú globálne, ale vyžadujú rozsiahle dáta o linkovom grafe alebo popularite.
*   **Doménovo-špecifické (DSet):** Paralelné crawlery ako **WEB-SAILOR** rozdeľujú web na logické partície (napr. podľa doménových prípon **.com, .edu, .net, .biz**, tzv. **DSet**) a dynamicky prideľujú úlohy jednotlivým Crawl-klientom zo **Seed-servera**.
*   **Tematické zameranie:** **Focused Crawlers** vyhľadávajú dokumenty relevantné k vopred definovanému súboru **tém**.
*   **Sémantická validácia entít (NER):** Semantic Crawler používa metódy ako **NER** (Named Entity Recognition) na filtrovanie dokumentov na základe **sémantického zmyslu kľúčového slova** (sense).

### B. LLM-Assisted Prioritizácia (Neural Prioritisation)

Moderné systémy využívajú neurálne modely, aby sa prispôsobili posunu k **natural language queries**.

*   **Sémantické Skórovanie Kvality (Neural Quality Estimators):** LLM-založené modely ($Q_{\theta}$) sa trénujú na odhad **sémantickej kvality** webových stránok, čo slúži na **prioritizáciu prehľadávania** (crawling prioritisation). Crawler uprednostňuje stránky s vysokým skóre.
*   **Predikcia Kvality zo Susedstva:** Navrhuje sa, že dokumenty s podobnou sémantickou kvalitou sa pravdepodobne odkazujú navzájom (**high-quality to high-quality**). Táto heuristika sa používa na **aproximáciu kvality** URL adresy pred jej stiahnutím.
*   **LLM-Guided Data Selection (LMDS):** Pri príprave dát pre pre-tréning LLM, silný model ($LM_{large}$) označuje dokumenty z hľadiska **kvality a vzdelávacej hodnoty**. Tieto labely sa následne destilujú do malého modelu ($LM_{small}$), ktorý filtruje celý korpus (napr. RPJ-CC).

---

## 2. Scoring a Validácia Relevancie a Kvality Dokumentov

Hodnotenie kvality je kľúčové pre rozhodovanie agenta o ďalšom postupe.

### A. Scoring Založený na Embedingu a Klasifikácii

*   **Kvalita ako predikcia relevancie:** **Neural Quality Estimator** je model (napr. QT5-small), ktorý odhaduje **log-pravdepodobnosť** toho, že dokument bude **relevantný k aspoň jednému užívateľskému dotazu**. Toto je **query-agnostic** prístup k relevancii.
*   **Využitie perplexity (PPL):** Skóre kvality možno odvodiť z **perplexity (PPL)** predtrenovaného jazykového modelu, čo nevyžaduje explicitnú supervíziu na dátach relevancie.
*   **Vektorová podobnosť:** V doménovo špecifickom prehľadávaní (napr. CTI) sa sémantická relevantnosť určuje pomocou **kontextualizovaných dokumentových embeddingov** (napr. BERT, S-BERT). Relevantné dokumenty sú tie, ktoré majú minimálnu **relatívnu vzdialenosť** od vektora **ground truth**.
*   **Validácia pre RAG:** V RAG systémoch je HTML obsah rozdelený na menšie **chunks**, ktoré sú prevedené na **vektory**. Extrakcia dát sa vykonáva len z **top k chunks**, ktoré vykazujú najvyššiu **sémantickú zhodu** (semantic alignment) s užívateľským dotazom.

### B. Validácia pre LLM Pre-tréning a Fine-tuning

*   **LLM Labely Kvality:** LLM ($LM_{large}$) hodnotí dokument (súbor tokenov) pomocou promptu, či je **vzdelávací a pútavý** pre študenta STEM/humanitných vied. Výsledkom je label **"Yes" alebo "No"**. Tento label slúži ako **supervízny signál** pre menší model ($LM_{small}$).
*   **Validácia LLM výstupu (RFT):** Pri trénovaní RL agentov sa na validáciu **celej trajektórie** (vrátane usudzovania a akcií) používa **Rejection Sampling Fine-Tuning (RFT)**. **Trajektórie sú zamietnuté, ak nevedú k správnej finálnej odpovedi**.

---

## 3. Používané Hodnotiace Metriky (Ranking a Crawling)

Váš výskum bude vyžadovať kombináciu tradičných a moderných metrík pre IR a agentov.

### A. Tradičné IR a Lexikálne Metriky

*   **Presnosť a pokrytie:**
    *   **Precision, Recall, F1 Score**. F1 skóre sa v prípade LLM agentov používa aj ako pravidlo pre **Reward** pri RL tréningu.
*   **Rankovanie a Poradie:**
    *   **Mean Reciprocal Rank (MRR)**.
    *   **Normalized Discounted Cumulative Gain (nDCG)**, často merané na @10.
    *   **Mean Average Precision (MAP)**.
*   **Sémantické Skóre:** **BLEU-1 (B1)** na zachytenie lexikálnej podobnosti na úrovni unigramov.

### B. Crawlovanie a Agenti-Špecifické Metriky

*   **Harvest Rate (HR):** Miera schopnosti crawlera maximalizovať nájdenie relevantných stránok pri minimalizácii irelevantných. HR sa počíta ako **|Relevantné Stránky| / |Prehľadané Stránky|** v danom čase $t$.
*   **MaxNDCG:** Najvyššia hodnota nDCG, ktorú by bolo možné dosiahnuť ideálnym rankerom na korpuse prehľadanom do času $t$. Slúži ako **horná hranica (upper-bound)** pre potenciálnu vyhľadávaciu efektivitu.
*   **Freshness (Čerstvosť):** Miera aktuálnosti lokálneho repozitára. Hodnotenie Freshness je kľúčovým cieľom **Incremental Parallel Web Crawler**.

### C. LLM-Based Metriky

*   **LLM-as-a-Judge (J):** Používa sa ako metrika na hodnotenie **faktuálnej správnosti, relevantnosti, kompletnosti a kontextovej vhodnosti** vygenerovanej odpovede. Je považovaná za viac **human-aligned** metriku ako F1 alebo B1.

---

## 4. LLM a Hodnotenie Kvality, Aktuálnosti a Dôveryhodnosti

LLM preberajú rolu sofistikovaných filtrov a klasifikátorov.

*   **Sémantické pochopenie:** LLM majú **hlboké pochopenie sémantiky jazyka** a **rozsiahle znalosti**, čo im umožňuje prekonávať problémy s **vocabulary mismatch** a presnejšie identifikovať zámer dotazu (query intent) v porovnaní s tradičnými štatistickými metódami.
*   **Generovanie Dát Kvality:** LLM generujú dáta pre tréning menších modelov (Knowledge Distillation). Silný inštrukčný model ($LM_{large}$) generuje labely kvality/vzdelávacej hodnoty, ktoré sa potom použijú na **škálovateľné filtrovanie** stoviek miliárd dokumentov. Model s 175-krát menším počtom parametrov môže dosiahnuť porovnateľný výkon.
*   **Aktuálnosť/Hallucinácie (RAG):** RAG systémy explicitne využívajú LLM na prácu s **reálne získanými a aktuálnymi dátami** z externého korpusu. **Retriever** v RAG architektúre vkladá relevantné úryvky do kontextu LLM (Reader), čím sa zmierňuje problém **hallucinácií**.
*   **Task-Awareness:** LLM môžu byť vyzvané, aby integrovali **task-specific instruction** do svojej logiky (napr. určiť **zamýšľaný zdroj** a **typ obsahu** pri vyhľadávaní), čo zlepšuje presnosť zachytenia zámeru užívateľa.
*   **Dôveryhodnosť (Weakly Supported):** Hoci integrácia referencií do LLM výrazne zmierňuje problém s halucináciami, stále **zostáva neisté, či sa LLM na tieto podporné materiály odvolávajú** pri generovaní odpovedí. Dôveryhodnosť záverečných odpovedí tak môže byť nižšia ako pri tradičných IR systémoch.

---

## 5. LLM ako „Reviewer“ alebo „Kritik“ (LLM-as-a-Judge)

LLM sú používané nielen na generovanie obsahu, ale aj na jeho hodnotenie a iteratívne zlepšovanie.

### A. ResearchAgent a ReviewingAgents

Systém **ResearchAgent** navrhnutý na generovanie vedeckých nápadov implementuje viacagentný systém, kde LLM hrajú rolu kritikov.

*   **Funkcia:** Používajú sa LLM inštancie, nazývané **ReviewingAgents**, ktoré **simulujú komunitu kolegov**.
*   **Proces:** **ReviewingAgents** poskytujú **recenzie a konštruktívnu kritiku** (Reviews and Feedback) k vygenerovaným častiam výskumného nápadu (Problém, Metóda, Experimentálny dizajn).
*   **Kritériá:** Kritériá sú odvodené od **ľudských úsudkov** a sú špecifické pre typ výstupu (napr. pre *Problém* sa hodnotí **Clarity, Relevance, Originality, Feasibility, Significance**).
*   **Výsledok:** ResearchAgent **iteratívne aktualizuje a spresňuje** svoje výstupy na základe tejto spätnej väzby.

### B. LLM pre Reranking a Filtrovanie

*   **Generovanie vysvetlení:** LLM môžu generovať **vysvetlenia** (explanations) k dátam (ExaRanker), ktoré sa následne použijú na trénovanie rankingového modelu.
*   **LLM-as-a-Judge pre RL:** V tréningu RL agentov (DeepResearcher) sa **GPT-4o-mini** používa na posúdenie, či je **odpoveď agenta správna** voči otázke a ground truth odpovedi. Toto skóre sa následne používa ako signál pre **Rejection Sampling** (pozri časť 7).

---

## 6. Multi-Agentné Systémy pre Vyhľadávanie a Crawlovanie

V dokumentoch sú popísané dva hlavné typy multi-agentných architektúr: tradičné distribuované crawlery a moderné LLM-riadené systémy.

### A. LLM-Riadené Multi-Agentné Architektúry

*   **DeepResearcher (Deep Research):** Využíva **špecializovanú multi-agentnú architektúru**, kde dedikovaní **browsing agents** extrahujú relevantné informácie z webových stránok. Tieto agenty spracúvajú segmenty webových stránok sekvenčne a rozhodujú, či **pokračovať v čítaní** ďalšieho segmentu alebo **zastaviť**, čím sa optimalizuje extrakcia informácií.
*   **ResearchAgent (Iteratívna Tvorba):** Architektúra kombinuje centrálny **ResearchAgent** (LLM pre generovanie nápadov) s viacerými **ReviewingAgents** (LLM pre kritiku a spätnú väzbu) v iteratívnom cykle.

### B. Tradičné Paralelné Crawlery

Tieto systémy sú multi-agentné z hľadiska distribuovaného výpočtu, nie kognitívnej stratégie.

*   **WEB-SAILOR:** Architektúra **Client-Server** s centrálnym **Seed-serverom** a distribuovanými **Crawl-clientmi**. Server má **globálny prehľad** o webe a robí **rozhodnutia o prehľadávaní**. Klienti prehľadávajú pridelené **DSety**.
*   **WebParF:** Rozdeľuje **URL frontier** na viacero prioritizovaných frontov, ktoré sú prideľované jednotlivým **Crawl-procesom**.
*   **BUbiNG:** **Plne distribuovaný crawler** bez centrálnej kontroly. Používa **identické a autonómne agenty** s **consistent hashing** na priradenie URL k agentom, čím sa minimalizuje prekrývanie.

---

## 7. Reinforcement Learning (RL) pre Navigáciu a Získavanie Dát

RL sa stáva kľúčovým pre tréning LLM Agentov, aby si osvojili adaptívne správanie a stratégie v dynamických prostrediach.

### A. Účel RL a Architektúry

*   **Úloha:** RL je nevyhnutný na to, aby sa LLM posunuli od statického fine-tuningu (SFT) k učeniu z **dynamickej, interaktívnej spätnej väzby**.
*   **Prostredie:** **DeepResearcher** aplikuje RL tréning **v reálnom webovom prostredí**, čím prekonáva obmedzenia RAG prostredí, ktoré sú statické a predpokladajú existenciu všetkých potrebných znalostí v pevnej korpusovej báze.
*   **Tréning:** RL tréning je často nasadený po **SFT (Supervised Fine-Tuning)**. Pri komplexných úlohách navigácie sa ukázalo, že **Rejection Sampling Fine-Tuning (RFT)** je **nevyhnutný pre "cold start"** (počiatočný stav), pretože RL odmeny sú spočiatku extrémne riedke (sparse).

### B. Akcie, Stavy a Odmeny

| Komponent | Popis a Podpora zo Zdrojov | Feasibility pre MT |
| :--- | :--- | :--- |
| **Stav (State)** | V RL architektúrach pre navigáciu sa LLM agenti trénujú na základe histórie akcií, usudzovania a aktuálneho kontextu (napr. **ReAct** - Thought-Action-Observation). V **DeepResearcher** sa používa **short-term memory repository** pre každú otázku. | **Weakly Supported:** Exaktná definícia RL stavu je v zdrojoch pre LLM agentov nejasná, ale je implicitná v spracovaní **kontextového okna a histórie**. |
| **Akcia (Action)** | LLM Agent generuje akcie pomocou **Tool Calls**. Príklady akcií: **`web search`** (vygenerovanie dotazov), **`web browse`** (spracovanie URL). V navigačných agentoch (Glider) sú akcie **sekvencie na UI**. | **High:** LLM-založené akcie sú jasne definované ako **JSON požiadavky** na volanie nástrojov. |
| **Odmena (Reward)** | LLM Agenti sú trénovaní len s **outcome rewards** (odmena na základe finálneho výsledku). Metrika **F1 Score** sa používa ako **reward** pre tréning DeepResearcher. RL sa používa na **maximalizáciu správnosti finálnej odpovede**. DPO (DPO De-biasing) využíva preferenčné páry (y+, y-) na posilnenie učenia v marginálnych prípadoch. | **Medium:** Odmeny sú založené na jednoduchých, ale kritických metrikách, ako je **F1 správnosť finálnej odpovede**. |

### C. Použité RL Algoritmy a Hodnotenie

*   **Algoritmy:** Používajú sa rôzne varianty PPO (Proximal Policy Optimization) a DPO (Direct Preference Optimization). **DeepResearcher** používa distribuovaný RL (napr. **GRPO**) na prekonanie problémov s latenciou a anti-crawling mechanizmami v reálnom webovom prostredí.
*   **Rejection Sampling Fine-Tuning (RFT):** Predstavuje metódu tréningu, ktorá generuje a filtruje trajektórie. **Len trajektórie, ktoré vyvrcholia správnou finálnou odpoveďou, sú retenované**.

---

## Zhrnutie a Návrh Akčných Krokov pre Diplomovú Prácu

Váš zámer je plne podporovaný modernými výskumnými prúdmi. Pre magisterskú prácu je kľúčové zamerať sa na implementovateľné sub-komponenty a validáciu.

### 1. Extrakcia Potenciálnych Výskumných Problémov

1.  **Validácia a Prioritizácia Kvality:** Ako najefektívnejšie destilovať LLM-generované **labely kvality** (LMDS/Neural Quality Estimation) do malého, efektívneho modelu pre **škálovateľné filtrovanie/prioritizáciu** na úkor tradičných metód (napr. PageRank, NER).
2.  **Architektúra Agenta a Pamäť:** Implementácia a experimentálne hodnotenie **Multi-Agent** architektúry (napr. inšpirovanej **DeepResearcher** alebo **ResearchAgent**) zameranej na riadenie **kontextovej pamäte** a dynamické preplánovanie pri dlhých vyhľadávacích úlohách.
3.  **RL pre Validáciu a Fine-tuning:** Využitie **Rejection Sampling Fine-Tuning (RFT)** a/alebo **LLM-as-a-Judge** na generovanie **vysokokvalitných tréningových dát (trajektórií)** pre LLM Agenta, s cieľom zlepšiť jeho schopnosť usudzovania a používania nástrojov, čo je kľúčové pre **Retrieval-Augmented Generation (RAG)**.

### 2. Odporúčané Akčné Kroky (Feasibility)

Pre maximalizáciu šancí na úspešnú implementáciu a robustnú evaluáciu v rámci magisterskej práce navrhujem zamerať sa na:

1.  **Ohraničenie Zamerania (Scoping):** **Vyberte si jedného Agenta a jednu Metódu validácie/RL:** Napríklad, implementácia pipeliny **LMDS** (Knowledge Distillation pre quality scoring) by bola výpočtovo menej náročná než trénovanie end-to-end RL agenta.
2.  **Implementácia Scoringu a Prioritizácie:** Vyberte si existujúci LLM (napr. Qwen, Llama) na **zero-shot labelling** dokumentov (ako $LM_{large}$ v LMDS) s cieľom stanoviť **baseline sémantickej kvality** (Neural Prioritisation).
3.  **Definovanie Metrík:** Jasne definujte, ktoré **LLM-špecifické metriky** (napr. **Harvest Rate, maxNDCG, LLM-as-a-Judge Accuracy**) budete používať na hodnotenie výkonu Vášho *LLM-prioritizovaného* korpusu v porovnaní s tradičným BFS/DFS.
4.  **Literatúra k Reviewing Agents:** Dôkladne preštudujte implementáciu **ReviewingAgents** ako **LLM-as-a-Judge** frameworku, ktorý slúži na **validáciu kvality syntetizovaných výsledkov** pri RAG a integruje "kritické myslenie" do architektúry.
---ENDanalysis---

# Version 1 created by Google Gemini 3 Pro
---STARTv1---
RESEARCH TARGET (Výskumný zámer)Student Name: [Your Name]Supervisor: [Supervisor Name]Thesis Title: Multi-agent, LLM-assisted Semantic Web Crawler using Reinforcement Learning and LLM-based ValidationTarget Length: ~7 Pages A41. DIPLOMA PROJECT ASSIGNMENT(Note: This page is typically printed directly from the university system - e.g., YonBan/AIS. In this digital draft, just acknowledge its place.)2. MOTIVATION & PROBLEM STATEMENTProblem DefinitionTraditional web crawlers primarily rely on statistical metrics such as PageRank, indegree, or Click-Through Rate (CTR) to prioritize content. While effective for global popularity, these methods lack deep semantic understanding. They often struggle to identify "high-quality" or "educational" content needed for specialized domains or modern RAG (Retrieval-Augmented Generation) systems, leading to the retrieval of irrelevant or low-quality documents.RelevanceWith the shift towards natural language queries and the rise of Large Language Models (LLMs), there is a critical need for neural prioritization and semantic validation in crawling.Why now: Integrating LLMs as "judges" or "validators" allows for capturing query intent and document quality (e.g., factual correctness, educational value) far better than keyword matching.Impact: An effective LLM-assisted crawler can significantly improve the quality of datasets used for LLM pre-training and RAG systems, reducing hallucinations and improving context relevance.3. STATE OF THE ART ANALYSIS (Related Work)A. Navigation Strategies: From Statistical to NeuralTraditional Focused Crawling: Existing systems like WEB-SAILOR use domain-specific strategies (DSets) and parallel crawling, but rely on statistical relevance. Semantic Crawlers have introduced Named Entity Recognition (NER) to filter based on entity sense, but still lack full contextual reasoning [Ref: 1406.5690, jcssp.2015].LLM-Assisted Prioritization: Modern approaches utilize Neural Quality Estimators ($Q_{\theta}$) to estimate the semantic quality of a page before downloading it. Heuristics such as "quality neighborhood" (high-quality pages link to high-quality pages) are used to approximate URL value [Ref: 2506.16146].LLM-Guided Data Selection (LMDS): Strong models ($LM_{large}$) are used to label data for educational value, distilling this knowledge into smaller models ($LM_{small}$) to filter massive corpora like RPJ-CC efficiently [Ref: 2406.04638].B. Scoring and Validation of Document QualityEmbedding-based Scoring: In domain-specific crawling (e.g., CTI), relevance is determined by the vector distance between document embeddings (BERT, S-BERT) and a ground truth vector.LLM-as-a-Judge: Validation is evolving from simple classification to LLM-based reasoning, where models assess if a document is "engaging" or "educational".Rejection Sampling Fine-Tuning (RFT): For training agents, entire trajectories (reasoning + actions) are validated. If a trajectory does not lead to the correct answer, it is rejected, providing a strong supervision signal for RL [Ref: 2507.02592].C. Multi-Agent Architectures & Reinforcement LearningDeepResearcher: Utilizes a specialized multi-agent architecture where "Browsing Agents" sequentially process web segments. It employs Reinforcement Learning (RL) to train agents in dynamic environments, overcoming the limitations of static datasets [Ref: 2509.13309].ResearchAgent: Implements an iterative "Reviewing Agent" system where LLMs act as critics (simulating peer review) to refine generated ideas. This "Critic" role is essential for validating crawler outputs [Ref: ResearchAgent].RL Algorithms: Systems typically use PPO (Proximal Policy Optimization) or DPO (Direct Preference Optimization). RL is crucial for the "cold start" problem in navigation, moving agents from Supervised Fine-Tuning (SFT) to dynamic adaptability.Synthesis & Gap AnalysisCurrent solutions are often fragmented: they are either efficient but semantically shallow (traditional crawlers) or semantically deep but computationally expensive and static (RAG). There is a research gap in effectively combining lightweight Multi-Agent architectures with RL-based validation to create a crawler that is both efficient (Harvest Rate) and semantically accurate (LLM-Judge Score) without prohibitive computational costs.4. PROPOSED SOLUTION & EVALUATIONResearch QuestionsRQ1: How can LLM-generated quality labels be effectively distilled into a lightweight scoring model to prioritize crawler navigation better than statistical baselines?RQ2: Can a Multi-Agent architecture with specialized "Critic" agents (LLM-as-a-Judge) improve the Harvest Rate of relevant documents compared to single-agent approaches?RQ3: To what extent does Reinforcement Learning (specifically Rejection Sampling Fine-Tuning) improve the agent's ability to navigate dynamic web environments compared to static prompting?Implementation PlanThe proposed system will be a Multi-agent, LLM-assisted crawler.Agent Architecture: Implementation of a "Browsing Agent" for navigation and a "Critic Agent" for validation.Scoring Module: Development of a Neural Quality Estimator using Knowledge Distillation (distilling $LM_{large}$ insights into a smaller efficiency model).RL Optimization: Applying Rejection Sampling Fine-Tuning (RFT) to train the navigation agent using trajectories validated by the Critic.Evaluation StrategyDatasets: Standard web crawling benchmarks (e.g., ClueWeb part) or domain-specific datasets (e.g., scientific papers, CTI reports).Metrics:Harvest Rate (HR): Ratio of relevant pages found to total pages crawled ($|Relevant| / |Crawled|$).MaxNDCG: To measure ranking efficiency against an ideal upper bound.LLM-as-a-Judge Accuracy: Using a strong LLM to evaluate the factual correctness and relevance of retrieved content (Human-aligned metric).Baselines: Comparison against a traditional BFS (Breadth-First Search) crawler and a standard Focused Crawler (e.g., similarity-based).Work Plan (3 Semesters)Semester 1 (DP1): Literature review of Neural Crawling and RL agents. Specification of the Multi-Agent architecture.Semester 2 (DP2): Implementation of the core Crawler Agents and the LLM-based Scoring module. Baseline testing.Semester 3 (DP3): Integration of RL (RFT) for optimization. Final evaluation using Harvest Rate and MaxNDCG. Thesis writing.5. REFERENCES[Author]. Focused Crawling and WEB-SAILOR architectures. [Ref: 1406.5690v1, jcssp.2015.120.126].[Author]. Semantic Crawling based on Named Entity Recognition. [Ref: Semantic_crawling_NER].[Author]. Neural Prioritisation for Web Crawling. [Ref: 2506.16146v2].[Author]. LLM-Guided Data Selection (LMDS). [Ref: 2406.04638v1].[Author]. Neural Quality Estimation. [Ref: 2407.12170v1].[Author]. DeepResearcher: RL for Web Navigation. [Ref: 2509.13309v2].[Author]. Rejection Sampling Fine-Tuning (RFT) for Agents. [Ref: 2507.02592v1].[Author]. Evaluation Metrics: Harvest Rate and MaxNDCG. [Ref: 3637528.3671620].6. DIPLOMA PROJECT 1 ASSIGNMENT TEXT (Required Format)Project Title: Multi-agent LLM-assisted Semantic Web CrawlerAssignment Text:[Introduction]:Traditional web crawlers relying on statistical metrics often fail to capture the semantic nuance required for modern information retrieval. The integration of Large Language Models (LLMs) and Reinforcement Learning (RL) offers a path to create agents capable of understanding query intent and validating document quality dynamically.[Analyze]:"Analyze existing solutions, frameworks, and methods regarding LLM-assisted crawling and Multi-Agent systems, specifically focusing on Neural Prioritization, LLM-based validation (LLM-as-a-Judge), and Reinforcement Learning strategies for navigation."[Implement]:"Implement the solution for targeted semantic retrieval using a Multi-Agent architecture with LLM-based quality scoring that addresses the identified limitations of statistical crawlers."[Evaluate]:"Evaluate the proposed system on web crawling benchmarks using metrics such as Harvest Rate, MaxNDCG, and LLM-as-a-Judge accuracy."[Discussion]:"Discuss the evaluation results, limitations of the approach, potential improvements, and broader applications beyond academic review."Supporting Sources:DeepResearcher: RL for Web Navigation (Ref: 2509.13309v2)LLM-Guided Data Selection (LMDS) (Ref: 2406.04638v1)Neural Prioritisation for Web Crawling (Ref: 2506.16146v2)

---ENDv1---

# Version 2 created by chat GPT 5.1 Thinking. 
---STARTv2---
Working title (draft)

Multi-Agent LLM-Assisted Semantic Crawler with RL-Guided Navigation, Validation and Scoring for RAG Systems

⸻

1. Motivation and background

Modern information retrieval is shifting from keyword search to natural-language, LLM-centric workflows. Classical web crawlers and ranking pipelines (PageRank, indegree, CTR, lexical relevance) were designed for traditional IR and are increasingly misaligned with:
	•	Semantic queries phrased in natural language.
	•	LLM-based consumers (RAG systems, autonomous “deep research” agents).
	•	The need for fresh, high-quality, domain-specific corpora.

Recent work on neural prioritisation for web crawling shows that crawling policies guided by semantic quality estimators can improve harvest rate and maxNDCG in early-stage crawling, compared to traditional link- or popularity-based policies.  ￼ Parallel and distributed crawlers such as WEB-SAILOR, WebParF and BUbiNG provide scalable architectures for large-scale crawling, but their prioritisation logic remains mostly non-semantic or heuristic.  ￼

In parallel, LLM-based methods for document selection, pruning and quality estimation (e.g., LMDS and neural passage quality estimation) show that an LLM can act as a powerful document grader whose judgments can be distilled into smaller classifiers for scalable filtering and static pruning.  ￼ Focused crawlers such as ThreatCrawl already integrate BERT/SBERT-style classifiers into the crawling loop to adapt paths based on semantic relevance in specialised domains such as cyber-threat intelligence.  ￼

At the same time, deep research agents such as DeepResearcher, WebSailor and WebResearcher train LLMs with reinforcement learning in real-world web environments. They rely on tool calls (search, browse), outcome-based rewards (e.g., F1 against ground truth), and sometimes LLM-as-a-Judge to evaluate long trajectories.  ￼ These systems show that:
	•	RL can improve navigation and reasoning in open-web settings beyond fixed-corpus RAG.
	•	Rejection Sampling Fine-Tuning (RFT) is helpful as a “cold start” to obtain good trajectories before outcome-based RL.
	•	LLM-as-a-Judge is increasingly used as an automatic evaluation signal.

Finally, there is a rich literature on incremental and parallel crawling and freshness metrics (freshness, age, crawl-hit rate) that focus on keeping repositories up to date under politeness constraints, often using change-detection and scheduling strategies.  ￼

The proposed thesis sits at the intersection of these lines of work:
	•	Semantic / LLM-aware crawling (Neural Prioritisation, ThreatCrawl, semantic crawling with NER).  ￼
	•	LLM-based validation, scoring and data selection (LMDS, neural passage quality estimation, LLM-as-a-Judge).  ￼
	•	Multi-agent deep research architectures (ResearchAgent-style reviewers, DeepResearcher/WebSailor/WebResearcher).  ￼
	•	Incremental parallel crawling and freshness (incremental parallel web crawler, adaptive and selective crawlers).  ￼

⸻

2. Problem statement and research gap

Existing systems mostly treat these aspects separately:
	•	Crawlers focus on coverage, freshness and politeness, with limited semantic prioritisation.
	•	LLM-guided document selection focuses on offline corpus filtering rather than online crawling decisions.
	•	Deep research agents use search APIs and ad-hoc browsing, but do not expose a reusable crawling layer with explicit metrics like harvest rate or maxNDCG.
	•	Multi-agent patterns (researcher + reviewers) mostly operate on text and reasoning, not on the underlying document acquisition and ranking pipeline.

There is a gap for a system that:
	1.	Treats crawling, validation and ranking as one integrated loop, explicitly optimised for a downstream RAG or deep research use-case in a bounded domain.
	2.	Uses LLM-based reviewers both:
	•	as semantic filters and quality estimators for retrieved documents;
	•	as sources of reward signals (via LLM-as-a-Judge) for RL policies that control crawling and query refinement.
	3.	Exploits a multi-agent architecture where diverse researcher agents explore in parallel, and reviewer agents critique and re-route them, similar in spirit to ResearchAgent but grounded in concrete crawling and IR metrics.  ￼

The thesis will investigate whether such an architecture can measurably improve:
	•	Harvest rate and maxNDCG during early crawling.  ￼
	•	RAG answer quality and robustness for domain-specific queries.
	•	Data quality and freshness of the resulting corpus.

⸻

3. Research objectives and questions

3.1 Main objective

Design and evaluate a multi-agent, LLM-assisted semantic crawling system with RL-guided navigation, LLM-based validation and scoring, integrated as a retrieval layer for RAG or deep-research tasks on a specific domain.

3.2 Core research questions
	1.	RQ1 – LLM scoring vs. traditional scoring
How does LLM-based document quality and relevance scoring (inspired by LMDS and LLM-as-a-Judge) compare to traditional or neural-embedding-based scoring for focused crawling and RAG retrieval, in terms of harvest rate, nDCG/MAP, and downstream answer quality?  ￼
	2.	RQ2 – Multi-agent researcher + reviewer architecture
Can a multi-agent setup with multiple researcher agents and reviewer agents (LLM critics) discover more diverse and higher-quality documents than a single crawler or single agent, under the same resource budget? How does inter-agent feedback (cross-checking, re-ranking, sharing discovered hubs) influence performance?  ￼
	3.	RQ3 – RL-guided navigation and strategy adaptation
Does applying RL over crawling actions and query refinement (link selection, source selection, depth vs breadth) improve early harvest rate, maxNDCG and RAG performance, compared to hand-designed heuristics and static policies?  ￼
	4.	RQ4 – Evaluation and reliability of LLM-based validation
How reliable are LLM-as-a-Judge signals (relevance, factuality, recency, usefulness) when used both for ranking and as RL rewards, compared to human annotations and traditional metrics? Under what conditions does LLM judgement introduce bias or instability?  ￼

⸻

4. Conceptual system overview

At a high level, the proposed system consists of:
	•	A Coordinator that manages goals, agent allocation, and RL policies.
	•	Several Research Agents that:
	•	issue search queries or follow links;
	•	fetch candidate documents/pages from a bounded domain (e.g., a subset of open-web sources, arXiv categories, or CTI websites).
	•	Reviewer Agents (LLM-based) that:
	•	evaluate each document for relevance, quality, recency, and task usefulness;
	•	provide structured feedback and scores;
	•	act as LLM-as-a-Judge for RL rewards.
	•	A Shared Index / Corpus that stores scored documents and embeddings for downstream RAG.
	•	A RAG Component that answers user queries using the scored corpus.

flowchart LR
  Q[User task / query] --> C[Coordinator & RL policy]

  subgraph Researchers
    R1[Research Agent 1\n(strategy A)]
    R2[Research Agent 2\n(strategy B)]
    R3[Research Agent 3\n(strategy C)]
  end

  C --> R1
  C --> R2
  C --> R3

  subgraph Corpus[Web / Domain Corpus]
    D[Pages, APIs, Docs]
  end

  R1 -->|crawl/search| D
  R2 -->|crawl/search| D
  R3 -->|crawl/search| D

  R1 -->|candidate docs| V1[Reviewer Agent 1\n(LLM-as-Judge)]
  R2 -->|candidate docs| V2[Reviewer Agent 2\n(LLM-as-Judge)]
  R3 -->|candidate docs| V3[Reviewer Agent 3\n(LLM-as-Judge)]

  V1 -->|scores + feedback| C
  V2 -->|scores + feedback| C
  V3 -->|scores + feedback| C

  V1 -->|validated docs| IDX[(Scored Index)]
  V2 -->|validated docs| IDX
  V3 -->|validated docs| IDX

  IDX --> RAG[RAG / QA LLM]
  Q --> RAG

The RL policy in the Coordinator uses reviewer scores and RAG performance as reward signals to update strategies for:
	•	Link prioritisation and domain/source selection.
	•	Query reformulation and exploration vs exploitation.
	•	Allocation of crawl budget across agents.

⸻

5. Methodology (first-version plan)

5.1 Domain and data
	•	Choose a bounded domain suitable for a thesis-scale project, for example:
	•	a subset of arXiv categories,
	•	cyber-threat intelligence websites (as in ThreatCrawl), or
	•	a focused academic/technical topic.
	•	Implement a baseline crawler leveraging existing tools (e.g., BUbiNG or a simpler Python-based crawler), with:
	•	polite fetching,
	•	constraints on depth and bandwidth,
	•	logging suitable for HR, freshness, and ranking metrics.  ￼

5.2 LLM-based validation and scoring
	•	Implement a prompted LLM reviewer, inspired by LMDS and LLM-as-a-Judge, that for each document returns:
	•	binary or multi-grade labels for quality, relevance, and educational value;  ￼
	•	explanations to aid analysis.
	•	Distill these labels into a smaller classifier (as in LMDS) to enable scalable scoring during crawling.
	•	Compare this with:
	•	embedding-based similarity (e.g., SBERT/ThreatCrawl-style scoring),  ￼
	•	neural passage quality estimation for static pruning.  ￼

5.3 Multi-agent architecture
	•	Implement at least two–three researcher agents with different strategies:
	•	different query templates,
	•	different risk/exploration levels,
	•	potentially different seed sets.
	•	Implement reviewer agents as separate LLM instances or roles, following ideas from ResearchAgent/DeepResearcher (critic patterns, structured critiques).  ￼
	•	Experiment with:
	•	independent agents (no sharing),
	•	cooperative agents that share discovered high-score hubs or negative examples.

5.4 RL and adaptation
	•	Start with a hand-crafted policy (no RL) to establish baselines.
	•	Introduce a simple RL formulation:
	•	State: recent query + embedding summary of discovered docs + basic statistics (HR so far, diversity).
	•	Actions: select next link/domain, choose agent strategy A/B/C, or adjust search parameters.
	•	Reward: mixture of reviewer scores (LLM-as-a-Judge), improvements in HR/maxNDCG, and RAG answer quality on held-out queries.  ￼
	•	Use RFT-style filtering of trajectories when training policies, to avoid noisy or degenerate behaviours.  ￼

5.5 Evaluation
	•	Crawling & IR metrics:
	•	Harvest Rate (HR),
	•	nDCG@k, MAP and MRR on a labelled subset,
	•	freshness metrics if incremental crawling is used.  ￼
	•	RAG / QA metrics:
	•	automatic metrics (F1, BLEU-1),
	•	LLM-as-a-Judge evaluation of answers (correctness, relevance, faithfulness).  ￼
	•	Ablation studies:
	•	single vs multi-agent,
	•	with vs without LLM-based scoring,
	•	with vs without RL adaptation.

⸻

6. Scope and feasibility (for a master’s thesis)

To keep the work feasible:
	•	Use existing LLM APIs or open-source models; focus on smart prompting, distillation, and evaluation rather than training large models from scratch.
	•	Limit the domain and web surface to a manageable scale (e.g., a few hundred thousand pages).
	•	Implement one main RL configuration rather than exploring many algorithmic variants.
	•	Prioritise clear, reproducible metrics and comparison to strong baselines (ThreatCrawl-style focused crawling, LMDS-like static filtering, non-agentic RAG).  ￼

⸻

7. Expected contributions (first draft)
	1.	Design and prototype of a multi-agent, LLM-assisted semantic crawling architecture with integrated reviewer agents and RL-guided navigation.
	2.	Empirical comparison of LLM-based document scoring (LMDS-style, LLM-as-a-Judge) against neural quality estimators, embedding similarity, and traditional IR baselines within a crawling pipeline.
	3.	Experimental evidence on the impact of multi-agent exploration and RL adaptation on harvest rate, nDCG/MAP, freshness, and downstream RAG QA quality.
	4.	A reusable experimental framework (code + dataset splits + evaluation scripts) that can be extended in future work on deep research agents and RAG-aware crawling.

⸻

8. References (initial, non-exhaustive)

(Ordering here is thematic rather than strict citation style; final thesis will use a consistent formal style.)
	•	Pezzuti, F., MacAvaney, S., & Tonellotto, N. (2025). Neural Prioritisation for Web Crawling. arXiv:2506.16146.  ￼
	•	Kong, X., Gunter, T., & Pang, R. (2024). Large Language Model-guided Document Selection. arXiv:2406.04638.  ￼
	•	Chang, X. et al. (2024). Neural Passage Quality Estimation for Static Pruning. arXiv:2407.12170.  ￼
	•	Kuehn, P., Nadermahmoodi, D., Bayer, M., & Reuter, C. (2023). ThreatCrawl: A BERT-based Focused Crawler for the Cybersecurity Domain. arXiv:2304.11960.  ￼
	•	Di Pietro, G., Aliprandi, C., De Luca, A. E., Raffaelli, M., & Soru, T. (2014). Semantic Crawling: An Approach Based on Named Entity Recognition. ASONAM 2014.  ￼
	•	Gupta, S., Bhatia, K. K., & Manchanda, P. (2014). WebParF: A Web Partitioning Framework for Parallel Crawlers. arXiv:1406.5690.  ￼
	•	Mukhopadhyay, D. et al. (2011). Architecture of A Scalable Dynamic Parallel WebCrawler with High Speed Downloadable Capability for a Web Search Engine (WEB-SAILOR). arXiv:1102.0676.  ￼
	•	Boldi, P., Marino, A., Santini, M., & Vigna, S. (2014). BUbiNG: Massive Crawling for the Masses. WWW Companion.  ￼
	•	Yadav, D., Sharma, A. K., Sanchez-Cuadrado, S., & Morato, J. (2012). An Approach to Design Incremental Parallel WebCrawler. Journal of Theoretical and Applied Information Technology.  ￼
	•	Olston, C., & Najork, M. (2010). Web Crawling. Foundations and Trends in IR.  ￼
	•	Zheng, Y. et al. (2025). DeepResearcher: Scaling Deep Research via Reinforcement Learning in Real-World Environments. arXiv:2504.03160.  ￼
	•	Li, K. et al. (2025). WebSailor: Navigating Super-human Reasoning for Web Agent. arXiv:2507.02592.  ￼
	•	Qiao, Z. et al. (2025). WebResearcher: Unleashing Unbounded Reasoning Capability in Long-Horizon Agents. arXiv:2509.13309.  ￼
	•	Du, M. et al. (2025). DeepResearchBench: A Comprehensive Benchmark for Deep Research Agents.  ￼
	•	Fang, T. et al. (2025). Enhancing Web Agent Self-Improvement with Co-evolving Data and Curriculum. EMNLP 2025.  ￼
	•	Additional surveys and focused crawling papers (Novak, Ester, etc.) will be included as the literature review is expanded.  ￼

⸻

This is a first-version “canvas” for your research target document. Next iterations can deepen:
	•	the precise RL formulation (state/action/reward),
	•	the concrete domain choice and data pipeline, and
	•	the exact evaluation protocol and baselines you will commit to.

---ENDv2---