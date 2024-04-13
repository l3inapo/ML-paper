# Advancing TTP Analysis: Harnessing the Power of Encoder-Only and Decoder-Only Language Models with Retrieval Augmented Generation

https://arxiv.org/abs/2401.00280

# Abstract

Tactics, Techniques, and Procedures（TTPs）概述了攻擊者利用漏洞的方法。在MITRE ATT＆CK框架中解釋TTPs對於資安從業人員來說可能具有挑戰性，原因在於假定的專業知識、複雜的依賴關係和固有的模糊性。與此同時，大型語言模型（LLMs）的進展導致最近對其在資安操作中的應用進行了大量研究。這引出了一個問題，即使用encoder-only（例如RoBERTa）和使用decoder-only（例如GPT-3.5）的LLMs能否理解和總結TTPs，以便通知分析人員有關攻擊程序的預期目的（即戰術）。最先進的LLMs已被證明容易出現幻覺，提供不准確的信息，這在資安等關鍵領域中是有問題的。因此，我們提出使用檢索增強生成（RAG）技術來提取每個攻擊程序的相關上下文，用於使用decoder-only的LLMs（無需微調）。我們進一步將此方法與僅使用encoder-only的LLMs的supervised fine-tuning（SFT）進行對比。我們的結果顯示，使用decoder-only的LLMs（即其預先訓練的知識）和使用encoder-only的LLMs的SFT都提供了對攻擊程序的不准確解釋。當找到直接相關的上下文時，使用RAG的decoder-only的LLMs顯示出顯著的改善。本研究進一步闡明了使用RAG來解釋TTPs的LLMs的限制和能力。

# Introduction

資安漏洞的不斷演變帶來了對事件和威脅分析的重大挑戰。資安威脅情報（CTI）報告致力於收集、分析和解釋有關潛在資安攻擊的數據。CTI旨在了解並預測資安對手的戰術、技術和程序（TTPs）。解讀和理解各種資安攻擊程序（在MITRE ATT＆CK等參考模型中不斷更新）的過程需要豐富的專業知識和努力。ATT＆CK框架在此方面充當了關鍵工具，提供了對攻擊者行為和目標系統技術的深入洞察。ATT＆CK框架利用3/4層次的Tactic-Technique/Sub-technique-Procedure結構來解釋攻擊者利用漏洞的**方法**和**原因**。然而，這些描述的複雜性和潛在的模糊性可能使其難以解釋，從而導致安全分析師對相同描述得出不同的結論。例如，考慮以下攻擊程序描述：

Threat Group-3390 has performed DLL search order hijacking to execute their payload.

ATT＆CK框架將此程序與三個戰術關聯起來，即Privilege-Escalation’, ‘DefenseEvasion’, 跟‘Persistence’。從描述中並不明顯DLL Search Order Hijacking 術將導致任何三者，更不用說全部三者的對立意圖。事實上，有人可能會認為此描述更符合“‘Execution’”戰術，因為對手試圖執行malicious payload。因此，我們問：LLMs能否解釋並將procedure descriptions對應到特定的ATT＆CK tactics？請注意，我們認識到ATT＆CK TTP映射的固有模糊性。事實上，這正是我們研究LLMs在這種現實世界不完美情況下的能力和限制的動機所在。

大型語言模型（LLMs）的發展，例如OpenAI的GPT-3.5模型，已在NLP任務中取得了顯著進展。這一進展主要是由於LLM的增強語義理解和可擴展性。還有一些工作是通過利用LLMs來處理TTPs。然而，這些工作中沒有一個是比較使用encoder-only的LLMs（例如RoBERTa）和decoder-only的LLMs（例如GPT-3.5），同時利用檢索增強生成（RAG）技術來分析攻擊程序描述的。

> encoder-only的LLMs擅長於通過處理輸入數據生成上下文表示來進行序列分類(sequence-classification)和實體識別(entity recognition)等任務。
> 

> 而decoder-only的模型擅長生成連貫的、具有上下文相關性的文本，使其成為語言生成任務的理想選擇。
> 

LLMs的一個主要問題是幻覺的趨勢。這個問題涉及模型會產生誤導性或完全捏造的回應，但表現得很自信。為解決這個問題，提出了透過RAG技術來提取最相關的文件，並將其作為額外信息餵入LLMs的輸入中。它用於通過首先檢索可能包含答案的文件，把模型的注意力轉移到相關信息上，然後使用此信息中來生成回應。這種方法是有益的，因為它不僅僅局限於模型的temporal knowledge，而且會從已選定的資源中做為基礎來當作引用回應，這對於减少幻覺方面有很大的作用。

