# Vulcan: Automatic extraction and analysis of cyber threat intelligence from unstructured text

# Introduction

![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled.png)

![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled%201.png)

## 2.1. CTI

- CTI 是一種劃分資訊安全的數據集合，可以更容易的讓使用者了解網路威脅並更有效的去建立防禦策略，其中最受關注的是 "indicators of compromise"（IOCs），它們用於識別系統或網絡上的惡意活動，如惡意 IP 地址和惡意文件的 MD5 哈希值，這些都是 IOCs 的例子。因為CTI的資訊來源有很多種，像是公家機關，私人公司，社群軟體，為了統一這些CTI有提出一些標準，像是"STIX" 和 "TAXII”，它們定義了用於交換 CTI 數據的協議和標準語言，還有一些威脅情資平台像是"MISP" 和 "OpenCTI"，它們的目標是有效地收集、存儲和管理關於網絡威脅的 CTI 數據。

## 2.2. Information extraction

- 從公開來源的文章中提取信息的時候，這些文章包含了有關事件的各種方面的信息。然而，這些文章通常以不同格式編寫，並且包含了人們不太感興趣的信息。機器無法直接處理原始文本數據。因此，需要通過一個名為信息提取（IE）的過程，從非結構的文本數據中提取結構化數據。

信息提取（IE）任務包括兩個子任務：

1. 命名實體識別（NER）：這一子任務的目標是在句子中定位並分類出命名實體，然後將它們歸類到預定義的類別中。命名實體可以是人名、地點、組織名稱等。例如，在圖中的示例中，NER 模型將“Jaff”和“emails”分別分類為勒索軟件和攻擊向量標籤。
2. 關係提取（RE）：這一子任務的目標是確定NER模型識別的命名實體之間的語義關係。例如，RE 模型在圖中的示例中確定了“Jaff”和“emails”之間的關係，並將其標記為“spread_via”。

![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled%202.png)

信息提取的結果是結構化的數據，這種數據使得可以進行各種類型的數據驅動分析，例如發現隱藏的關係和趨勢分析。通過將自由文本轉換為結構化數據，可以更輕松地進行自動化分析和應用

# 3. Problem statement

先介紹目前現有的CTI system 問題跟作者這次所想做到的範圍，之後會解釋從unstructured 文件中提取CTI data所會遇到的困難及挑戰

## 3.1. Research goal

1. 研究目標（Research goal）：
    - 研究的主要目標是解決現有威脅情報（CTI）系統存在的不足，這些系統用於分析和應對新興的網絡威脅。
    - 現有的CTI系統主要集中在收集一種特定類型的CTI數據，即威脅的跡象（IOCs）。
    - 但是，對於理解和檢測威脅的效力仍存在疑問，因為IOCs難以提供全面的威脅視圖，且具有短暫的實用壽命(大約兩天)，且很容易修改像是換個IP或名稱。
2. 解決方法（Addressing the shortcoming）：
    - 為了解決現有CTI系統的不足，研究旨在收集多種描述攻擊的CTI數據，特別是戰術、技術和程序（TTP），如圖2所示:
    
    ![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled%203.png)
    
    - 文中提到，TTP相對於IOCs具有更長的實用壽命，因為攻擊者難以輕易改變他們可用的技術，並且開發全新的技術需要時間和系統知識。
    - 這些與TTP相關的CTI數據有助於檢測和理解網絡威脅，特別是針對勒索軟件攻擊的CTI數據。
