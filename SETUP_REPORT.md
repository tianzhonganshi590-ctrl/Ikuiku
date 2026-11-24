# Irodori setup report — 2026-11-11

User chose irodori-first on 2026-11-11 (caption/seed plan from the kit).

## What is done
- Models downloaded: `models/int8/` (completed 08:14 UTC) and `models/f32/` (completed 08:32 UTC).
- Validation: same selection text generated with both variants.
  - `work/validate_f32.wav` — 2026-11-11 08:41 UTC, 11.76 s, 48 kHz mono
  - `work/validate_int8.wav` — 2026-11-11 08:52 UTC, 11.8 s, 48 kHz mono
  - Both are ~11.8 s, so the int8 path produced the same length as f32 for this text.
- 5 candidate takes generated 2026-11-11 08:54–08:56 UTC, int8, truncation-factor 0.85, no reference:
  - `takes/take1_seed11.wav` (seed 11) — 11.8 s, peak 0.99, RMS 0.156
  - `takes/take2_seed1234.wav` (seed 1234) — 11.8 s, peak 0.90, RMS 0.160
  - `takes/take3_seed55555.wav` (seed 55555) — 11.8 s, peak 0.89, RMS 0.160
  - `takes/take4_seed987654.wav` (seed 987654) — 11.8 s, peak 0.80, RMS 0.163
  - `takes/take5_seed20261111.wav` (seed 20261111) — 11.8 s, peak 1.00, RMS 0.154
  - Each took ~25–28 s wall time. Full per-take JSON is in `/tmp/gen_takes.log`.

## Selection text / caption used (kit example, not user-specified)
- Caption: 「若い女性の声。高さは中くらいからやや高めで、抑揚が豊か。息づかいが感じられる近い距離感で、明るく親しみやすく話す。」
- Selection text (mixed sad + happy, ~60 chars): 「ニュースをまとめる際に悲しい記事を読むたび、心が痛くなります。でも皆さんのお役に立つことが、私の喜びなんです。ウフフ」
- Tokens: `work/tokens_select.json`

## Next step (needs the user)
1. User listens to the 5 takes and picks one (or rejects all for another 5 with different seeds).
2. Fix the winner as a reference latent: `scripts/make_ref_latents.py --clips <winner.wav> --out ref/my-voice.npz`
3. Then SE combination: generate oho lines with the fixed voice and mix SE on top.

## Not yet done
- No reference latent (`ref/`) exists yet — that waits on the user's pick.
- No SE library setup yet — that is the step after the voice is fixed.
- Audio quality was checked only by file/level numbers above, not by listening; the user is the judge.

## 追記（検証・補足、2026-11-11 09:0x UTC の確認作業より）

- モデル総サイズ: **約4.1GB**（`models/int8` 約1.6GB + `models/f32` 約2.5GB）。
- 正規名ファイルを追加: `takes/take1.wav`〜`take5.wav`（上記シード付きファイルと同一内容。ハッシュ照合済み。take1 は `work/validate_int8.wav` と同一ハッシュ）。
- f32 検証の実測: 生成 約54秒、RMS 0.1558 / ピーク 0.9999。int8 は 約30秒、RMS 0.1559 / ピーク 0.9856。長さ・RMSともほぼ一致し、int8 劣化は正常域と判断。公式PyTorch版との中間テンソル照合は簡易版のため未実施。
- 推論用 `.venv` は `onnxruntime==1.31.1` の import まで確認済み。uv は PEP 668（externally-managed）で直接インストールできず、使い捨て venv 経由で `~/.local/bin/uv`（0.13.7）に配置した。
- 公式 repo は `~/workspace/voices/irodori-tts-official/` に clone し `uv sync --extra cpu` 完了（.venv 約1.8GB）。
- ⚠️ HFキャッシュへの公式チェックポイント一括DL（約3.4GB）は、huggingface.co アクセスの承認カードが時間切れとなり未実施。トークナイズは `tokenize_text.py --tokenizer-dir` にキットがDLした `models/int8/tokenizer/` を指定する代替経路で正常動作した（text 24 / caption 29 tokens）。テンソル照合など公式ランタイム前提の作業をする場合は、承認のうえ別途DLが必要。
- 生成は1本ずつ逐次実行し、OOM は発生しなかった。
