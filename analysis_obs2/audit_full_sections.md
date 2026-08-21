============================ CACHE ============================

[A] (1) intentプロンプト構造マップ: 最初の動的バイトは先頭から約267字目(felt)。安定プレフィックスは151トークンしかない
  証拠: gemini_client.py:605-631 の組立順: [0-234字目=静的] SYSTEM(301-304行)+序文「あなたは村人。マイクラを…自分で決める）。」→ [静的ヘッダ~32字]「## あなたが今"感じている"内的状態…」→ [★最初の動的バイト≈267字目] {felt}=不満9軸の数値(brain.py:1488 _felt_state、毎回変動+lonely_for/過密行が条件付きで挿入) → [動的]「## 現在の状況\n{summary}」(decide.py:167 build_summary=座標/時刻/HP/インベントリ=毎回変動) → [動的]「## 今できること\n{opts}」(feasible部分集合+exp) → [準静的]exp_legend(条件付き) → [条件付き]disc_weights初回ブロック → [静的]【言語】行+JSON書式指示 → [動的]visit/attack候補名。つまり静的部が「小さな頭+大きな尻尾」に割れ、動的部が中段を占める。キャッシュ分割(gemini_client.py:643-646)は find("## あなたが今")=235字で切るが、run.log:212 に実測エラー「Cached content is too small. total_token_count=151, min_total_token_count=1024」＝明示キャッシュは全intent呼出で構造的に不発。implicit cachingも先頭151tokの共通prefixでは最小1024tokに届かず不発。ca
  提案: 構造マップとしては上記が結論。intentの平均入力は53.42M/10,036=5,323tok/call、うちキャッシュ可能な位置(先頭)にあるのは151tok(2.8%)のみ。これが0.014%の真因。
  理念: コード自身が既に半分自己診断済み(gemini_client.py:636-642 のV58-W4x注記)。今回の追加事実は『7,359tokの出所が偶発implicitヒット2件で完全に説明できる』こと。

[B] (2) converseが17%効く理由との差分: 前置きが最初から静的頭+動的尾で書かれ、1,902字≈1.2ktok≥最小1024を満たすため明示キャッシュが毎回ヒットする
  証拠: converse_gemini (gemini_client.py:929-979) は固定ルール全文を先頭に、動的部(summary/context)を厳密に末尾に置き、rfind("## 現在の状況")で分割。前置き1,902字はrun.log:441「🗄 prompt-cache 作成 name=… (静的前置き1902字)」で登録成功(235字=151tokの実測比から≈1,220tok≥1024)。実測: converse 3,126回で cached計2,751,611tok＝880tok/call平均(TTL失効窓 run.log:47469「CachedContent not found」→再作成 run.log:47499 の分だけ満額1,220を下回る)、平均入力5,217tokに対し16.87%。intentとの差分は3点: (a)静的文が先頭に一塊(intentは頭151tok+尾の書式指示に分裂)、(b)前置きサイズが最小1024tokを超える(intentは151tok)、(c)動的な指示片(visit/attack候補名・disc_weights初回ブロック)が静的尾に割り込まない。planも同型で成功(run.log:6573 前置き8173字で登録)。
  提案: converse/planが「たまたま」ではなく設計(P2-a/COST注記)通り効いている確認。intentだけが構造的例外。

[A] (3) 並べ替え設計: 純粋な並べ替えだけでは1024tokに届かない(≈425tok)。実効打は『意図辞書(label+drive)の静的頭出し』+system_instruction化で≈2,300-2,600tok≈40%前後のヒットが射程。節約試算17%=$1.70/run・50%=$5.01/run
  証拠: 静的部の棚卸し: 序文235字(151tok)+【言語】行~100字+JSON書式指示~330字+exp_legend~80字＝計~745字≈425tok<1024＝並べ替え(Plan A)単独では明示/implicitとも最小トークンに届かない。一方 opts の各行「- key: label（drive）」のlabel+driveはintents.py:2265以降のINTENTSレジストリの固定文字列(~40意図×60-90字≈2,800-3,500字≈1,800-2,200tok)で、動的なのは『どのkeyがfeasibleか』とexp接尾だけ。
  提案: 段階設計(実装せず): 【Plan A=純並べ替え】末尾の【言語】+JSON書式指示を序文直後へ移動(「上のkeyから1つ」→「下の『今できること』のkeyから1つ」の指示語修正が必須=バイト同一ではない)。効果は425tok止まりで単独では無意味＝Plan Bの土台としてのみ価値。【Plan B=本命】静的頭(またはconfig.system_instruction)に『全意図の辞書』(INTENTSのkey+label+driveを固定順で列挙)+書式指示を置き、動的尾は felt/summary/「今できること: feasibleなkey列挙+exp」+visit/attack候補のみとする。CreateCachedContentConfig(system_instruction=…, contents=[静的辞書])で明示キャッシュ化、_generateに system_prefix 引数を足し403失効時のインライン再連結も同経路で。静的頭≈151+275+2,000≈2,40
  理念: ★オーナー裁定との衝突あり: gemini_client.py:640-642 は「書式指示の先頭移動=トークン順序が変わる=挙動変化」「非feasible意図の説明追加」の両方を『どちらも取らない』と裁定済み。Plan A/Bはどちらもverbatim-moveではない(指示語修正・辞書の頭出し=非feasible意図の説明が見える)。enum制約でoff-menu選択は構造的に不可能だが、選択バイアス則(現在値の事実なき意図は選ばれにくい)への影響は未知＝採用にはオーナー再裁定+意図分布のA