3. 研究範圍（Fig. 3 shows CTI data and their relationships to be covered in this work）：
    - 雖然有各種威脅類型（例如拒絕服務、帳戶劫持），但研究專注於提取與勒索軟件攻擊相關的CTI數據。
    - 文中提到，圖3顯示了研究中將涵蓋的CTI數據以及它們之間的關係。
    
    ![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled%204.png)
    
    - 名詞介紹:
        1. 勒索軟件（Ransomware）：勒索軟件是一種恶意軟件，旨在加密受害者設備上的所有文件，然後要求贖金以提供解密密鑰。
        2. 攻擊向量（Attack-vector）：攻擊者使用多種攻擊向量來將勒索軟件傳送到受害者設備上。例如，攻擊向量包括通過驅動程序下載、釣魚攻擊和可移動媒體等方式。
        3. 漏洞（Vulnerability）：漏洞指的是軟件中公開披露的安全漏洞。這裡包括了CVE標識符和微軟安全公告。
        4. 平台（Platform）：平台指的是各種操作系統，包括Windows、Linux和Android等，這些是勒索軟件攻擊的主要目標。
        5. 算法（Algorithm）：一旦勒索軟件進入受害者設備，它會使用加密算法來靜默加密一組文件。
        6. 工具（Tool）：一旦勒索軟件進入受害者設備，它會使用多種類型的工具執行惡意行為，例如側向移動。這包括了從Windows內建工具到開源工具的多種類型工具。
        7. 行為（Behavior）：行為指的是恶意软件（包括勒索軟件）執行的惡意技術。
        8. 日期（Date）：日期用於描述勒索軟件首次被發現的時間，並用於分析勒索軟件隨時間的特徵變化。

## 3.2. Technical challenges

1. 先前的CTI研究方法：
    
     之前的CTI研究提出了各種方法來從非結構文本數據中提取威脅的跡象（IOCs）。
    
    - 拍這些方法包括基於dictionary-based、rule-based、基於deep learning-based不同類型方法。
2. 技術挑戰（Technical challenges）：
    - 存在一些挑戰，使得這些方法難以用於識別我們感興趣的CTI數據。
    - C-1：基於dictionary-based的方法無法識別以前未見過的CTI數據，也無法區分同名但含義不同的CTI數據。
    - C-2：與IOCs不同，我們感興趣的CTI數據可以由任意字符和數字的組合表示，難以使用正則表達式等固定模式。
    - C-3：基於深度學習的方法需要大量labeled data來訓練深度神經網絡，難以應對多種類型的CTI數據。
    - C-4：現有方法大多專注於提取單獨的IOCs，而未考慮它們之間的關係，然而，為了深入了解威脅的特徵，應該考慮CTI數據之間的語義關係。

# 4. The design of Vulcan

- 為了解決上述提到的問題，作者開發了一個新型的CTI system，圖4顯示了這個的總體架構跟流程，並會一一介紹

![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled%205.png)

## 4.1. Overall workflow

- Vulcan分為兩部分:CTI data collection跟CTI data usage，data collection又分為五個部分，每個部份以前一個部份的輸出作為輸入。
- data collection:
    1. **data scraper**: Vulcan通過數據爬取器收集可能包含CTI數據的文本數據，這些數據來自多個來源（例如威脅報告、文章）
    2. **pre-processor**: pre-processor對收集的數據進行過濾，僅選擇與勒索軟件攻擊相關的數據。
    3. **threat entity identifier(TEI)**: TEI從文本數據中提取實體（entities），例如，文中提到了一個句子：“Revil has been distributed via malicious e-mail attachment.” TEI將兩個單詞Revil和e-mail分別分配為實體類型，分別表示ransomware（贖金軟件）和vector（攻擊向量）。
    4. **entity linker**:分為entity segmentation and entity integration:
        - entity segmentation:把不同事物的同名實體分離開來。例如，盡管 Magniber 勒索軟件家族有多個變形，但它們都使用相同的名稱 "Magniber"。因此，Magniber 實體根據變形的獨特特徵被分離成不同的事物
        - entity integration:將具有不同名稱但指相同事物的實體（如 MailTo、NetWalker 和 Koko）連接起來
    5. **threat relation identifier**:用來確認同一個句子中兩個entity的語意關係，ex: entity pair (Revil, e-mail) 被指定為 spread_via type.

Vulcan將收集到的CTI數據及其關係存在graph database中，並提供搜索API，以使從業人員能夠訪問數據庫。這些API使他們能夠開發各種類型的用於威脅分析的應用程序。文中還介紹了使用搜索API構建的兩個應用程序，分別是“evolution identification”和“threat profiling”

## 4.2. Data scraper & pre-processor

- **Data scraper**:
    - Data scraper類似於一種爬蟲程序，用於從多個網站中收集以自然語言書寫的與威脅相關的文本數據。
    - Data scraper首先開始收集文章的連結，直到每個網站上再也沒有連結為止。
    - 對於每個文章連結，數據爬取器提取標題、編寫日期、內容和域名等信息。
    - ex:在Twitter的情況下，數據爬取器收集包含“Ransomware”標籤的推文。
    - 一些網站在提供詳細解釋之前提供了結構化的威脅摘要，數據爬取器也收集這些數據，作為威脅分析的資源。