認識到encoder-only和decoder-only的LLMs在資安操作中可能有不同的用途和好處，本研究旨在比較這兩者在解釋資安攻擊程序並將其映射到ATT＆CK戰術方面的有效性。

作者會檢查三組方法：

1. 直接使用decoder-only的LLMs（GPT-3.5-Turbo）、
2. 使用ATT&CK描述進行監督微調（SFT）的encoder-only LLMs（RoBERTa和SecureBERT）
3. 在decoder-only的LLMs上使用RAG。

請注意，在這三種方法中，我們都避免明確訓練LLMs把 procedure descriptions map到MITRE ATT＆CK tactics 上。

這意味著我們在測試LLMs的解釋能力，而不是建立一個傳統的分類模型。從某種程度上說，我們希望模擬LLMs在網路上不存在procedure description的情況下的表現。據我們所知，這是第一個正式研究比較這些方法來解釋資安攻擊程序描述的研究。

通過使用從MITRE ATT＆CK（截至2023年10月31日的發布版本v14.1）中提取的最新TTP描述進行實驗，我們提出了兩種LLM類型的優缺點，並提出了未來的研究方向。主要貢獻如下：

- 分析並比較了smaller encoder-only LLMs的SFT與RAG-enhanced的larger decoder-onlyLLMs在解釋TTPs方面的性能。
- 當提供了正確的訊息片段（通過RAG）時，顯示了對於decoder-only LLMs解釋TTPs的顯著提升。
- 我們研究了提供procedure descriptions相關上下文的不同檢索方法，並展示了當RAG技術無法檢索到相關信息時的限制。
- 我們證明了雖然decoder-only LLMs保持較強的“’recall’”，但缺乏“’precision’”，詳細說明了如果要增加LLMs的“precision”而不影響“召回率”的新進展中的需求。
- 通過提供具體的例子，我們展示了使用decoder-only LLMs與RAG的當前能力，並提供了關於它們缺乏“可解釋性”的見解。

# Related Works

## Large Language Models

大型語言模型（LLMs）的進步主要歸功於Transformer架構的採用（Vaswani等人，2017年）。這個架構的核心是encoder和decoder，在各種LLMs中被獨特地利用。

### Encoder-Only

其中一個開創性的引入是

1. BERT（Devlin等人，2019年），這是一個基於Transformer的pre-trained encoder-only LLM。它利用Transformer編碼器來實現對語言上下文和細微差異的深入理解。BERT的重要性在於其雙向訓練，這使得它能夠全面理解語言結構。
2. RoBERTa（Liu等人，2019年）擴展了BERT的功能。它優化了hyperparameters，消除了下一句預測任務，並通過更大的mini-batches和longer sequences增強了訓練。
3. SecureBERT（Aghaei等人，2022年），它基於RoBERTa，但是專門針對資安數據進行了微調。

### Decoder-Only Models

LLM的格局進一步被Decoder-Only Models所改變，其特點是具有龐大的訓練數據集和重要的計算資源。OpenAI的GPT-3.5和Meta的LLAMA是這一類中的代表，兩者都包含數十億個參數。這些模型不僅表現出了規模化的能力，還表現出了生成類似人類文本並識別各個領域中複雜模式的能力。這些模型在處理多樣化和複雜任務方面的有效性和多功能性在文獻中有詳細記錄，如（Min等人，2023年；Zhao等人，2023年）所描述。然而，人們仍然對這些模型在解釋TTPs方面的能力存有疑問。

## LLM for TTP interpretations

在處理TTP描述的背景下，已經有一些工作提出了使用自然語言處理技術的方法（Sauerwein和Pfohl，2022年；Husari等人，2017年；Kim，Kim等人，2022年）。

最近，以下工作利用了encoderonly LLMs（例如BERT）進行TTP分類。

