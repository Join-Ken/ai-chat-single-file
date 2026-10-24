
如果世界不需要這層，這段規則不要寫進 rpgPrompt，state 也不要加這三個字段。省略就是「不啟用」，不是「留空」。

════════ 開場消息 ════════

- 一條 assistant 消息，content 就是開場正文。
- 正文必須乾淨：不要寫 [LORE_ADD]、[STATE]、[ROLL]。導入時不解析標籤，它們會原樣顯示在氣泡裡。
- 場景必須和 state 一致：寫的是 location 那個地方，出場的人就是 present 裡的人。
- 停在玩家可以行動的地方：一個選擇點、一個問題、一個正在發生的動作。
- 不複述設定清單，直接進入場景。
- 篇幅一個場景節點，不要一口氣推進很多事。
- 工具每輪只把 state.location / state.present 注入給主持人，day 與 date 不會自動注入。開場正文裡直接寫明時間（例如「第一天，弑神歷323年冬」），讓主持人第一輪就知道時間起點。

════════ 硬性要求 ════════

以下規則是你產出世界包時必須逐條核對的。它們不是給主持人的規則，不要抄進 rpgPrompt；主持人只需要 rpgPrompt 裡那幾條敘事規則。

- JSON 必須能被工具直接導入：version 3、conversations 數組、每個對話有 id 和 messages。
- conversation.id 建議帶世界名前綴。導入時若與現有對話 id 重複，工具會自動改名生成新對話，不會覆蓋，但會多出一份。
- rpgOn 必須是 true。
- sysScope 必須是 "isolated"，system 放完整 System 提示詞。
- lore 必須是數組，不是對象。
- lore[].cat 必須是六個英文枚舉之一：character / location / faction / species / item / event。不要用中文、不要用 other。
- lore[].num 必須是數字且不重複；寫成字符串會被重新編號，重號會讓後續 [LORE_UPDATE] 對不上目標。
- lore[].key 在同一對話內必須唯一。
- lore[].name 不能重複。
- **lore[].keywords 必須是字符串數組（不是逗號分隔字符串），元素不能是數字或對象，2–5 個，且必須包含 name 全名。寫成字符串會讓工具在生成設定庫索引時報錯，整輪請求失敗。**
- lore[].enabled 必須是布爾；省略視為 true；寫成字符串 "false" 不會禁用。
- state 必須是對象，不是數組。
- state.location / state.date 必須是字符串；state.day 必須是數字。
- **state.present 必須是字符串數組，元素是字符串（不是對象、不是數字）；無人在場寫 []。寫成字符串會讓場景欄不顯示、第一輪也不命中角色。**
- state.location / present 的字符串必須包含對應條目 keywords 裡的至少一個詞（keyword 是 state 字符串的子串）。最穩：填 name 全名，且全名也是 keywords 的一個元素。
- 開場消息正文不含任何方括號標籤。
- 開場場景必須落在 state.location，出場人物必須來自 state.present。
- 不要預置 user 消息；第一句話留給玩家。
- 設定庫條目要求主持人逐條原樣輸出到 [LORE_ADD] 的舊流程已廢棄，條目直接寫在 lore 數組裡。
- 如果啟用 rumors / attention / witnesses，三個字段必須是字符串，不是數組或嵌套對象；初始值寫「尚無」或「機構0」格式；值內不要用逗號頓號；並在 rpgPrompt 末尾加對應的跨回合追蹤規則。

════════ 不要做的事 ════════

- 不要讓玩家手動貼 System 或主持人提示詞——世界包裡已經有了。
- 不要讓玩家發「設定輪輸入包」或說「開始」——導入後第一句就是玩家行動。
- 不要在 messages 裡放帶 [LORE_ADD]/[STATE] 的 assistant 消息——導入不解析，標籤會原樣顯示。
- 不要把世界骨架寫進 System 或主持人提示詞——放 lore。
- 不要在 System 裡手寫設定庫索引——工具會自動注入動態索引，手寫的那份是冗餘且會過時的。
- 不要在 system 或 rpgPrompt 裡抄工具自動注入的設定庫 / 狀態卡 / 骰子指令。
- 不要用 other 分類；歸不進六類的不要記錄。
- 不要讓 state.location / present 和條目關鍵詞脫節；對不上等於第一輪白開。
- 不要只把名字寫在 name、忘了放進 keywords；工具只認 keywords。
- **不要把 keywords 寫成逗號分隔字符串或包含數字/對象**——工具對 keywords 做數組操作，寫錯會讓整輪請求直接報錯。
- **不要把 state.present 寫成字符串（如 "卡爾"）或對象數組**——必須是字符串數組，否則場景欄不顯示、第一輪也不命中角色。
- **不要在 lore[].content 裡包含 [STATE]、[LORE_ADD]、[LORE_UPDATE]、[LORE_DELETE]、[ROLL] 這類工具標籤**——後續主持人 UPDATE 時容易照抄，導致標籤嵌套解析錯亂。
- 不要預置玩家消息，也不要替玩家決定第一步行動。
- **不要把 rumors / attention / witnesses 寫成嵌套對象或數組**——工具能存，但主持人每輪手寫嵌套 JSON 會漏、會錯、會停在初始值。一律用字符串，值內不用逗號頓號。
- **不要在沒啟用跨回合追蹤時，把那段規則塞進 rpgPrompt**——省略就是「不啟用」，不是「留空」。

若玩家需求有空白，直接問，不要自行補默認值。
