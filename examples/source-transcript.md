# Source Transcript

> The talk that this skill's methodology is built from. Original language: Traditional Chinese (繁體中文). An English-language summary follows below for non-Chinese readers.

---

## Key Takeaways (English summary)

The talk synthesizes three threads:

### Part 1 — Why `/goal` exists across vendors

In 2026, Claude Code, OpenAI Codex, and Manus Agent simultaneously shipped a `/goal` feature. The shared motivation: LLMs suffer from **context anxiety (上下文焦慮)** — when their context window approaches full, they shortcut to a wrap-up and stop, regardless of whether the task is actually done. Anthropic's late-2025 research documented this pattern. `/goal` fights it by pairing an **implementer agent** with a **reviewer agent** that re-evaluates progress after every round.

### Part 2 — Why `/goal` succeeds or fails on prompt precision

A vague prompt like "make this project better" gives the AI freedom to define "better" itself, and it will pick the easiest definition and quit in three minutes. A precise prompt — illustrated in the talk by "drop the checkout page response time below 0.2s, verified by a speed test tool, only touching the checkout module, recording what changed each round" — keeps the same AI iterating for hours.

The talk extracts the structure of a good `/goal` prompt into **five elements**:

1. **Outcome** — what "done" looks like (e.g. "response time under 0.2s")
2. **Verification** — how to prove it's done (e.g. "verified by speed test tool")
3. **Constraint** — what must not be touched (e.g. "only the checkout module")
4. **Iteration policy** — what to record each round (e.g. "what changed / what's the result / next direction worth trying")
5. **Error handling** — when to pause and report (e.g. "if the test tool can't run, stop and report what was tried / what's blocking / what info would unblock")

### Part 3 — For subjective tasks: the 6-step rubric SOP

Most knowledge work (design, writing, marketing, video editing) is taste-based and has no unit tests. The talk references Anthropic's **frontend-design skill** case study — building a museum website — where they decomposed "beautiful website" into four dimensions: Design Quality, Originality, Technical Execution, Usability. The reviewer agent scored against this rubric, allowing the AI to iterate across 15+ rounds and produce designs the researchers themselves described as creative leaps they "couldn't have produced in a single prompt".

The talk then generalizes the rubric-building process into a 6-step SOP anyone can apply:

1. **Run a baseline.** Let the AI take one pass without any rubric. This calibrates its default behavior.
2. **Read every baseline output and write down each "frown point" — specifically.** Not "the opening is bad", but "the opening uses 「in this rapidly changing era」".
3. **Group frown points into dimensions.** 50 frown points might collapse into 3–4 categories.
4. **For each dimension, write specific "Never do X" rules.** Concrete enough that a reviewer agent can catch the violation at a glance. The talk references Anthropic's actual rule: *"Never use Inter / Roboto / Arial / system fonts."*
5. **Add diverse positive directions to avoid overfitting.** Anthropic's blog described their own failure: writing "design with museum-level quality" produced 11 outputs that all looked like museums. The fix: list 11 aesthetic styles (brutalist / art deco / pastoral / industrial / retro-futuristic / etc.) and let Claude pick one per iteration.
6. **Feed the rubric into the reviewer agent and run.** For the first few iterations, manually compare the reviewer's judgments to your own — if they diverge, the rubric isn't capturing your real taste yet.

### The deeper claim

The talk's closing argument: **evaluation, not prompt engineering, is what determines AI output quality.** Writing a rubric is ostensibly for the AI — but its real value is forcing *you* to put into words the taste that previously only lived in your head. Once it's in words, the AI can hold the line at scale.

This is the methodology `skill-to-goal` automates: the Q&A walks you through articulating outcome, verification, constraint, iteration policy, error handling, and (for subjective tasks) a complete rubric — codifying what was previously implicit in your head.

---

## Original Transcript (Traditional Chinese)

> The text below is the raw transcript of the source talk, preserved as-is. The talk was delivered as continuous speech with minimal pauses; line breaks have not been inserted to keep the source faithful.