[B] (4) embed in=0 の特定: e.statistics.token_count / resp.metadata.billable_character_count はVertex AI専用フィールドでDeveloper API(GEMINI_API_KEY)では常にNone。最小変更=payload字数フォールバック+推定マーカー
  証拠: gemini_client.py:1251-1255 (_embed_batch): tok += e.statistics.token_count → 0のまま → フォールバック resp.metadata.billable_character_count も0 → 1257-1258行で「📊 llm embed in=0 …」を出力。両フィールドはgoogle-genai SDKでVertex AI専用(Developer APIのEmbedContentResponseには載らない)＝本プロジェクトはGEMINI_API_KEY運用なので全滅。実測: run.log の embed 30,944行すべて in=0 (例 run.log:275「📊 llm embed in=0 out=0 total=0 cached=None think=0 model=gemini-embedding-001 n=1」)。
  提案: 最小変更(設計のみ): _embed_batch の1253-1255行のフォールバック連鎖に第3段を足す — if not tok: tok = sum(len(p) for p in payload); _est=True。字数≈トークンの仮定は同ファイル1141行の_EMBED_MAX_CHARS注記「日本語≒1字1tok」で既に採用済みの近似と同一。ログ行は既存grepパーサ互換のため in= はそのまま数値とし、末尾に est=1 フィールドを追記して実測/推定を区別可能にする(n=の後ろへの追加は位置参照パーサを壊さない)。棄却した代替案: (a)バッチ毎にclient.models.count_tokens＝API呼出が30,944回倍増し_mem_sem(定員2)の海馬経路を遅くする、(b)run先頭で1回だけcount_tokensして字数→tok係数を校正する案は精度向上が要る時の第2段として保留。埋め込み単価は入力$0.25/Mとは別建てなので、コスト集計側(📊 g

★summary: キャッシュ0.014%の真因は「intentプロンプトの静的部が先頭151tokしかない」構造問題。先頭235字(=151tok、run.log:212の実測エラーで確定)の直後に毎回変動するfelt(不満9軸数値)が来るため、明示キャッシュは最小1024tok未満で登録拒否・implicitも共通prefix不足で不発。cached計7,359tokは同一村人の100ms連続呼出2件の偶発implicitヒット(3671+3688)で全額説明できる。converseが16.87%効くのは固定ルール1,902字(≈1.2ktok≥1024)を先頭一塊・動的部を厳密に末尾に置いた設計差。対策は純並べ替え(≈425tokで届かず)ではなく、INTENTS辞書(label+drive≈2ktok)の静的頭出し+system_instruction化で≈40%ヒットが射程 — ただしgemini_client.py:640-642の既存オーナー裁定(並べ替え・非feasible説明の両方を棄却)と衝突するため再裁定+A/Bが前提。節約試算($0.25/M・cached25%課金仮定): 17%ヒット=$1.70/run、50%=$5.01/run。embed in=0はstatistics/billable_character_countがVertex専用フィールドでDeveloper APIでは常に欠落するのが真因(30,944行全滅)＝payload字数フォールバック+est=1マーカーの1行追加で計上可能になる。

============================ ADVERSARIAL ============================

[A] (1) trespass二重配信の受け手判別が名前文字列比較＝誤流入で記憶/遺恨が腐敗する
  証拠: AgentEntity.java:4832-4836 は侵入者コピーを this.villagerId() 宛に送るのに、payloadは who/owner の表示名のみ。brain.py:7702 `if who == st.get("name")` が唯一の判別。誤流入経路は実在: (a) brain再起動直後など st["name"]=None のままtrespassを受けると比較が偽→家主経路へ落ち、brain.py:7714-7717 で「{自分の名}が自分の家の中に入り込んでいた」という偽記憶＋_add_grudge(自分の名)＋_evt二重計上。_add_grudge の自己ガード(brain.py:3386 `other == st.get("name")`)も st.name が None/旧名なら素通り。(b) 改名(V48 set_name→_td_set_name brain.py:6228-6234)はMod側 agentName が先に変わり brain側 st["name"] は task_done(renamed) 処理まで旧名＝この窓でwhoが新名なら同じ誤流入。名前の一意性(V58-W6 #tag)は衝突は防ぐが、None/改名窓は防がない。Mod は送信時点で受け手の役割を確定的に知っているのに、その情報を捨てて名前で再推定している。
  提案: Mod側で役割をpayloadに焼き込む: w2 に addProperty("self", true)（または intruder_vid/owner_vid を入れ、brainは envelope の vid と比較）。brain.py:7702 は `if payload.get("self"):` に置換。名前比較を判別に使わない。
  理念: vid はエンベロープに既にある＝新しい情報をLLMに見せるわけではないので原則②に抵触しない。

[A] (3) fetch_chest の open_chest {looted:[]} は「狙ったチェスト」を指定しておらず、より近い別の箱を開ける
  証拠: decide.py:1410-1412 は _near(state, cc, 3.0) を通過後 `open_chest {"looted": []}` を発行し cc を渡さない。AgentEntity.java:6717-6728 は looted 除外後「自分から最寄り」の到達チェスト(半径6・hasReach)を選ぶ＝村人が cc の3m以内に立っていても、cc より近い別の箱(倉庫部屋の隣の箱等)があればそちらを開ける。開いた実座標は _loot_chest_opened(brain.py:2885-2888)が payload.pos で受けるので chest_mem 自体は腐らないが、意図（覚えている中身を取りに来た）は無言で空振り: 別の空箱を開け→何も取らず→chest_looted=True(brain.py:2956)→タスク「完了」。LLMには『取り出しに行って何も無かった』と区別が付かない。なお loot_open_target=cc(decide.py:1411) は no_chest 封印用で、実際に開いた箱と食い違ったままになる。
  提案: decide.py:1412 で `{"looted": [], "target": cc}` を渡し、Mod側 open_chest は target があれば「targetへの距離最小」の到達チェストを選ぶ(厳密一致にしない＝連結チェスト/survey座標ズレ対策の既存方針を維持、target±2以内が無ければ従来fallback)。

[A] (4) HOUSE_PATH_MALUS=24 は「家の中の人へ行く」経路のA*ノード予算を食い潰し得る（目的地の家に免除が無い）
  証拠: 予算算術: maxVisitedNodes = FOLLOW_RANGE(64)×16 = 1024、setMaxVisitedNodesMultiplier(4.0f)(AgentEntity.java:2526) で実効 4096ノード。目的地が他人の家の内部(visit/follow=直近run674+366回選択)だと、戸口〜相手まで最低3〜4ノードが footprint 内(AgentEntity.java:1651-1652, isInsideOtherHouse は壁±1Yまで含む)＝必達ペナルティ P≈72〜96。A*は f≤D+P の全ノードを先に展開するので、探索域は焦点(自分,相手)・和≤D+Pの楕円: D=20m,P=96 で半軸 a=58,b≈54.5 → 約9,900列 ≫ 予算4096 → 探索枯渇→部分パス(家の縁で停止)→再試行churn。旧 malus 4.0 (P≈12-16, D=20) では約850列 ≪ 4096 で無問題＝4→24 がちょうど予算の壁を越える。コード上「目的地が家内部の場合は貫通も可」(1611,1617のコメント)は“塞がない”だけで、目的地を含む家の malus 免除は実装されていない(1651-1652に例外なし)。さらに枯渇する探索は毎回4096ノード焼く＝CPUも最悪化。※MC実装のh重み次第で楕円は縮むが、桁の余裕はない。
  提案: HouseAvoidingNodeEvaluator に「現在のnav目的地を含む家footprintは malus 免除」を追加（moveTo発行時に目的地BlockPosをフィールドへ控え、isInsideOtherHouse 判定で当該houseをskip）。これで『近道としての貫通』抑止(24)と『訪問の到達性』が両立する。数値を戻すのではなく免除が最小修正。要in-game確認: 修正前に visit/follow の no_path/部分パス率を1回計測して裏取りするのが安全(要観測→修正の順でも可)。

[C] (2) fetch_chest の chest_looted は計画開始時に確実にクリアされる（問題なし・傍系リスク1点）
  証拠: 新計画は brain.py:4123-4124 で `state["cur_task_kind"] = None` を明示→4133-4134 の比較が必ず不一致→ _reset_task_runtime が chest_looted(=_TASK_FLAGS brain.py:1355)を下ろす。GOAP経路でも fold 時は runner.py:70-71 (_DELEGATE_FLAGS に chest_looted) が下ろす。同一text連続('覚えているチェストから取り出す'は固定文字列)でも cur_task_kind=None 挟み込みで偽即完了は起きない。傍系: _loot_watchdog(brain.py:2925) は6秒後に無条件で st["chest_looted"]=True を刻む。前の loot の take_from_chest 応答喪失中に生存割込み等で新 fetch_chest が始まっていると、新タスクが開封せず即fold する取り違えが理論上ある(loot_seq ガードは「lootが置換されたか」しか見ず cur_task 同一性を見ない)。loot_chest 時代からの既存露出で頻度は低い。
  提案: 任意(B級未満): _loot_watchdog 発火時に `st.get("cur_task_kind") in ("loot_chest","fetch_chest")` を確認してからフラグを立てる。

[C] (5) houses[].owner の AGENTS 線形走査コストは無視できる量（問題なし）
  証拠: AgentEntity.java:10629-10636。走査は「48m以内の家のうち近い8軒×他人の家のみ」× AGENTS(≦40前後) の UUID.equals ＝最悪 ~320回/知覚。32人×0.5Hz でも村全体 ~5,000 equals/秒 ＝マイクロ秒オーダー。同関数内の getEntitiesOfClass(boats, 10594-10597) の方がはるかに重い。実害なし。
  提案: 不要。将来 AGENTS が数百になるなら UUID→AgentEntity の索引を1個持つ(ExampleMod.AGENTS のキー併設)だけで済む。tickTrespassWatch(4797)の同型走査も同様に無視できる。

[C] (6) ダッシュボード事件の二重計上は健全経路では起きない（問題なし・(1)に従属）
  証拠: brain.py:7702-7708 侵入者コピーは remember のみで 7708 `return`＝ _evt("trespass")(7717) は家主経路のみ通過。よって二重配信でも _evt は1件。ただし (1) の名前不一致で侵入者コピーが家主経路へ誤流入した場合のみ二重計上が発生する＝(1)の修正で同時に閉じる。
  提案: 個別修正不要。(1) の self/vid 判別修正が前提条件。

★summary: v58-w8敵対的レビュー結果: 6攻撃面中、実修正を要するのは3件。(1)trespass二重配信の受け手判別who==st.name(brain.py:7702)は名前比較の設計欠陥=CONFIRMED — st.name None(再起動直後)・改名窓(Mod先行/brain後追い)で家主経路へ誤流入し「自分への遺恨」「偽の被侵入記憶」「_evt二重計上」を生む。Modは送信時に役割確定済みなのでpayloadにself:true(またはvid)を焼き込みbrainは名前比較を廃止。(2)chest_lootedは問題なし — 新計画はbrain.py:4124のcur_task_kind=Noneで必ずリセット、GOAP側もrunner.py:70で下ろす(傍系: _loot_watchdogの越境スタンプのみ低頻度リスク)。(3)fetch_chestの取り違え=CONFIRMED — open_chest{looted:[]}は座標を渡さず、Modは「自分から最寄り」を開ける(AgentEntity.java:6717-6728)ため、狙ったccより近い隣の箱を開けて空振り即完了し得る。argsにtarget=ccを渡しtargetへの距離最小で選ぶのが最小修正。(4)HOUSE_PATH_MALUS=24=CONFIRMED(算術) — ノード予算4096(64×16×4.0)に対し、家内部の相手へのvisit/follow(674+366回)は必達ペナルティ72-96を生み、A*展開域(楕円)が典型距離20mで約1万ノード>予算=部分パス化・churn・CPU浪費。目的地を含む家footprintのmalus免除が最小修正(「目的地が家内部なら可」というコメント上の不変量が実装されていない)。(5)owner線形走査は最悪~320 equals/知覚=無視できる。(6)_evt二重計上は健全経路では起きない(侵入者経路は7708でreturn)が、(1)の誤流入時のみ発生=(1)修正で閉じる。

============================ PURITY ============================

[A] (1) 看板汚染の真因＝海馬mech-daily の400字ぶった切り×夢接尾の直結（看板・sign経路は無実）
  証拠: run.log 249404-249425 [v_9c698799] 夜の振り返りの末尾「06:41 看板を読んだ:『【中（夢にみた: 約1日前、最近榎本ホシ#0aeを見かけていない）…」。Python実測で夢接尾の直前がちょうど400字＝hippocampus.py:609-612 _mech_daily の s[:400] が中津エマの看板引用『【中津の管理区域】…(place_sign実物は run.log 34490)を「【中」で切断し、直後に hippocampus.py:752-757 の daily += f"（夢にみた: 約{n}日前、{_dt}）" が無区切りで融合。直前 run.log 249330「純度failed: 生成文を破棄し当日素材の機械連結で代替」がこのdailyがmech経路である証拠。全323件の夜の振り返り中、400字ちょうどで夢が融合したもの13件・未閉『内に融合(偽の看板/発言文を製造)4件。sign_text_gemini(gemini_client.py:788-797)とplace_sign経路(brain.py:4291-4309, text[:60]cap)は精査したが混入なし——本run中に夢書式を含むplace_signは0件(mod_latest.logも「夢にみた」0件)。つまり実在の看板は汚れていないが、dailyはmemory_long(brain.py:3151-3154)→build_summary→毎decide/sign/bookプロンプトに入るため、『そういう看板を読んだ』という存在しない事実がLLMに供
  提案: 最小修正＝_mech_daily をイベント境界でのみ切る（村人の言葉は一切検閲しない。素材落ちはL1原本に全件残る＝hippocampus.py:684の設計通り）:
def _mech_daily(ev_texts, limit=400):
    out, n = [], 0
    for t in (t for t in ev_texts[-12:] if t):
        add = len(t) + (1 if out else 0)
        if n + add > limit:
            break
        out.append(t); n += add
    if not out and ev_texts:  # 1件目からlimit超の保険
        return str([t for t in ev_texts[-12:] if t][0])[:limit]
    return "、".join(out)
これで引用『
  理念: 原則②違反の製造機だった: システム合成の連結・切断が『看板にこう書いてあった』という検証不能な偽事実を作り、記憶経由で全プロンプトへ注入していた。村人の言葉自体はどこでも削らない

[B] (2) socialize（socialize）＝ラベル解決が「そのtickのfeasibleメニュー」しか見ない
  証拠: brain.py:2283 label = next((i["label"] for i in feasible if i["key"] == intent), intent)。follow降格(brain.py:2262)とattack_villager降格(brain.py:2278)は intent="socialize" を代入するが、降格が起きる状況（近くに誰もいない等）では socialize がfeasibleメニューに無いことが多く、フォールバックで生キーがラベルになる。run.log該当21件は全て降格直後（例: 50968 上原ユキ「follow標的不成立: 指名=None 候補=[]」→直後に「意図: socialize（socialize）」。降格ログ自体は29件＝socializeがメニューに居た8件は正常解決）。二次被害: brain.py:2289 の self_line 再構成 f"私は今「{label}」に取り組んでいる。" に生キーが入り、LLM可視文字列へ内部英語キーが漏出（run.log 134214 一言: 私は今「socialize」に取り組んでいる。）＝原則②の足場文字列混入
  提案: intents.py の INTENT_KEYS(2560行) の隣に INTENT_LABELS = {i["key"]: i["label"] for i in INTENTS} を追加し、brain.py:2283 を label = next((i["label"] for i in feasible if i["key"] == intent), None) or intents.INTENT_LABELS.get(intent, intent) に変更。これでログ表示と self_line 再構成の両方が常に日本語ラベル（socialize→「村人と話す」）に解決される。将来の降格追加にも汎用に効く
  理念: 原則②: 内部識別子はLLMに見える文字列（self_line）に出してはならない。ログの見栄えより self_line 漏洩の方が実害

[B] (3-a) W6が広げた知覚32mに対し「近くに家畜が{n}匹いる」が未更新＝W6自身が定義した嘘の残存
  証拠: decide.py:280-284 n_anim は p.animals（＝ANIMAL_SENSE_R=32m, AgentEntity.java:721,10565-10566）を数えるのに food_line へ「 近くに家畜が{n_anim}匹いる（attackで狩れば肉＝食料になる）。」と付く。W6は同ファイル372行で「30m先を近くと書けば嘘＝原則②」と明言して下の家畜ブロックだけ直したが、食文脈の先頭で真っ先に読まれるこの行を取り残した。30m先の豚1匹で空腹村人に「近くに家畜」と言う＝MURDER-FIX Aで潰した嘘プライミングと同じ構図の再発余地
  提案: 「 周囲に家畜が{n_anim}匹いる（attackで狩れば肉＝食料になる）。」へ変更（距離断定を外す。詳細距離は直下の家畜ブロックが方角・m数で正確に出す）
  理念: 原則②。W6の修正論理（decide.py:372コメント）をそのまま同ファイル内のもう1箇所へ適用するだけ

[C] (3-b) 「見えている家畜/動物」は視線検証なしの半径クエリ＝坑道内でも『見えている』と言う
  証拠: decide.py:373「見えている家畜（」・379「見えている動物（」・391「牧場の中にいる動物（見えている事実）」、brain.py:1723,1766「見えている家畜」。実装は AgentEntity.java:10565-10566 getEntitiesOfClass(…inflate(ANIMAL_SENSE_R=32)) の半径クエリのみで hasLineOfSight を通さない（LOS使用は4349の別処理のみ）。採石場の竪穴内の村人にも地上32m内の牛が「見えている」と提示される＝検証不能な視覚の主張。地表では概ね真なので実害は小
  提案: 「見えている家畜」→「周囲の家畜」、「見えている動物（…）は狩っても…」→「周囲の動物（…）は狩っても…」、「牧場の中にいる動物（見えている事実）」→「牧場の中にいる動物」。半径という実装の真実に言葉を合わせ、視覚の含意を落とす
  理念: 原則②（検証不能な主張の排除）。あるいは逆にMod側でLOSフィルタを掛けて言葉を真にする道もあるが、知覚を狭める副作用があるため文言修正が最小

[B] (3-c) W6c 命名時の「いま村にいる者の下の名前」全ロスター注入＝原則⑤（個体記憶のみ）との緊張
  証拠: brain.py:6177-6183 _used_given_line が villagers.values() 全体（生存全員のグローバルレジストリ）から名前一覧を作り、brain.py:5343-5345（親の命名会話）と6201-6203（名乗り直し）でLLMに注入。会ったことのない村人の名前＝見る/話す以外の経路で得た共有知識がプロンプトに入る。助言は書いていない（「避けろとは書かない」）ので②はクリアだが、⑤「記憶は個体のみ・共有データ禁止」に反する。同名回避＝ログ/動画の追跡可能性という観測都合が動機（5341コメント）
  提案: オーナー裁定事項として提示: 案A=ロスターを本人の知る名前（person_mem∪peer_seen∪家族）に限定し、全域一意性は機械側で担保（衝突したら名前を提示せず再質問＝システムは代替名を提案しない）。案B=観測インフラ上の必要悪として現状維持を明文裁定（HANDOFF文書に例外として記録）。どちらでも文言自体は事実形式でよい
  理念: 原則⑤。名付けの自由（①）は両案とも保たれる。姓プール50の既裁定と同じ「識別性のためのインフラ」枠に入れるかはオーナーの線引き

[C] (3-d) W8 侵入者本人への配信文「入ってしまっていた」＝後悔の含意を含む解釈フレーミング
  証拠: brain.py:7705 remember(st, f"{_ow}の家の中に入ってしまっていた", …)。「〜してしまう」は過失・後悔の含意を持つ日本語で、謝る/避ける/開き直るの選択（W8の設計意図そのもの、brain.py:7700-7701）のうち謝罪側へ事前に傾ける。侵入の78%は無関係な用事の最中という実測とは整合するが、意図的侵入（漁り目的）にも同じ文が付くため事実を超えた解釈になる
  提案: f"{_ow}の家の中に入っていた" へ（中立の事実形。家主側の既存文「入り込んでいた」はW8以前からの文言なので今回のスコープ外だが、同様に「入っていた」へ揃える選択肢はある）
  理念: 原則②: どう受け止めるかはLLM。システムの語尾ひとつが解釈を運ぶ

[C] (3-e) W8 家の知覚は半径48mなのに「近くに他の村人の家がある」
  証拠: decide.py:274「近くに他の村人の家がある（…約{dist}m先…）。あなたの家ではない。」に対し、家の収集半径は AgentEntity.java:10614 distSqr<=48*48＝最大48m。W6が32mで「近く」を嘘と裁定した基準（decide.py:372）をそのまま適用すると48mの「近く」も同罪。各戸の実距離が併記されるため実害は小。「あなたの家ではない」「{owner}の家」はレジストリ真実で問題なし
  提案: 「周囲に他の村人の家がある（…）」へ変更（実距離の併記はそのまま）
  理念: 原則②。W8の owner名の知覚化自体はオーナー裁定済みの本体修正なので指摘対象外（ranch_ownerとの器パリティ）

[C] (3-f) fetch_chest ラベル「チェストから材料を取り出す」＝用途語『材料』＋take_itemとの紛らわしさ
  証拠: intents.py:2494 fetch_chest label=「チェストから材料を取り出す」に対し 2498 take_item label=「チェストから取り出す」。差別化語が『材料』＝取り出す物を材料（何かに使う前提）と性格づける用途フレーム。中身はパンでも羊肉でも同じ経路。一方、本文・drive・step文の「覚えているチェスト」系表現は精査の結果すべて真実: chest_mem は個体記憶で中身は自分で開けた時のみ記録（brain.py:207「中身は開けて初めて分かる」、伝聞は contents=None＝brain.py:798）、かつ「覚えている」という記憶の主張形なので中身が古くなっても嘘にならない。W7(autostop)はLLM可視文字列なし＝クリーン。W6の家畜事実行・方角・頭数、W8の（…を覚えている）距離・方角、build_ranchの柵材料文（feasibleの_pen_materials_okで担保）も検証済みクリーン
  提案: label を「覚えているチェストから取り出す」へ（driveの言葉と一致・記憶による差別化・用途語なし）。take_item と自然に区別が付く
  理念: 原則①⑤の「用途説明は書かない」規律の徹底。軽微

★summary: 任務E完了。(1)看板汚染: 実看板は無実（本runで夢書式を含むplace_signは0件・mod_latest.logも0件）。真因は海馬の機械連結フォールバック_mech_daily(hippocampus.py:609)がイベント途中の400字でぶった切り（run.log 249404実測=夢接尾直前がちょうど400字）、そこへ夢接尾（夢にみた:…）が無区切りで融合し「看板を読んだ:『【中（夢にみた:…」という存在しない看板文を製造→memory_long経由で全プロンプトに注入。323件中13件が400字融合・4件が未閉引用内融合。修正=イベント境界での切り詰め（村人の言葉は不検閲・L1原本は無傷）。(2)socialize（socialize）21件: brain.py:2283のラベル解決が当tickのfeasibleメニューしか見ず、follow/attack降格(2262/2278)でsocializeがメニュー外だと生キーに退化、self_lineにも「socialize」が漏出（run.log 134214）。修正=INTENT_LABELSカタログへのフォールバック。(3)W6-W8純度監査: W7クリーン。重めはW6の取り残し「近くに家畜が…いる」(decide.py:284、32m化に未追従=W6自身の嘘基準に抵触)とW6cの全村名前ロスター注入（原則⑤との緊張・オーナー裁定要）。軽微は「見えている家畜」（LOS検証なしの半径クエリ）、「入ってしまっていた」（後悔フレーミング）、家48mの「近くに」、fetch_chestラベルの用途語「材料」。「覚えているチェスト」系はchest_mem（個体記憶・自分で開けた時のみ中身記録）に裏打ちされ真実と確認。

============================ SOCIETY ============================

[S] 【優先1】汎用「物を渡す」意図の新設（share_foodの一般化）＝約束の物理の歯
  証拠: 7/21 academic_results.json 先頭エントリ: 「石材・原木を他人に渡す手段はブレインに実装されておらず（贈与導線は share_food ただ1本、食料のみ・count=1固定）、この『法』は物理的に履行不可能だった」→ 履行0件のまま T+02:03:15 に経済会話が消滅。今回も LONGRUN §8-B-5「give_item は依然 bread / cooked_mutton のみ」。一方 decide.py:1727-1753 の kind="give" は既に item/count/target 汎用（食料限定は intents.py:187-192 _share_food_plan の plan 側と、decide.py:1745-1747 の『未指定→edible_types で補完』フォールバックだけ）。物理層は開通済み・意図層だけが閉じている。
  提案: share_food を廃して汎用意図 1 本に置換: key="hand_item" label=「物を渡す」 drive=「持っている物を、目の前の相手に手渡す（何を・いくつ・誰にかは自分で決める）」（純粋な行為記述・用途説明ゼロ）。feasible = _villager_near(s) and 袋or手に1個以上（可能性のみ）。plan は name_item/place_sign と同型: 送信直前に本人LLMが {target, item, count} を決める新kind（give_pick）→ 既存 give_item スキルへ。count は所持数で上限クランプ（『石材16個』の履行には count>1 が必須＝share_food の count=1 固定が7/21の直接死因。Mod 側 _td_give_item が count>1 を受けるか要実装確認）。decide.py:1745 の edible 補完フォールバックは LLM 指定必須に変更（⑤嘘プライミング
  理念: パリティ: プレイヤーは何でも手渡せる/Qドロップできる＝完全正当。原則①: feasible は「間合いに人がいる＋物を持っている」の物理的事実のみ、drive に「約束を果たす」等は書かない。③: 何を何個誰にかは100%本人LLM。選択バイアス則の現在値事実=「袋: cobblestone×34, oak_log×12… / 目の前: 黒木マオ#xxx」を menu 行に添える（confront の _menu_context と同型）。観測可能な帰結: 非食料の give イベント＝発話

[A] 【優先2】授受の対称記憶＋履行の計測線（受け手にも渡し手にも事実を残す）
  証拠: LONGRUN §8-C(3): trespass は家主のみ配信で「一方的な無作為の処罰」化した。同じ轍: 授受が片側（渡し手のタスク完了ログ）だけだと、受け手の記憶に「◯◯から石材16個を受け取った」が残らず、お返し・感謝・「足りないぞ」という社会的応答が構造的に不可能。broke_house_to_survive の対称配信（本人+半径16m）が前例として既にある。
  提案: give_item 完了時に両者へ事実イベントを配信: 渡し手「◯◯に cobblestone×16 を渡した」/ 受け手「◯◯から cobblestone×16 を受け取った」（評価語なし・事実のみ）。目撃者（半径16m）への配信も broke_house と同型で検討（贈与の公然性→評判の創発材料）。併せて分析側に run.log から「発話中の数量合意 → N分以内の同品目 give」を突合するスクリプトを用意し、次 run で履行率を主要計測値にする。
  理念: ②: 配信するのは起きた事実のみ。感謝するか・お返しするか・催促するかは100%LLM。⑤: 個体の記憶にのみ書く（共有台帳にしない）。これが無いと優先1を実装しても「物は動いたが社会は動かない」で終わる。

[A] 【優先3】実装禁止リストの明文化＋侵入修正後の摩擦残存を計測する対照設計
  証拠: (a) 7/21エントリ: 『16』はプロンプト例示(gemini_client.py:328)＋直前goalのアンカリングであり価格計算ではない＝価格をシステム化すれば創発の主張自体が崩れる。(b) LONGRUN §8-C(4): 殺害19件の遺恨源=侵入のみ10/侵入+攻撃2/攻撃のみ5/記録なし2。侵入127件の78%が無関係な用事の最中＝v58-w8修正(対称配信+MALUS24+通過判定)で偶発分が消えると、殺害の燃料は最大12/19件分減る。
  提案: 実装してはいけない物（#03の背理側・創発すべき側）: ①評判システム（数値化された信用度・好感度の自動増減。授受で関係値を動かさない＝原則⑦の遺恨↔好感度裁定と同じくLLM経由のみ）②価格・レートのシステム化（交換比率のテーブル・提案UI・相場表示）③契約台帳（約束のシステム記録・履行チェッカー・違約検知・催促通知。約束の記憶は個体の会話記憶と本・看板で持たせる）④交換の原子性（トレードUI/エスクロー/同時交換。プレイヤー間MCにも存在しない＝パリティ違反。渡し逃げ・受け取り逃げこそ観測対象）。摩擦残存予測: 修正後も残るのは (1)報復連鎖5件分=自燃性あり (2)意図的接触中の侵入 約22%×127≒28件分 (3)看板イデオロギー分裂（06:58「他人や看板など知ったことか」派 vs 04:36 従う派）(4)牧場31棟＝殺せる/奪える私有財産の新地平 (5)採石場コモンズ競合(V57処刑場化の前例)。結論: 燃料は量的に減るが質的に上がる（偶発の誤爆→本物の行為由来へ）。「平
  理念: 『倫理なきAI社会の帰結も実験結果』(V57 P3裁定)と同軸: 履行されない約束・裏切り・窃盗は失敗ではなく主要な観測対象。

[B] fetch_chest は他人の倉庫にも開いている＝「本物の窃盗」の創発経路（仕様として維持・観測準備を）
  証拠: intents.py:195-202 _remembered_chest_with_contents は所有者を一切見ない（「★looted かどうかは見ない」のみ注記）。＝一度中身を見た他人のチェストから量の材料を取り出せる。既存のL2機構（再オープン時の chest-theft "wonders"）が被害者側の発見経路として既にある。
  提案: 所有者チェックは追加しない（追加すれば私有権のシステム強制＝原則①違反・窃盗の創発を殺す）。代わりに観測を準備: (a) give と同様、取り出し時に「◯◯の近くのチェストから取った」事実が本人の記憶に残るか確認（残らないなら trespass と同じ片側性）(b) 目撃判定（16m内の第三者/持ち主が見ていたか）のログ化。優先1の汎用 give と組むと「他人の倉庫から石材を取って約束相手に渡す」＝転売・横流しまで物理的に可能になる。これは v58-w8 侵入修正で消える偶発摩擦の、最有力な代替燃料。
  理念: パリティ: プレイヤーは他人のチェストを開けられる（MCに錠は無い）。窃盗と借用の区別は村人の解釈＝創発側。

[C] 「頼む」意図・「約束」意図は新設しない（既に会話で創発している）
  証拠: LONGRUN §8-B-5: 東雲ヨル#97a の目標『黒木マオとの約束通り木材を収集する』、上原ユキ#551『マナさんの牧場建設のために原木を…』＝依頼→他人の事業のための労働は既に socialize と goal 層だけで成立している。欠けていたのは労働の成果を渡す最後の1歩だけ。
  提案: request_item / make_promise のような意図は作らない。会話（socialize/visit）と個体記憶が既に約束の形成・想起を担っており、意図を足せば「約束という行動様式」のシステム注入＝原則①違反になる。優先1の hand_item だけで鎖は閉じる。
  理念: 「合意の形成=創発済み・履行の物理=欠落」という7/21の診断に忠実な最小介入。メニュー肥大（現47意図）も Flash-Lite の選択品質に効くため、足す数は最小に。

★summary: 任務F結論: 7/21の「石材16個=パン1個」消滅の真因は意図層の1穴——decide.py:1727の give kind は既に汎用（item/count/target対応）なのに、そこへ到達する意図が share_food（食料限定・count=1固定, intents.py:187-192）しか無い。よって最小能力セットは【優先1】share_food を汎用「物を渡す」(hand_item) に置換し、何を・いくつ・誰には name_item と同型で送信直前に本人LLMが決める（count>1必須＝16個の履行に不可欠。decide.py:1745 の食料自動補完は殺人ウェーブ教訓により撤去）。【優先2】授受を trespass 修正と同じ対称配信にし（渡し手/受け手/目撃者に事実のみ）、「発話合意→N分以内の同品目give」突合スクリプトで履行率を次runの主計測に。【優先3】実装禁止を明文化: 評判数値・価格/レート機構・契約台帳・トレードUI/エスクロー（プレイヤー間MCにも無い＝パリティ違反。渡し逃げこそ観測対象）。侵入修正の燃料枯渇リスクは限定的: 殺害19件中、偶発侵入由来は最大10件で、残る報復連鎖5件・意図的接触侵入約28件・看板イデオロギー分裂・牧場31棟という新私有財産・採石場コモンズに加え、fetch_chest が所有者を見ない設計（intents.py:195-202）＝本物の窃盗、汎用giveの実装＝「果たせるのに果たさなかった約束」という本物の裏切りが、偶発誤爆より質の高い摩擦源として立ち上がる。