1. You等人（2022年）介紹了TIM框架，使用continuous description sets和SentenceBERT embedding，以及TTP特定features，如IP和URL，根據ATT＆CK框架的五種techniques和一種tactic來分類TTPs。
2. Alves，Filho和Gonc ̧alves（2022年）測試了不同超參數的各種BERT模型變體，以找到最適合TTP分類的模型。
3. Rani等人（2023年）開發了TTPHunter，這是一個使用BERT和RoBERTa模型以及linear classifier從非結構化文本中提取TTPs的工具。
4. Orbinato等人（2022年）進行了一項實驗研究，比較了傳統ML和DL模型（例如SVM vs. SecureBERT）在從資安威脅情報文本中映射TTPs的效果。作者發現SecureBERT是最有效的，但他們沒有提供SecureBERT和其基礎RoBERTa模型之間的比較。
5. 作者之前的工作（Fayyazi和Yang，2023年）比較了encoder-only與直接使用decoder-only LLMs來處理和分類更高層次的tactic and technique descriptions。encoder-only的實驗專注於傳統的分類器訓練，其中訓練集和測試集來自相同的數據集。decoder-only方法沒有與RAG整合在一起。

這片論文主要將之前的發現擴展到處理更難解釋的攻擊程序描述，並使用更先進的LLM方法，而無需將測試數據明確對應到tactic labels。

## Retrieval Augmented Generation (RAG)

儘管最先進的語言模型在instruction-following tasks（例如，問題回答）方面具有強大的能力，但這些模型已被證明在生成的回應中容易出現幻覺。它們對預訓練知識的依賴（即受限於訓練數據的時間範圍）存在限制，特別是在精確性至關重要的領域，例如在電腦安全領域。使用即時數據進行微調可能成本高昂，而且仍然容易出現幻覺。因此，為了適應新信息並提供事實知識，人們引入了檢索增強生成（RAG）（Borgeaud et al. 2022）。

在RAG中，相關的信息片段存儲在vector database中。當提出查詢時，RAG從該數據庫中選擇最相關的數據，然後將其合併到語言模型的prompt中，然後再生成回應。這種方法提供了兩個關鍵優勢:

1. 允許語言模型整合當前信息，且不用一直微調
2. 通過提供有背景依據的基礎信息，增強了回應的清晰度和相關性。

基本上，RAG使LLM能夠專注於特定查詢的特定上下文，從而改善了回應的清晰度和準確性，並將模型的注意力導向檢索到的最相關上下文。已有研究在自然語言處理任務中使用了RAG（Borgeaud et al. 2022；Mallen et al. 2023；Ram et al. 2023；Al Ghadban et al. 2023）。

在本研究中，我們提出使用RAG來向語言模型提供有關攻擊程序的更多信息，並將語言模型的注意力集中在通過RAG檢索到的文本上。

# Methodology & Experimental Design

作者設計了這個研究來比較以下幾點：

1. 對encoder-only LLMs進行監督微調（SFT），使用帶有標記的ATT＆CK技術/子技術描述，
2. 直接使用僅解碼器LLMs（即預先訓練的知識）
3. 使用RAG來進行decoder-only LLMs，通過找到最相似的 attack procedures來檢索出相關的URL。

此外，考慮一種理想（但不切實際）的情況，即對於RAG，使用每個程序的確切URL以提供最佳性能。此實驗設計使我們能夠評估LLMs解釋資安攻擊程序描述並將其映射到相應戰術的能力。由於存在多個原因，這是一個具有挑戰性的過程：

1. 沒有公開可用的數據集將程序描述映射到ATT＆CK戰術
2. 每個程序可以映射到LLMs有多個ATT＆CK戰術需要考慮要對到哪個
3. LLMs需要一個表達很完整的prompt來生成所需的輸出
4. RAG需要檢索最相關的信息。

接下來討論我們的方法和實驗設計來應對這些挑戰。

## Datasets

為了這個實驗，作者從MITRE ATT＆CK框架中收集了數據。之所以選擇ATT＆CK，是因為它已經在行業SIEM tool中廣泛應用，並提供了詳細的資安描述。我們匯總了企業戰術、技術和子技術的描述，以及它們與相應戰術的映射，用於對encoder-only模型進行微調。作者總共從MITRE ATT＆CK框架獲得了639個描述。一些描述有2、3或甚至4種不同的戰術。值得注意的是，使用639個帶標籤的ATT＆CK描述來對encoder-only LLMs進行監督微調，反映了現實世界中可能只存在有限的帶標籤數據的情況。此外，為了測試encoder-only和decoder-only LLMs的性能，我們爬取了所有從ATT＆CK框架中描述攻擊方法的企業程序範例。作者刪除了程序描述中包含任何14個MITRE ATT＆CK戰術的描述，以防止LLMs出現潛在偏差或shortcuts。總共收集了9532個程序描述以及它們相應的URL（用於RAG）。值得注意的是，程序描述沒有用於encoder-only LLMs的監督微調。

