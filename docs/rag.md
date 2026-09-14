# Evidence-first RAG design

## Goal

「どのディストリビューターが自分に合うか」「最安はどこか」「100%還元は本当に100%か」「解約後も配信されるか」などを、比較表だけでなく一次情報の根拠付きで回答できるようにする。

## Data layers

```text
company/distributor facts
        ↓
comparison records (data/distributors.json)
        ↓
atomic evidence claims (data/evidence.json)
        ↓
retrieval / ranking
        ↓
answer + evidence_ids + source URLs + checked_at
```

## Retrieval rule

1. First filter structured fields: price, revenue share, store count, artist count, social platforms, features.
2. Retrieve matching evidence records by `evidence_ids`.
3. Never answer a volatile commercial claim without an evidence record.
4. Prefer official pricing/product/support/terms pages over secondary reviews.
5. Show `checked_at` with price/revenue claims.
6. Distinguish **artist share** from **store gross**, and distinguish **master recording royalties** from **publishing/songwriter royalties**.
7. If sources disagree, keep both claims and mark the conflict instead of silently choosing one.

## Example questions

- 「年3曲のシングルを出すならどこが安い？」
- 「TikTokを最優先にするとどこ？」
- 「日本語サポートとカラオケが必要」
- 「年間契約を避けたい」
- 「解約しても楽曲を残したい」
- 「100%還元だけを比較して」
- 「7社で、料金・還元・配信先・SNS・権利管理・分析・スプリットを比較して」

## Answer contract

Every factual answer should be returned as:

```json
{
  "answer": "natural language answer",
  "matched_distributors": ["..."],
  "claims": [
    {
      "claim": "fact",
      "evidence_ids": ["..."],
      "source_urls": ["..."],
      "checked_at": "YYYY-MM-DD"
    }
  ],
  "caveats": ["..."],
  "decision_basis": ["price", "revenue", "features"]
}
```

## Evidence quality

- `high`: official pricing/product/support/terms page directly states the claim.
- `medium`: official source supports the claim indirectly or requires interpretation.
- `low`: secondary source; use only when primary evidence is unavailable.

## Important semantic normalization

- `100% revenue` is not necessarily 100% of consumer gross; store/platform fees may already be deducted.
- `100% royalties` generally concerns recording/distribution royalties, not necessarily publishing/songwriter royalties.
- `free` can mean zero upfront cost while still using a revenue share.
- `subscription` and `per-release` are fundamentally different cost models.
- Prices are stored in the source currency. FX conversion must carry its own timestamp.

## Future implementation

A thin query API can expose `/distributors`, `/compare`, `/search`, and `/evidence/:id`. A vector index is optional: structured filtering should happen first, and embeddings should be used only for semantic questions such as "TikTokで伸ばしたい新人向け". This keeps answers explainable and avoids RAG returning a plausible but unsupported commercial claim.