- **pre-processor**:
    - pre-processor用於對通過Data scraper收集的文章進行文本處理，包括topic filtering和text sanitization。
    - 首先，它通過一組與勒索軟件攻擊相關的關鍵詞進行篩選，以確定文章是否涉及勒索軟件攻擊(Examples of keywords include ‘encrypt’, ‘decrypt’, ‘demand’, and ‘ransom.’)。
    - 其次，為了提高input text的品質，預處理器透過正規表示式從文章中刪除noisy字符，如HTML標記和空白字符，以獲得clean text
    - 最後，對每篇文章進行句子分割，將文章劃分為句子列表。

## 4.3. Threat entity identifier (TEI)

1. 先前的NER研究和挑戰：
    - 先前在自然語言處理（NLP）中提出了許多關於命名實體識別（NER）的研究。
    - 存在多個開源的NER工具，如CoreNLP和SpaCy。
    - 但是，NER任務高度依賴於目標領域，因此為一個領域設計的NER模型在其他領域難以表現良好。
    - 作者感興趣的實體不遵循特定模式，並且經常出現以前未見過的實體，如勒索軟件的名稱和工具，因此現有based on dictionaries or rules的NER model無法捕捉到和ransomware有關的實體
2. TEI模型的設計：
    - 為解決上述問題，他們設計了TEI，這是一個基於well-known language model BERT進行fine-tuning的NER模型，專門針對勒索軟件攻擊進行設計。
    - TEI的目標是使用文本中單詞的上下文信息，並利用單詞的雙向信息進行學習。