[圖2](https://www.notion.so/Advancing-TTP-Analysis-Harnessing-the-Power-of-Encoder-Only-and-Decoder-Only-Language-Models-with-R-b01f70d0a08a40fc9705e0208e79004b?pvs=21)顯示了攻擊程序描述的戰術之間的成對重疊。可以看出，“持續性”、“特權升級”和“防禦逃避”戰術與其他戰術之間的重疊更高。我們將與我們的源代碼一起共享精選的數據，作為研究社區的工具。

![Untitled](Advancing%20TTP%20Analysis%20Harnessing%20the%20Power%20of%20Enc%20b01f70d0a08a40fc9705e0208e79004b/Untitled.png)

## Supervised Fine-Tuning of Encoder-Only LLMs

為了進行與encoder-only LLMs的監督微調，我們選擇了RoBERTa-base和SecureBERT。我們之所以選擇這些模型，是因為它們在處理TTPs方面已經表現出優於傳統ML和DL技術的優越性。SecureBERT模型已經使用廣泛的資安文本語料庫進行了微調。我們的目標是評估這些LLMs從資安描述中提煉關鍵信息並將其映射到相關ATT&CK戰術的效能。因此，我們使用ATT&CK的戰術、技術和子技術描述對它們進行微調，而不是對程序進行微調。值得注意的是，RoBERTa和SecureBERT模型都有512-token輸入限制的限制，因此，我們截斷了超出此限制的描述的末尾部分。

[圖1](https://www.notion.so/Advancing-TTP-Analysis-Harnessing-the-Power-of-Encoder-Only-and-Decoder-Only-Language-Models-with-R-b01f70d0a08a40fc9705e0208e79004b?pvs=21)顯示了微調RoBERTa和SecureBERT模型的整個過程。

![Untitled](Advancing%20TTP%20Analysis%20Harnessing%20the%20Power%20of%20Enc%20b01f70d0a08a40fc9705e0208e79004b/Untitled%201.png)

1. 首先，對ATT&CK描述進行標記化，以使其與RoBERTa和SecureBERT模型兼容。
2. 其次，我們添加了一個具有14個節點的分類層，每個節點對應一個ATT&CK戰術。為了應對多標籤分類的挑戰，我們在此層內採用了Sigmoid activation function。這個函數至關重要，因為它計算了每個描述對應的每個戰術的獨立概率。這使得模型能夠有效地辨別並適當地分配多個標籤。

[表1](https://www.notion.so/Advancing-TTP-Analysis-Harnessing-the-Power-of-Encoder-Only-and-Decoder-Only-Language-Models-with-R-b01f70d0a08a40fc9705e0208e79004b?pvs=21)顯示了微調模型所選擇的超參數。我們採用了binary Cross-entropy作為損失函數（因為它是二進制的性質，用於預測每個戰術），batch size為16，30個epoch，以及5e-5作為learning rate的最佳值。模型的輸出形式為一個14維的二進制向量，每個元素代表一個ATT&CK戰術，sigmoid輸出超過0.5表示對該特定戰術的預測。

![Untitled](Advancing%20TTP%20Analysis%20Harnessing%20the%20Power%20of%20Enc%20b01f70d0a08a40fc9705e0208e79004b/Untitled%202.png)

## Decoder-Only LLMs with and w/o RAG

我們考慮OpenAI的GPT-3.5-turbo-1106模型，它的上下文窗口為16K token，作為我們的decoder-only LLM。值得注意的是，在OpenAI模型中，它們固有的設計是非確定性的。這意味著相同的輸入可能會產生不同的輸出。因此，為了管理這一方面並確保一致性，我們將temperature參數設置為“0”，並設置一個seed number（1106）以確保可再現性。我們的baseline使用GPT-3.5-Turbo將不使用RAG。我們設計了一個簡單而乾淨的prompt，除了程序描述之外，不會過度影響LLM的attention，根據社群報告的最佳做法來嘗試，避免出現胡言亂語，以下是用於decoder-only baseline的prompt：

> 你是一名資安專家。知道<<procedure>>，一個攻擊者會使用這個技術實現哪些MITRE ATT＆CK tactics？請僅回答你確定的MITRE ATT＆CK戰術。
> 

我們設計了帶有RAG的decoder-only LLM，試圖模擬一個現實世界的情境，在這個情境中，所討論的procedure 不存在，也無法直接搜索以找到相應的戰術。我們利用FAISS來查找我們的數據集中每個被測試程序的前3個相似程序。這3個相似的程序將提供最多3個URL，RAG將使用這些URL 來檢索作為prompt上下文的文本片段。我們選擇使用3個8,000字符的文本片段，重疊度500（使用langchain框架），並將它們存儲在Vector Store中。然後，我們對問題和存儲的片段都使用OpenAI embeddings，並根據問題檢索前3個最相關的chunks ，將它們作為‘相關上下文’插入到prompt中。GPT-3.5-Turbo使用的提示模板如下：

> 你是一名資安專家。請考慮下面提供的相關上下文並回答問題。
> 

> 相關上下文：{上下文}
> 

> 問題：知道<<procedure>>，一個資安對手會使用這個技術實現哪些MITRE ATT＆CK戰術？請僅回答你確定的MITRE ATT＆CK戰術。
> 

需要注意的是，在這種情況下，一些相似的程序可能與所討論的程序具有相同的URL。這可能會導致GPT-3.5-Turbo通過RAG直接找到答案。然而，正如我們將在結果中展示的那樣，即使精確的URL位於找到的URL中，性能也很好，但仍然不完美。我們還注意到，ATT＆CK的技術/子技術URL實際上包含了戰術名稱（見[圖3](https://www.notion.so/Advancing-TTP-Analysis-Harnessing-the-Power-of-Encoder-Only-and-Decoder-Only-Language-Models-with-R-b01f70d0a08a40fc9705e0208e79004b?pvs=21)中顯示的屏幕截圖）。

![Untitled](Advancing%20TTP%20Analysis%20Harnessing%20the%20Power%20of%20Enc%20b01f70d0a08a40fc9705e0208e79004b/Untitled%203.png)

為了提供理想的RAG性能作為參考，我們還測試了一個理想（但不現實）的情況，即所討論的程序所在的確切URL是提供給RAG的唯一URL。這強制選擇前3個8,000字符片段來自最相關的URL。我們注意到，一些URL包含的文本較少，而其他URL包含的文本較多。這種變化可能也會影響RAG的性能。儘管如此，正如我們將在結果中展示的那樣，這種理想情況實現了極佳的上限性能。

# Results & Discussion

首先評估哪個encoder-only模型表現更好。然後，比較帶有和不帶有 RAG 的decoder-only模型。

接下來，比較所選的encoder-only和decoder-only的LLM模型與 RAG 的能力和局限性。最後提供具體例子來深入分析。

本研究考慮了“樣本平均 F1” 分數。F1 分數捕捉了精確度和召回率的調和平均值。

- 考慮“召回率”來評估 LLM 解釋procedures並將其映射到預設戰術的能力
- 考慮“精確度”來評估 LLM 胡言亂語的程度。
- 樣本平均 F1 分數計算了每個實例的 F1 分數，然後對這些分數進行了平均。使用樣本平均分數而不是微觀、宏觀和加權平均分數的原因是為了減少由於戰術間procedures數量不平衡或過於強調具有多個標簽的procedures而產生的biased。

## Evaluation of SFT of Encoder-Only LLMs

我們使用精心挑選的MITRE ATT＆CK企業戰術、技術和子技術描述（mapped到AT＆CK的戰術）來對encoder-only LLM（RoBERTa-SFT和SecureBERT-SFT）進行微調。作者這裡強調不會使用procedure descriptions對這些模型進行微調，因為想評估這些LLM的“解釋”能力（不提供明確的程序，戰術映射給這些模型）。就結果而言，SecureBERT和RoBERTa的樣本平均F1分數分別為0.54和0.41。與RoBERTa模型相比，SecureBERT表現更佳，並被選中decoder-only LLM進行比較。

## Prompt-Only vs. RAG of GPT-3.5-Turbo

[表2](https://www.notion.so/Advancing-TTP-Analysis-Harnessing-the-Power-of-Encoder-Only-and-Decoder-Only-Language-Models-with-R-b01f70d0a08a40fc9705e0208e79004b?pvs=21)顯示了所有9,532個ATT＆CK程序描述的性能，這些描述根據ATT＆CK與其相關聯的戰術進行區分(參考圖2）。由於一些程序與多個戰術相關聯，因此所有支持的總和（10,952）超過了測試procedures 的數量（9,532）。根據結果顯示，僅使用prompt （即依賴於GPT-3.5的預訓練知識）在解釋ATT＆CK戰術的性能較差，樣本平均F1為'0.60'（但高於SecureBERT-SFT）。在14種戰術中，有4種戰術的F1分數超過了'0.50'，但資源開發的得分為'0.00'。相比之下，當使用RAG提供精確的URL（理想情況）時，模型的性能顯著提高，樣本平均F1達到'0.95'，其中有11種戰術的F1超過了'0.90'。這表明，通過為模型提供準確的信息（包含答案的數據片段），它可以在不需要進行微調的情況下實現顯著更好的性能。然而，在更現實的情況下（RAG與前3個相似程序），新的攻擊程序的確切信息並不容易獲取，性能雖然仍然優於只提供prompt ，但樣本平均F1得分降至'0.68'。這意味著人們對LLM解釋新的、可能是未見過的攻擊程序的期望應該有限，即使使用了RAG。

值得注意的是，decoder-only LLM的回答不僅包含戰術名稱。我們在本研究中應用了ATT＆CK戰術的關鍵字搜索，並忽略了潛在的類似詞（由於其生成能力）。我們對decoder-only LLM的回答進行了檢查，很少出現這種情況，如果我們進行了類似詞的全面提取，我們的性能測量可能會有很小的偏差（約1%）。

![Untitled](Advancing%20TTP%20Analysis%20Harnessing%20the%20Power%20of%20Enc%20b01f70d0a08a40fc9705e0208e79004b/Untitled%204.png)

## Decoder-Only w/ RAG vs. SFT of Encoder-Only

我們通過基於精確度、召回率和F1分數分析它們的性能，進一步比較了Encoder-only（SecureBERT）的SFT和帶有RAG的Decoder-only模型。對於帶有RAG的Decoder-only模型，我們將9,532個程序分為兩個子組：

1. 至少有一個前3大的URL與問題程序的URL切確配對到
2. 沒有前3大的URL配對到精確URL。

這種分離能夠詳細了解RAG在有沒有精確URL的情況下的表現，並提供了對比encoder-only和decoder-only模型之間的“主要”差異的比較。表3顯示了它們的性能。encoderonly模型在“精確度”方面表現比“召回率”好得多。這意味著SecureBERT（encoder-only LLM）嘗試提供精確的回答。然而，正如GPT3.5（decoder-only模型）的結果所示，它們在“召回率”方面遠遠優於“精確度”。這表明，LLM中的幻覺通常是在decode階段出現的，模型在試圖生成不嚴謹的存在或暗示的信息時。這是降低精確度的一個主要因素。因此，這表明需要在不影響“召回率”的情況下改進LLM中的“精確度”。此外，通過比較只有Prompt的結果（[表2](https://www.notion.so/Advancing-TTP-Analysis-Harnessing-the-Power-of-Encoder-Only-and-Decoder-Only-Language-Models-with-R-b01f70d0a08a40fc9705e0208e79004b?pvs=21)）和沒有匹配的URL（[表3](https://www.notion.so/Advancing-TTP-Analysis-Harnessing-the-Power-of-Encoder-Only-and-Decoder-Only-Language-Models-with-R-b01f70d0a08a40fc9705e0208e79004b?pvs=21)），我們可以看到，當LLM提供了額外的“‘distracting’”的上下文（RAG）時，其表現比僅依賴於模型訓練知識更差。這表明RAG對檢索到的上下文（插入到prompt中）有很高的attention，並且不attention“正確”的答案（因為它試圖尋找額外上下文的最適合的片段）。

總結，[表2](https://www.notion.so/Advancing-TTP-Analysis-Harnessing-the-Power-of-Encoder-Only-and-Decoder-Only-Language-Models-with-R-b01f70d0a08a40fc9705e0208e79004b?pvs=21)和[表3](https://www.notion.so/Advancing-TTP-Analysis-Harnessing-the-Power-of-Encoder-Only-and-Decoder-Only-Language-Models-with-R-b01f70d0a08a40fc9705e0208e79004b?pvs=21)中顯示的結果表明，具有RAG的decoder-only LLM（GPT-3.5）通常優於encoder-only LLM（SecureBERT）。然而，值得注意的是，與decoder-only 模型相比，encoder-only LLM只需要更少的資源進行訓練和評估。這種區別對於理解這兩種模型架構之間效率和效果之間的權衡至關重要。

![Untitled](Advancing%20TTP%20Analysis%20Harnessing%20the%20Power%20of%20Enc%20b01f70d0a08a40fc9705e0208e79004b/Untitled%205.png)

![Untitled](Advancing%20TTP%20Analysis%20Harnessing%20the%20Power%20of%20Enc%20b01f70d0a08a40fc9705e0208e79004b/Untitled%206.png)

## Specific Examples for Decoder-only w/ RAG

把LLM應用於TTP解釋提出了幾個挑戰。在這一部分中，我們展示了Decoder-only LLM如何處理檢索到的文本。 

### example 1

---

**問題中的程序（策略：憑證訪問）：**

Lslsass可以從lsass進程中轉儲活動登錄會話密碼哈希。

**前3個相似的程序：** 

1.  Mafalda可以從LSASS.exe轉儲密碼哈希。 
2.  Earth Lusca使用ProcDump來獲得LSASS進程內存中的憑證哈希。
3. CrackMapExec可以從LSA秘密轉儲針對系統的散列密碼。

**檢索到的上下文：**

...對手試圖訪問存儲在本地安全性機構子系統服務（LSASS）進程內存中的憑證材料...

…策略：憑證訪問... 

---

在此示例中，'問題中的程序'的URL是與前3個相似程序之一關聯的URL之一。在這3個檢索到的URL中，所有3個都描述了憑證訪問的一個技術/子技術，這與問題中的程序關聯的策略相匹配。這樣的情況是一個理想的情況，RAG成功地為decoder-only LLM（GPT3.5）提供了正確的上下文信息。

### example 2

---

**問題中的程序（策略：執行）：**

HOPLIGHT使用svchost.exe執行惡意DLL。 

**前3個相似的程序：** 

1.  Hydraq使用svchost.exe執行包含在新服務組中的惡意DLL。 
2. Higaisa將一個shellcode加載器二進制文件命名為svchast.exe，以欺騙合法的svchost.exe。 
3. Tropic Trooper將一個DLL後門注入到dllhost.exe和svchost.exe中。 

**檢索到的上下文：**

...通過DLL注入的執行也可能從安全產品的檢測中逃脫，因為執行是掩蓋的…

...策略：防禦逃避，特權升級...

---

對於這個例子，問題中的程序'的URL也與前3個相似程序的URL匹配。然而，這次檢索到的片段包括'執行'、'防禦逃避'和'特權升級'的技術/子技術描述。GPT-3.5將此程序錯誤地映射到了'防禦逃避'和'特權升級'策略，但沒有'執行'。我們注意到，其中一個chunks中存在以下信息： 策略：防禦逃避，特權升級。我們認為模型過於attention這些特定關鍵詞，導致策略映射錯誤。有趣的是，即使一些句子討論了通過DLL進行的'執行'，這種情況仍然發生了。這個例子顯示了GPT-3.5對提供的上下文缺乏真正的理解能力，因為似乎模型過度優先考慮了某些關鍵詞，而忽略了內容的更廣泛的上下文。

### example 3

---

**問題中的程序（策略：防禦逃避）：**

Chimera已清除了受Compromised主機上的事件日誌。 

**前3個相似的程序：** 

1. Chimera已在受Compromised主機上本地設置了被盜數據。 
2. Chimera已使用受Compromised域帳戶來獲取對目標環境的訪問權限。 
3. Chimera已使用SMB在受Compromised主機之間複製工具。 

**檢索到的上下文：**

...監控那些看起來正在從不同的位置讀取文件並將它們寫入相同目錄或文件的進程，這可能表明數據正在進行本地設置... 

---

在這個例子中，“問題中的程序”的URL與前3個相似程序的任何URL都不匹配。即使沒有訪問source程序最初所在的reference URL，GPT-3.5也正確地將此程序預測為“防禦逃避”。 對於這個程序的檢索上下文，沒有提到“‘Tactics：”這個關鍵詞（不像示例1和2），也沒有提到14個ATT&CK策略的確切提及。然而，GPT-3.5模型根據檢索到的上下文正確解釋了此程序（如示例所示）。這表明了模型的一定解釋能力，即使沒有明確的文本提示或預定義的類別可用。

### example 4

---

**待解釋程序（戰術：資源開發）：**

GALLIUM使用了各種廣泛可用的工具，有些情況下他們修改了這些工具以添加功能和/或繞過反恶意軟件解決方案。

**前3個相似的程序：**

1. GALLIUM使用被盜的證書簽署了其工具，包括Whizzimo LLC的證書。
2. GALLIUM利用有效帳戶來維持對受害者網絡的訪問權限。
3. GALLIUM使用了修改版本的HTRAN，其中他們混淆了字符串，例如顯示試圖逃避檢測的debug messages。

**檢索的內容：**

...TA416消失并返回，帶著一個Golang PlugX惡意軟件加載程序...

---

在這個例子中，“待解釋的程序”的URL也與前3個的任何URL都不匹配。檢索到的上下文描述了“Resource Development”戰術的技術/子技術。對於這個程序，GPT-3.5完全亂想，預測了14個ATT＆CK戰術中的10個（不包括資源開發）。在這個程序的檢索到的片段中，並沒有提到“戰術：”和“資源開發”這些關鍵詞。我們還注意到，檢索到的信息與“待解釋的程序”無關。這表明GPT-3.5模型無法推理解釋程序描述（當提供不太相關的上下文時）。

以上的例子揭示了RAG如何幫助或干擾decoder-only LLM回答問題。

- 例子1和2表明，LLM實際上是在尋找關鍵詞來提供答案（而不是解釋它）。
- 例子3顯示了模型對“防禦-逃避”這個戰術的某些理解。
- 在例子4中，我們看到額外的上下文使模型亂講話，而不是增進它的效能。

仔細觀察，我們發現對於9,532個描述中有8,543個（約90％）包含“‘Tactic：”或“‘Tactics：”關鍵詞（參見圖3）。出現在檢索到的片段中

![Untitled](Advancing%20TTP%20Analysis%20Harnessing%20the%20Power%20of%20Enc%20b01f70d0a08a40fc9705e0208e79004b/Untitled%207.png)

事實上，在 4115 个 "匹配URL "中，有 3866 个（∼94%）存在這種“Tactic：...... "格式。與[表3](https://www.notion.so/Advancing-TTP-Analysis-Harnessing-the-Power-of-Encoder-Only-and-Decoder-Only-Language-Models-with-R-b01f70d0a08a40fc9705e0208e79004b?pvs=21)中顯示的數字相互對照，推測這是“匹配的URL”性能高的主要原因（0.88 F1分數）。相比之下，在“無匹配的URL”的情況下，我們觀察到5,417個描述中的4,677個（約86％）具有“Tactic：...”格式，同時具有較低的F1分數（0.52），也低於僅使用prompt的情況。這代表在提供額外上下文時，decoder-only LLM中使用RAG缺乏真正的“解釋”能力，而是試圖通過查找其“最佳猜測”的內容來尋找答案。

![Untitled](Advancing%20TTP%20Analysis%20Harnessing%20the%20Power%20of%20Enc%20b01f70d0a08a40fc9705e0208e79004b/Untitled%208.png)

# Concluding Remarks

我們進行了對smaller encoder-only LLMs進行監督微調的模型跟使用RAG增強的larger decoder-only LLMs的性能進行比較分析。這種比較旨在確定哪種模型類型在分析網絡攻擊程序描述方面更有效。我們觀察到，當用適當的RAG input補充decoder-only  LLMs時，對TTP的解釋顯著改善。這表明了正確信息對LLM性能的影響。

此外，我們的研究揭示了RAG在完善TTP描述方面的不同方法，同時展示了RAG技術的局限性，特別是當它們未能檢索到最相關信息時。我們指出，目前的decoder-only LLMs雖然具有很高的召回率，但通常缺乏準確性。這一觀察結果強調了在不犧牲召回能力的前提下增強LLMs準確性的新方法的必要性。此外，我們還展示了decoder-only LLMs在提供相關上下文時缺乏“解釋”的能力，並通過具體例子提供了分析。在這種情況下，我們推測LLMs試圖通過搜索它們認為最可能的解決方案來提供回答。

[逐字稿](https://www.notion.so/2da44c94d4544adf9c77c4ead34173ea?pvs=21)

[問題](https://www.notion.so/8dd70ae9d2f2425a9b482476914c117e?pvs=21)

[Advance TTP 報告](Advancing%20TTP%20Analysis%20Harnessing%20the%20Power%20of%20Enc%20b01f70d0a08a40fc9705e0208e79004b/Untitled.pptx)