影片分成三個部分第一我們會拆解最近主流的AI工具跟實驗室針對AI做Luningtask分別推出了什麼新功能又做了哪些研究第二從這幾個工具的設計原理來看讓AI能長時間運行的秘密到底是什麼？要怎麼做才能讓你的agent不免不休的為你工作？第三你該如何立刻把這套東西應用到你的工作上看完這支影片之後我希望你不再只是AI的協作者而是AI時代下的管理者那我們直接開始如果你有在關注AIagent的圈子你會發現一件很有趣的事情CloudCode,OpenAICodex還有hermusAgent這三間公司幾乎在同一時間推出了一個一模一樣的新功能甚至名字也都一樣叫做GO這個功能是乾嘛的簡單講就是你只要在對話框裡打斜線G然後寫下你想要它達成的目標它就會自己跑下去直到完成為我看社群媒體似乎不夠重視這個趨勢這些公司推出的這個功能其實都是想要解決一個所有AI工作者都遇到過的問題那就是AI會偷懶你一定有過這種經驗叫AI做一件事他做到一半停下來問你我可以繼續嗎？你要A還是B或者更糟明明沒做完但卻跟你說他做完了寫了一個漂亮的總結然後把球丟回給你你想要的就是ent可以直接做完沒做完真的不要回來反我但他就是會停到底為什麼會這樣呢？Anthropic在2025年底發了一篇研究他們發現LM做到一半會停下來的根本原因叫做contextanxiety翻譯成中文就是上下文焦慮用白話文解釋的話就是LM在執行任務的時候會一邊看自己的contextwindow用了多少當他感覺到c快滿了就會開始慌然後就莫名其妙開始rapup想要快點交菜了事然後就停了這是刻在LM基因的一種墮性我稱之為下班心態而G這個功能就是為了對抗這種墮性而生的接著我們聊聊的運行原理GO在工作的時候通常會有兩個角色協作一個是做活的實作者另一個是負責確認產出品質以及任務進度的評審實作者負責執行你的指令產出東西平審在每一輪結束後都會檢查一次用戶給的目標完成了嗎？只要答案是沒有平審就會點出問題並教師作者繼續往下就像你把一根胡蘿蔔掉在豬鼻子前面豬還沒吃到那根胡蘿蔔它就不會停下來而你只有在這隻豬真的到達終點的時候才會給他吃胡蘿蔔夠這樣的功能讓你不再需要每三分鐘跑回來檢查吹他戳他他讓agent有能力自我邊測自己跑到你給他訂好的終點講到這邊夜界內的觀眾應該會聯想到去年中有一個蠻火的插件也在做一樣的事情叫做RoughLoop這這個插件的命名我覺得很有意思R就是新普生家庭裡那個有點傻但永遠不會放棄的小孩整個插件的精神就是無論如何都會持續做到底的循環當時這個插件也在AI邊程掀起一陣風潮而今天這三家公司把這個理念變成了官方feature接著我們說一下具體怎麼使用G這個功能其實這背後沒什麼大災問就在ClockCODCcodex或者MAgent的對話框打上斜線購然後說明你希望它能完成的任務就好當然還有一些更進階的設置像是限制它可以使用token的上線等更詳細的使用說明各位可以查閱官方文檔我會把連接放在資訊欄功能的使用方式很簡單暴力但真正的問題是要怎麼寫這個提示詞才能讓GO跑出正確的東西而不是另外一個AIslop如果你很隨性的丟一個prompt像是把這個專案改得好一點AI會做一兩個小改動然後說我已經讓它變得更好了五分鐘內就把這個任務完為什麼？因為好一點是一個沒有界的目標AI不知道什麼叫做好一點它就只能猜只能自己定義而他定義的definitionofD你通常不會滿意再看一個寫的好的例子把網站結賬頁面的反應速度降到0.2秒以內用速度測試工具驗證過程中其他的功能要保持完好無缺只能改結賬這個區塊的程式碼跟相關測試別的地方不要動沒改一次就記錄下你改了什麼測出來的速度是多少下一個最值得試的方向是什麼如果速度測試工具跑不起來或者所有想得到的方法都試過了那就停下來告訴我你試過什麼卡在哪需要我給你什麼資訊才能繼續你開始差別了嗎？第二個版本裡面有五個關鍵的元素第一是outcome也就是這個任務如果完成的應該是什麼狀態在這個例子中是反應速度0.2秒以內第二是verification怎麼證明它真的完成了也就是用速度測試的工具驗證第第三是constraint哪些事情你不能做也就是除了結賬頁面以外的其他所有功能第四erationpolicy也就是每次嘗試之間要做什麼我的例子是要求AI記錄改了什麼結果是什麼還有可能的下一步第五是errorhandling,AI在什麼狀況下要暫停並回報而不是無腦的一直做下去一份好的Gprompt的設計關鍵不是promp都會寫而是你把所謂的完成定義的有多明完成的定義寫得好它就會一直跑跑到完成為止而且完成後是你要的樣子定義寫的爛AI就吵結束三分鐘就收工產出不是你想要的不如不要浪費這些oken講到這邊你可能會說那寫程式的人很爽因為測試通過反應速度低於0.12秒或者是令乾淨這些都是明確可驗證的標準但大部分的白領知識工作不是這樣你的圖好不好看你的網頁有沒有品位你的文章寫的好不好你的產品好不好用你做的影片是不是有質感這些東西沒有單元測試也很難定義何位過關何位失敗那這種佔了我們白領工作絕大多數比例偏向直化的工作該如何套用一樣的邏輯讓AI可以長時間替你在回答這個問題之前我想先說一下ropic的另一篇研究他們讓Cloud去設計一個漂亮的網頁然後也運用了剛剛提到的一個執行者搭配一個平審的架構去設計這個網站但網頁設計的票不漂亮是一個極度主觀的事情你叫AI自己憑自己做的網頁漂不漂亮？明明做的超爛超醜滿滿的機油味它還是會把自己的設計定義成現代感高質感所以ic把漂亮的網站這個模糊的概念拆成了四個明確的維度第一個維度是設計品質整個網頁有沒有像用戶傳達出一個整體的設計語言顏色字體排版有沒有共同營造出一種獨特的氛圍跟識別感第二個維度是原創性有沒有刻意的設計選擇還是用了一堆預設的模板anthropic團隊羅列了一些AI很愛用的設計元件比方說一張卡片上面有建成的顏色這種太通用的模板就會被判定沒有原創第三個維度是技術執行字體階層兼具配色對比這些技術細節有沒有整齊保持一致如果每個頁面的標題字體大小都不一致那這一關就沒過第四個維度是可用性這個維度不管沒感存看實用性比方說使用者看得懂這個界面在幹嘛嗎？找得到主要按鈕嗎？能不能很直覺的就完成他們進來這個網站的最初目的更有趣的是他們在這四個維度之上還會故意加中Cloud平常做不好的那些維度的權眾例如他們發現Cloud在技術執行跟可用性這兩個維度通常做得不錯但在設計品質跟原創性這兩個維度上常常產出平用到不行的網站所以他們就把平審評分的權眾故意往這兩個弱像偏每個模型都有自己的一些傾向而你設計的評分標準可以當做一個benchmark叫正模型的預設行為讓模型朝著你想要的方向前進定完前面講到的幾個維度後他們把這份rubric位給評審並讓平審自己去看實際做出來的網頁然後用playri打開瀏覽器自己截圖然後打分不是讓他看程式碼是讓平審使用者真的會看到的畫面Anthopic用以上的評估架構去做了一個美術館的網站跑到第九個iteration的時候Cloud做出了一個還不錯的東西深色主題的landingpage版面乾淨視覺精緻但基本上就是你預期的美術館網站長的樣子但到第十輪它把整個網站重新想像成一個空間體用cs透視渲染出一個3D房間地板是黑白方格藝術品像在真實化狼一樣掛在牆上不規則排列觀眾不是用scroll跟click在頁面之間跳而是穿過虛擬的門走進不同廳這就像是Cloud突然被反骨附身了一樣他們的研究人員也說這是他從來沒看過不可能從單一次prompt產出的創意月千而且這個月不是現性的並不是每一輪都比上一輪好有可能第10輪的產出反而遠遠比第15輪的產出更漂亮更有創但只要評審跟執行者繼續對話下去複雜度會增加野心會增加然後在某幾輪會出現那種我自己都想不到的飛躍所以你看出來了嗎？不論是G這個功能的設計或者是enthropy的研究又或者是前陣子AndreCarpathy推出的AUTCH其實全部都指向同一件事那就是evaluation不是promptengineering不是contextengineering而是你身為使用者能不能定義清楚什麼叫做得好並請他根據你的指示去做評分evaluation代表的是一個評分標準也是審合者評估者在每一輪的疊帶中都要守住的原則就是有這些原則跟評估標準審核者才能逼執行者不斷疊帶並朝著你真正想要的結果前進最後我想聊聊那我們一般人究竟要怎麼把自己工作領域裡的品位拆解成AI可以follow的準則呢？這一題我自己也摸索了很久最後收斂成了一套六個步驟的SOP在這邊分享給各位第一步是先讓AI做一輪你的工先不要急著寫rubric或者任何評估標準如果你要請AI寫作先丟五到10跟你想寫的主題進去讓它隨便跑這是在測驗你使用的這個AI現在的基準能力也就是所謂的baseline第二步是親自看一遍AI的baseline產出每一個都看然後去感受哪些你看會皺眉更重要的是皺眉的具體原因是什麼？是因為開頭沒有吸引人注意的hook沒有提供具體例子闡述你要說明的概把這些皺眉的理由寫下來這些會是你rubric的雛形到最後你會有一份清單裡面寫著AI才過的地雷像是第一句話又是在這個快速變化的時代整片都在用一般人不會用的成語或者文章裡面沒有任何具體的人民或數字第三步是把這些東西給分類拆成幾個不同的維度就像剛剛提到anthropic的做法一樣這樣之後的AI平審就可以根據你分類出來的維度去打舉個例子你列出了50條皺眉的理由可能最後收斂成三大類第是邏輯鬆散這個分類的定義是文章邏輯有斷成前文跟後文接不起來第二類是沒有人位定義是寫出來的內容沒有作者個人視角沒有具體例子在文章中一直使用像是破折號這類人類不會使用的標點符號第三類可能是文章開頭沒Hook第一句話就讓人想關掉根本讀不下去這三個維度就是你寫作rubric的股價我們我們也回顧一下anthropic是怎麼做的他們也是先看了一堆C做的網頁把皺眉的點核最後收斂出了設計品質原創性技術執行還有可用性這四個維度一模一樣的動作只是不同領域而已第四步是把每個維度整理成具體的案例當做reference這一步是整套rubric的核心你不能寫避免AI這種抽象描述要寫絕對不要用破折號絕對不要用不是A而是B這種劇型你的案例要足夠具體平審能夠一少文字就馬上幫你抓戰反或許你覺得這樣講還是太抽象沒關係我們一起來看ropic是怎麼寫的他們在官方的frontenddesign這個skill裡面為原創性這個維度提供了幾個具體的案例像是絕對不要用interrobotoarealsystemfonts這四種字體還有絕對不要用直建層蓋在白色卡片他們不是寫要保持原創性而是通過經驗以及人的品位把那些AI一再犯的錯誤一個一個點名出來放到我們剛剛的例子上寫作的raubric完全可以照班這套寫法你所謂的沒有人位這個維度可以這樣寫成絕對不要用破折號連接兩個短句當做節奏感絕對不要用不是A而是B這種劇型絕對不要用在這個快速變化的時代在AI時代這種棋手式每一條都是平審一就抓得到可以推斷的標準第五步是要用多樣化的案例去取代單一的案例描述在blog裡面自爆過一個血類教訓他們一開始在Rubric寫了一句話請設計成博物館等級的質感結果跑出來所有產出都變成博物館風非常的單一且沒有多樣性後來他們把這句刪掉改成在文件裡列出11種美學風格比方說brutalistartdecopastalindustrial還有retrofuturistic然後讓Cloud在設計時根據當下狀況選擇其中一種確保產出的多樣性有實作經驗的朋友應該都知道AI很容易因為你給的範例而有overfitting的狀況發生也就是過度遵從你給的範例所以你要寫多個方向才能確保多樣化激發AI創意第六步也是最後一步就是把你的rubric為給平審ent直接跑看產出如何如果你有紮實的執行完前五步這個時候你應該已經有一份相當紮實的rubric下一步你所當然的就是把這份rubric放進的goprompt裡面跟你的AI說這些就是他要遵循的審核標準請確保在以上rubric沒有達成之前持續疊帶不要停下但這邊有個注意事項建議你剛開始跑的時候還是要人工檢查一下看看AI執行每一輪後的產出確保平審判斷的結果跟你親眼看的結果是否一致如果不一致那大概率是你的robric寫法還沒抓到你內心真正渴望的那個平位標準這時候你要做的事情就是回去跑隔三四輪之後你就會發現你心裡對做得好的定義正在被你一條一條的梳理出來當有這種感覺的那一刻你就不再只是AI的協作者了而是能夠定義自己品位的AI管理者這就是我最近自己處理出來的一套SOP希望可以對你有幫助講到這邊也帶到整支影片最想跟你說的是寫的Rubric表面上是給AI用的但實際上它是在逼你把那些一直以來只存在你腦袋裡的模糊品位具體寫成文字一旦一旦寫成文字AI就可以幫你守住它幫你大規模執行它當初我也花了大量的時間在摸索如何定義自己的品位所以我準備了兩個提示詞第一個提示詞可以幫助你把過去AI的壞產出收斂成AI平審可以拿來打分的Rubric另一個是回到影片最開始提到的購功能這個提示詞幫你把一個模糊任務改寫成AI可以長時間執行的GO提示詞包含完成標準驗證方式限制條件可以動什麼不能動什麼以及卡住時要怎麼回報有需要的朋友可以在影片下方資訊欄查看完整文章還有領取提示詞看完這支影片後我給你的小練習是挑一件你最常做又最需要你個人品位的任務可能是寫貼文回客戶email做行銷圖寫產品文案又或者是影片剪輯進下新來花30分鐘去跑以上六個步驟試試看能不能讓AI的產出更穩定更貼近你