3. TEI模型的架構：
    - 表五show了TEL的架構，分別有三個layer:BERT、BiLSTM跟[CRF](https://www.notion.so/Vulcan-Automatic-extraction-and-analysis-of-cyber-threat-intelligence-from-unstructured-text-c3d4260f8b7442fb8f8c4e6262b9f852?pvs=21)
    
    [（四十四）通俗易懂理解——BiLSTM-CRF](https://zhuanlan.zhihu.com/p/115053401)
    
    ![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled%206.png)
    
    [一文读懂BiLSTM+CRF实现命名实体识别 — PaddleEdu  documentation](https://paddlepedia.readthedocs.io/en/latest/tutorials/natural_language_processing/ner/bilstm_crf.html)
    
    ![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled%207.png)
    
    - 第一步先把輸入句子轉換成一連串的token，變成BERT模型的格式
    - 第二步把BERT模型中提取的final hidden state vectors(這裡頭包含了上下文中每個位置的語意表示)餵入BiLSTM層(Bert layer中的Rbad中的R代表經過處理的高維向量，這就是final hidden state vectors?)，The BiLSTM layer從forward跟backward提取feature(標籤向量)
    
    ![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled%208.png)
    
    - 第三步BiLSTM的output餵給CRF(CRF的作用就是在所有可能的路径中，找出機率最大效果最佳的一条路径，那這個標籤序列就是模型的輸出(也就是解碼的意思，softmax也是一樣的作用，CRFs are discriminative(判別) models。)去decode出best label sequence
    - 最後對於每個輸出給上不同的label，有這七種:❶ ransomware, ❷ attackvector, ❸ vulnerability, ❹ platform, ❺ algorithm, ❻ tool, and ❼other
    
    [BiLSTM跟CRF](https://mp.weixin.qq.com/s?__biz=Mzg5ODAzMTkyMg==&mid=2247488458&idx=1&sn=50d4bd1e94c85b4c589baa9be6757ae7&chksm=c0699b97f71e1281c62c7e3691cbc4981afd0f8f0a4f7070beaa15b33e687b3431914d5ab851&scene=21#wechat_redirect)
    

## 4.4. Entity linker

1. TEI的限制：
    - NER模型，包括TEI，通常僅能識別文本中的命名實體，但不能理解實體的語義含義。
    - 這導致兩個主要問題：(i) 單個實體經常指的是不同的事物，或 (ii) 具有不同名稱的多個實體指的是相同的事物。
2. Entity linker的操作：
    - 實體鏈接器執行兩種類型的操作，分別是"Entity segmentation"和"Entity integration"。
3. Entity segmentation:
    - 這個操作解決了第一個問題，當文本中未明確提到實體時，如多個變體的勒索軟件。
    - 例子:A勒索軟體又出現了，這次又變得更強大，在這句中TEI會識別A為勒索軟體，但這裡的A指的是第二版本的A勒索軟體，而不是之前第一版本的A
    - 通過分析文本中的Twitter帖子，識別出可能涉及多個勒索軟件變體的候選實體。
    - 候選實體的Twitter帖子通常包含不同的技術術語，用於描述各個變體，這些術語用於區分不同的變體。
    - 通常當突然twitter出現很多相關惡意程式的推文時，通常就代表有新的變體出現，此時再去提取他的推文看有沒有出現新的技術術語，當有出現新的術語時就會把這些候選實體分成不同實體，會帶上版本號（例如，Magniber v1.0，Magniber v2.0）
    
    ![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled%209.png)
    
4. Entity integration:
    - Revil 和 Sodinokibi 這兩個不同名稱的勒索軟件實體指的是同一個勒索軟件，EternalBlue 和 CVE2017-0144 這兩個不同類型的實體指的是同一個漏洞。這些指同一事物的實體在各種來源（如文章、部落格）中被交替使用，由於 TEI 是根據名稱來區分實體的，因此它將名稱不同的實體視為不同的事物
    - 從提供結構化 CTI 數據的多個來源收集威脅相關術語及其別名。然後，對於每個別名對 (a1，a2)，我們在名稱與別名中對名稱相同的實體之間添加 alias_for 關係。在這項工作中，我們將由 alias_for 關係連接的實體視為同一事物
    - 這個操作解決了第二個問題，即一個實體可能有多個不同的名稱（別名）。
    - 通過構建一個包含別名對（a1, a2）的字典，其中a1是a2的別名，將不同名稱的實體關聯在一起。
    - 當搜索特定實體時，與輸入實體具有別名關係的實體也被一同考慮。

## 4.5. Threat relation identifier (TRI)

TEI能夠從輸入的句子中識別與勒索軟體攻擊相關的entity，並將它們分類為預定義的類別，例如勒索軟體和工具。對於已識別的實體，我們需要通過一個稱為關係提取（RE）的過程來提取它們之間的語義關係。RE任務被制定為一個成對分類問題，用於確定同一句子中所有可能的實體對（e1、e2）的類別。為了執行這個任務，我們設計了一個基於語言模型的RE模型，稱為Threat relation identifier（TRI）。具體來說，**我們對BERT模型進行了RE task的fine-tune**。

![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled%2010.png)

如圖7所示，TRI接收一個句子和一個entity pair作為輸入，並分配給以下標籤之一，用於描述entity pair之間的關係：❶ variant_of(e1, e2)、❷ variant_of(e2, e1)、❸ spread_via、❹ exploit、❺ target、❻ encrypt、❼ use和❽ other。由於variant_of類型由兩個相同類型的實體（例如，ransomware, ransomware）組成，根據關係的方向，它被細分為兩種類型，(像是A是B的後代或是B是A的後代)。而"other"表示給定的兩個實體之間的關係不屬於前面提到的7種關係類型之一。如果一個句子中包含三個或更多實體，我們將組合所有可能建立語義關係的實體對，並將它們視為不同的輸入(代表他們有可能可以組出更多的語句)。在訓練TRI之前，我們對輸入句子進行預處理(是手動嗎)，以使TRI更好地學習實體之間關係的語境信息。

首先，在兩個輸入實體的開頭和結尾插入特殊標記（即<e1>，</e1>，<e2>和</e2>），這可以提高關係提取的性能(可以更方便知道句子中哪些是entity哪些是其他內容)。其次，輸入句子中提到的兩個實體對它們的類型名稱做mask，以防止TRI overfitting。預處理後的句子然後被轉換成BERT模型使用的格式。每個句子通過WordPiece tokenizer被分成一個token sequence，該分詞器將單詞分成預定義詞彙中包含的多個sub-words。接下來，[CLS]標記附加到token sequence的開頭。為了確定兩個給定實體之間的關係類型，TRI使用BERT模型的[CLS]和兩個實體類型的final hidden state vectors。使用[CLS]標記的向量的原因是它包含sentence-level contextual information。由於某些實體類型被tokenized為多個tokens，它們用多個向量表示。因此，對每個entity types的hidden state vectors進行average operation(r計算平均向量用來代表這個實體)。然後，將[CLS]和兩個實體類型的三個向量分別輸送到[fully connected（FC）層](https://leemeng.tw/shortest-path-to-the-nlp-world-a-gentle-guide-of-natural-language-processing-and-deep-learning-for-everyone.html#%E5%85%A8%E9%80%A3%E6%8E%A5%E5%B1%A4)。FC層的輸出被串聯在一起，並輸送到另一個FC層，然後是一個[softmax層](https://cloud.tencent.com/developer/article/1676286) [這個也是介紹](https://leemeng.tw/shortest-path-to-the-nlp-world-a-gentle-guide-of-natural-language-processing-and-deep-learning-for-everyone.html#Softmax-%E5%87%BD%E5%BC%8F)(用來計算每個單詞的概率，原本輸出可以是任意值，把他移到0跟1之間，因為提取出來的label是0或1)，三個輸出向量壓縮到像是分別為0.4,0.3,0.3總和為一。TRI根據項量最高分識別輸入實體之間的關係類型 。例如，如圖7所示，TRI為實體對（ChaCha，Maze）分配variant_of(e2,e1)類型，這意味著Maze是ChaCha勒索軟體的一個變種。

[[Notes] BERT / BERT 架構理解](https://haren.medium.com/paper-notes-bert-bert-架構理解-31c014d7dd63)

# 5. Evaluation

## 5.1. Settings

- 進行這項實驗的伺服器用了40 of Intel Xeon E5-2630 CPUs, 12 of 16 GB memories, and 4 TITAN Xp
- 為了評估Vulcan，作者建立了一個ground-truth dataset其中註釋了包含6,791個entity和4,323個entity之間relation的句子，找了五個研究生手動標註地，而標註任務有兩步驟:
    1. 使用[Fig. 3](https://www.notion.so/Vulcan-Automatic-extraction-and-analysis-of-cyber-threat-intelligence-from-unstructured-text-c3d4260f8b7442fb8f8c4e6262b9f852?pvs=21)中定義的實體類型對句子中的單字進行標註。在這一步中註釋的單詞用於訓練和評估TEI
    2. 作者label被註釋的實體之間的語義關係，在這一步中註釋的關係用於訓練和評估TRI
    
    圖8就是被註釋過的句子的例子，每句的命名實體中都包含至少一種relationship
    
    ![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled%2011.png)
    
- 除了勒索軟體攻擊的關係類型之外，作者也考慮與其他類型相對應的negative samples of relations，以使 TRI 能夠區分我們在圖3中定義的感興趣的關係與其他關係。例如，考慮以下句子：“就WannaCry而言，它不會感染Mac系統。”在這裡，我們將其他類型賦予實體對（WannaCry，Mac）之間的關係。

![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled%2012.png)

- 某些類型的實體由多個單詞組成，所以使用 BIO tagging scheme標記這些單詞，該方案將 B 標記分配給命名實體的開始，I 分配給內部，O 分配給其他。例如，如圖8所示，構成單一實體 'Cobalt Strike' 的兩個詞 'Cobalt' 和 'Strike' 分別標記為 B-TOOL 和 I-TOOL。
    
    ![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled%2013.png)
    
- 語言模型
有許多基於深度神經網絡的語言模型，包括BERT（Devlin等人，2018）、GPT3（Brown等人，2020）和XLNet（Yang等人，2019）。由於這些模型在大量文本數據上進行預訓練，它們的輸出包含豐富的單詞上下文信息。例如，BERT模型使用英語維基百科（2500 M字）和BookCorpus（800 M字）進行預訓練，基於masked language modeling（MLM）和next sentence prediction（NSP）。因此，基於預訓練語言模型的各種NLP任務，包括問答、自然語言推理，都取得了領先的結果。選擇使用哪種神經語言模型取決於目標（例如性能、推理速度）和目標NLP任務。例如，DistilBERT（Sanh等人，2019）是BERT（Devlin等人，2018）的一個縮小版本，性能略低於BERT，但保證了更快的推理速度。作者選擇BERT作為基本神經語言模型，是因為它有多個版本適用於不同環境。
    - 暫時先可以不用講
        - 與外部source的整合
        獲取更深入和更廣泛的對網絡威脅的理解的一種方式是盡可能收集多的CTI數據。為了擴展Vulcan的數據集，我們從一些外部來源（例如MITRE，2022；NVD，2022）中收集結構化的CTI數據及其關係，並將它們整合到Vulcan中。例如，MITRE提供了工具和對手行為之間的結構化關係，如（PsExec，lateral movement）、（AdFind，discovery）。通過將這些關係整合到Vulcan中，我們可以通過從Vulcan中提取的（勒索軟件，工具）的關係了解勒索軟件進行哪些惡意行為的詳細信息（詳見第6.2節）

## 5.2. Performance evaluation

### 針對TEI

1. 目的: TEI的主要目的是對輸入的句子進行分析，從中提取出六種不同類型的實體：(1) 勒索軟體（ransomware）、(2) 攻擊向量（attack-vector）、(3) 弱點（vulnerability）、(4) 平台（platform）、(5) 演算法（algorithm）和(6) 工具（tool）。
2. 評估方法: 為了針對TEI的性能評估，用**三個指標**評估TEI將句子中的單詞準確分類為相應實體類型的能力：
    - (1) 精確率（Precision）所有被預測為正確的樣本中，有多少比例是實際正確的(指正確識別的比例
    - (2) 召回率（Recall）所有實際正確的樣本中，有多少比例是被預測為正確的。
    - (3) F1分數（F1-score）是精確率和召回率之間的調和平均值。
    
    [Precision, Recall, F1-score簡單介紹](https://medium.com/nlp-tsupei/precision-recall-f1-score簡單介紹-f87baa82a47)
    
3. 實驗設定:batch大小為16組tensor，執行30個epochs（想要問說這樣的意思是指這16個batch分別跑了30次嗎)。經驗上我們發現，ground-truth dataset中的所有句子都不超過128個token涵以token化的數量。因此輸入的最大序列長度設置為128，這是模型接受的最大輸入長度

- 我們以7:3的比例隨機分割ground-truth dataset，將其分為訓練集和測試集，並重複實驗5次以獲得試驗的平均性能。表格1顯示了每種實體類型的平均精確度、召回率和F1分數。 TEI以平均精確度0.968和平均召回率0.977檢測我們感興趣的實體。在本研究中涉及的實體類型中，漏洞類型的F1分數最高，為1.000，而攻擊向量類型的F1分數最低，為0.943。我們觀察到**，那些傾向於遵循固定模式的實體類型表現出色，如漏洞和算法**。這可以歸因於實體類型的模式是確定給定詞的實體類型的重要特徵。另一方面，對於諸**如勒索軟體和攻擊向量之類的實體類型，可以確定該類型的特徵在給定詞中並不明顯**。因此，在識別勒索軟體和攻擊向量時存在一定的性能下降。

![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled%2014.png)

![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled%2015.png)

![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled%2016.png)

### 針對TRI

- 接下來，評估**TRI的性能**，用來了解TRI在多大程度上準確地確定實體之間的語義關係。在這裡，實體都先被正確標記，專注於評估TRI是否正確識別了實體對之間的關係。與先前的實驗一樣，我們將ground-truth數據集按7:3的比例分成訓練集和測試集，用於每種關係類型。batch size為32，epochs為20。
- TRI的average precision為0.981，average recall為0.90。實驗結果證實了TRI的參數被訓練成能夠正確判斷給定輸入的關係類型。此外，TRI在區分實體之間存在方向的variant_of類型的關係方面表現出很高的precision和recall。這是因為放置在實體之間的單詞有很明顯的不同(不會有很相同的嗎)，例如，對於variant_of（e1，e2），可以是<e1，a successor to e2>，而對於variant_of（e2，e1），可能是<a variant of e1，dubbed e2>。與在網絡安全領域進行關係提取的現有工作（Dong等，2019；Jo等，2020；Pingle等，2019）相比，TRI在提取CTI數據之間更多樣化的關係類型時表現出可比較的性能。

![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled%2017.png)

### Baseline comparison

- 為了比較Vulcan與現有的NER（命名實體識別）、RE（關係提取）模型的性能，我們設計了baseline NER和RE模型，並使用Vulcan的數據集進行訓練。更具體地說，我們實現了兩個基於CRF和BiLSTM + CRF的NER模型，用來從unstructured text中識別命名實體。然後實現了一個基於卷積神經網絡（CNN）的RE模型，用於確定實體之間的語義關係。
- [表3](https://www.notion.so/Vulcan-Automatic-extraction-and-analysis-of-cyber-threat-intelligence-from-unstructured-text-c3d4260f8b7442fb8f8c4e6262b9f852?pvs=21)顯示了Vulcan和現有NER、RE模型的性能比較。首先，與CRF、BiLSTM + CRF模型相比，我們的NER模型TEI在precision和recall方面均表現更好。其次，我們將兩個RE模型:CNN和TRI與兩種不同類型的數據集進行性能比較，即（1）ground-truth（g-truth）數據集和（2）NER模型的輸出（即CRF、BiLSTM + CRF和TEI）。g-truth數據集包含正確標記的實體及其關係，用於評估RE模型如何分類實體之間的關係。另一方面，NER模型的輸出包括一些錯誤分類的實體及其關係，用於評估[end-to-end](https://blog.csdn.net/qq_34840129/article/details/89388897)的端到端性能。

![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled%2018.png)

- 在 g-truth 數據集上，CNN 模型和 TRI 都表現出很高的性能，精確度和召回率都超過了 0.9。在 5 個不同的實驗案例中，TRI 的性能最高。另一方面，當RE 模型使用 NER 模型的輸出時，RE 模型的性能有所下降。特別是當CNN使用 CRF 模型的輸出時， CNN 模型的性能最差，精確度為 0.783，召回率為 0.839。為什麼沒有CNN(w/TEI)
- 性能下降的原因:是因為NER模型的錯誤不可避免地傳播到RE模型。NER模型的錯誤使得RE模型無法識別實體對的正確關係。例如，NER模型無法從以下句子中識別勒索軟件實體Magniber：“南韓的PC用戶嚴重受到Magniber勒索軟件的威脅，黑客通過Magnitude Exploit Kit進行傳播。” 然後，具有使用關係的實體對（Magniber，Magnitude）無法被抓出來。因此，為了提取實體之間的正確關係，NER和RE模型都應提供高精度和召回率。

# 6. Applications for threat analysis

作者提出的威脅情報系統Vulcan，從非結構化數據中提取了大量的威脅情報數據及其關係。為了提供一個安全從業者可以開發威脅分析應用的環境，我們實現了一組搜尋API，用於訪問收集到的威脅情報數據。[表4](https://www.notion.so/Vulcan-Automatic-extraction-and-analysis-of-cyber-threat-intelligence-from-unstructured-text-c3d4260f8b7442fb8f8c4e6262b9f852?pvs=21)列出了Vulcan支持的搜尋API的摘要。目前，我們提供基本功能，比如查找與實體x存在y關係的實體，但計劃在不久的將來支持比較複雜功能。通過這些支援的API，安全從業者可以開發他們需要的威脅分析應用。在本節中，我們介紹了兩個有助於**理解**和**檢測**不斷演變的威脅的應用程序：***evolution identification***, and ***threat profiling***。

![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled%2019.png)

## 6.1. Evolution identification

Vulcan 的實用功能之一是了解勒索軟體攻擊在技術上是如何隨時間演變的。對於資安人員來說，這是建立防御策略打擊勒索軟體的必要步驟。

- [圖9](https://www.notion.so/Vulcan-Automatic-extraction-and-analysis-of-cyber-threat-intelligence-from-unstructured-text-c3d4260f8b7442fb8f8c4e6262b9f852?pvs=21)顯示了一個應用程序的程式碼，該應用程序把勒索軟件實體作為輸入參數並返回其演變時間軸。為了做到這一點，我們首先從輸入實體中找到所有可觸及的勒索軟體實體(變體)，而實體之間的關係類型對應於 "variant_of”。接下來，我們按照它們被檢測到的時間對實體進行排序。最後，我們提取與每個勒索軟件實體相連的任何類型實體，以提取勒索軟件實體的特徵。

![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled%2020.png)

- [圖十跟十一](https://www.notion.so/Vulcan-Automatic-extraction-and-analysis-of-cyber-threat-intelligence-from-unstructured-text-c3d4260f8b7442fb8f8c4e6262b9f852?pvs=21)顯示出了兩個勒索軟體家族演變的時間軸，顯示了每個變體新增了那些特徵，時間軸上，不標記已經出現過的特徵，以確保時間軸的清晰度。換句話說，為了使時間軸更易於理解，已經顯示過的特徵不再被標示或標記 ，其中像是Satan這個勒索軟體，再三年內發展出了三種新後代，以及可以看到他在加密方面使用混和加密方法，結合了RSA跟AES，後面的所以有變種也都採用這個方法。Satan勒索軟體家族的一個顯著變化是其具備傳播到網路中其他設備的能力。Satan的第二個版本DBGer開始利用兩個工具，Mimikatz和EternalBlue，以及兩個漏洞，CVE-2017-12149和CVE-2017-10271。DBGer變種透過Mimikatz從感染的設備中提取所有登錄憑證。它還掃描設備的本地網路，查找其他受EternalBlue漏洞（即CVE-2017-0144）或其他兩個漏洞（即CVE-2017-12149，CVE-2017-10271）影響的設備。然後，它嘗試使用竊聽的憑據或三個漏洞之一來破壞設備。Satan的第三個版本Lucky添加了在多個軟件（例如JBoss、Tomcat、Samba）中發現的多個漏洞，以利用基於Windows或Linux的系統。至於5ss5c變種，從Vulcan中並未識別到新的特徵。
- 而Magniber家族中的虛線是來自NVD網站的，他們最大的特色:
    1. 是一直都有在持續的一直更新，從2017到作者發這篇論文的時候都還在更新，
    2. 使用新的漏洞的速度很快，像是CVE-2021-34527剛出一個月就開始利用這個漏洞進行破壞
    
    這代表用戶定期把程式更新到最新版是非常重要的
    

![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled%2021.png)

## 6.2. Threat profiling

Threat profiling是對攻擊者難以改變的行為進行詳細分析，該程式接受兩個輸入參數：一個勒索軟件實體和一個要分析的特徵。然後，這個應用程式返回從輸入實體到輸入特徵的之間所有實體。 

![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled%2022.png)

# 7. Discussion

Vulcan還存在一些需要改進的地方。文本數據收集中最關鍵的兩個因素是（1）coverage（2）reliability。由於沒有文本數據就無法提取CTI data，因此CTI system應該盡可能的多收集文本數據。這邊的數據來源僅限於人工選擇與安全有關的資源和 Twitter，未來會考慮更多種多樣的來源像揭科學期刊

CTI系統應確保所收集的文本數據的可靠性。如果文本數據不可靠，其中包含的CTI數據的可信度也可能受到否定，CTI系統可能會根據其收集文本數據的地點獲得不同的CTI數據。此外，對手可能會上傳虛假的文本數據到網絡上，以干擾CTI系統，為解決此問題，我們需要考慮設計API，以實現現有可靠性驗證技術的應用

![Untitled](Vulcan%20Automatic%20extraction%20and%20analysis%20of%20cyber%20%20c3d4260f8b7442fb8f8c4e6262b9f852/Untitled%2023.png)

Vulcan被訓練用來識別6 types of CTI data and 7 types of relationships，代表Vulcan不能識別在訓練範圍外的新CTI data 。

# 8. Related work

和其他CTI system最大的差別是Vulcan主要在從unstructed data中提取IOCs以外的描述性或靜態的CTI資料，也會考慮CTI data互相的行為關係

基於預訓練的語言模型設計 NER 和 RE 模型，顯示出很高的性能。此外，我們的工作還為安全從業人員開發威脅分析應用程序提供了一個環境

# 9. Conclusion

大部分的CTI system都是提取IOCs，但IOCs的缺點就是有效時間很短，而且沒有描述威脅的技術細節，為了解決這個問題開發了Vulcan可以從unstructed data中提取任意形式的CTI data跟他的語意關係，Vulcan還提供資安人員search APIs去開發threat analysis的應用程式。

為了證明Vulcan的可用性，作者還展示了使用search API構建的兩個應用程式。

[逐字稿](https://www.notion.so/8b15e2e72b45405797b70681e3ea7899?pvs=21)

[要問的問題](https://www.notion.so/27d4f6a7997647ed877952b087f0fb06?pvs=21)
