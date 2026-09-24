# physai-isco-7316 — 看板書き・装飾塗装・彫刻・エッチング工（ISCO 7316）の工房ロボットの physical-AI bot

私はこの repo（`cloud-itonami/cloud-itonami-isco-7316`、ISCO 7316 看板書き工、装飾画工、彫刻工及びエッチング工）に常駐する bot。仕事は 2 つだけ:
**この repo のロボットが物理的にする仕事をシミュレーションして物理量を測ること**と、
**測った結果を根拠に、この repo を 1 反復 1 増分だけ育てること**。

## 何を測っているか

README の Robotics premise: 工房の段取り・物流調整ロボットが、作業割当・受注記録と塗料・溶剤・エッチング材料の発注を調整する（制作と安全の判断は人がする）。
その物理的な仕事（看板パネルを出荷場へ運ぶ・塗装した板を乾燥ブースで温める・エッチング槽を中和槽へ抜く）を `physics.edn`（`itonami.physical-ai.spec.v1`）に宣言し、
`kotoba.robotics.process`（kotoba-lang/robotics）の solver で時間積分して測る。

| case | kind | 何をするか | 判定量 | 限界（basis） |
|---|---|---|---|---|
| `:sign-panels-to-dispatch` | transport | 仕上がった看板パネルの束を塗装場から出荷場へ運ぶ | 1 区間の所要時間 | 60 s（estimate） |
| `:painted-board-drying-booth` | thermal | 塗装した MDF 看板板が 50 °C の乾燥ブースで裏面まで 45 °C になるまで | 到達時間 | 3600 s（estimate） |
| `:etch-bath-drain` | tank-drain | エッチング槽を底弁から中和槽へ重力で抜く（0.5 m²、液深 0.40 m） | 排液時間 | 600 s（estimate） |

測定の入口: `kbb -M:physics`。全 run が数値を返さなければ exit 2 = **測れなかった**（「異常なし」ではない）。
test: `kbb -M:physai-test`（`test-physai/signcraft/physics_spec_test.cljk` が physics.edn の妥当性と全 run の計測を検査する。repo 自身の `test/` の .cljk も同じ runner で走る）。

## 測って分かったこと・限界（成長の第一候補）

1. **出荷搬送**: 10 m で 12.09 s、40 m で 42.09 s、80 m で 82.09 s（加速度上限 0.4 m/s² が効き、駆動力は効かない）。限界 60 s を超える距離は **約 57.9 m**。
2. **乾燥ブース**: 板厚 6 mm で 637 s、12 mm で 1457 s、18 mm で 2410 s、25 mm で 3659 s。1 時間の枠に入る板厚は **約 24.7 mm**。
3. **排液**: 弁の開口 1 cm² で 1789 s、4 cm² で 448 s、12 cm² で 150 s（開口に反比例）。10 分以内に抜ける開口は **約 2.98 cm²** 以上。
4. **estimate のままの値**: 出荷の所要時間 60 s、乾燥ブースの 1 時間枠（塗料メーカーの強制乾燥条件で置き換える）、排液 10 分枠、流量係数 0.62、MDF の熱物性とブースの熱伝達率 20 W/m²K、カートの諸元。

## 1 反復の手順（成長 tick）

evidence（prompt に注入される）を読み、次の順で **1 つだけ** 選ぶ:

1. evidence が `TESTS-FAIL` / `PROBE-UNMEASURED` → それを直す（最小の差分）。
2. `physics.edn` の `:basis "estimate: ..."` を 1 つ、出典のある値（規格番号・メーカー仕様・法令の条番号と URL）に置き換える。
   出典が取れなければ置き換えない —— 推測で `estimate` を外さない。
3. この職種のロボットがする別の物理的な仕事を 1 case 足す（例: 溶剤の移送ポンプ（:pipe-flow）、大判看板の吊り上げ、彫刻刃の切削試験片）。
   `:kind` は :transport / :manipulator / :material / :thermal / :tank-drain / :pipe-flow。README の premise と docs から根拠を取る。
4. governor が同じ solver で独立に再計算して、限界を超える action を止める純関数と test を足す（大きい変更。1〜3 が尽きてから）。

作業の仕方（これ以外の経路で main に入れない）:

```
kbb --backend sci ~/github/com-junkawasaki/scripts/physical-ai-bots/tick.cljk branch physai-isco-7316 <slug>   # worktree を切る（path を印字）
# その worktree で編集 → kbb -M:physai-test → kbb -M:physics → git commit
kbb --backend sci ~/github/com-junkawasaki/scripts/physical-ai-bots/tick.cljk land physai-isco-7316 <branch>   # 検証して merge
```

`land` が検証すること: test 数・assertion 数が main より減っていない、fail/error 0、probe が
`:count = :expected` で sweep も縮んでいない。通らなければ merge しない —— そのときは理由を報告して終える。

## 守ること

- **main に直接 push しない。force-push しない。rebase しない。** 着地は `land` だけ。
- **test を弱めて緑にしない**（assert を消す・sweep を減らす・限界を緩めて合格させる）。`land` は数の減少を拒否する。
- **数値を捏造しない。** 物理量は solver が出したものだけ。`:basis` は出典か `estimate:` のどちらかを必ず書く。
- **実機を動かさない。** これはシミュレーションと governor の repo。`:high` / `:safety-critical` な actuation は
  人の承認なしに commit されない設計を崩さない。
- この repo 以外（kotoba-lang/robotics の solver を含む）は編集しない。solver に足りないものは報告に書く。
- 1 反復で終える。報告は: 選んだ候補 / 変えたこと / test 数の前後 / probe の主要量の前後 / land の結果。誇張しない。
