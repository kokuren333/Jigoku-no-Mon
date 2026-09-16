# Data publication policy

## Current status

The local SQLite schema stores X post text, account identifiers, profile fields,
engagement metadata, scores, and evidence links. This is suitable for local
research, but it is **not** a dataset that can be published openly as-is.

Do not publish `data/jigoku.sqlite3`, raw tweet text, raw profile data, cookie
files, or a public Git repository containing hydrated X objects.

## Safer research release

If release is needed, generate a separate sanitized artifact containing only:

- tweet IDs and/or user IDs where permitted
- derived aggregate statistics with identifiers removed or pseudonymized
- model scores and score/version metadata
- code, configuration, and a rehydration procedure

Recipients must be told that IDs can become unavailable and that rehydration
must follow the current X policies. Review legal, ethical, institutional, and
IRB requirements before release.

## Model licensing

- `TomokiFujihara/twhin-bert-base-japanese-offensiveness-estimation`: Apache-2.0
- `Twitter/twhin-bert-base`: Apache-2.0
- Transformers, PyTorch, and SentencePiece each have their own licenses and
  must be included in a release's notices.

The model license does not grant permission to redistribute X content. The
model's custom code is loaded with `trust_remote_code=True`; pin its revision
before reproducible or public distribution.

References:

- https://docs.x.com/developer-terms/policy
- https://docs.x.com/developer-terms/restricted-use-cases
- https://huggingface.co/TomokiFujihara/twhin-bert-base-japanese-offensiveness-estimation
- https://huggingface.co/Twitter/twhin-bert-base

## Seed selection protocol: adversarial review and timeline

### Review conclusion

The proposed procedure—selecting balanced topic groups, searching X, and
promoting high-exposure posts to seeds—is acceptable only as a discovery
sampling frame. It is not a uniform random sample of all X posts and must not
be interpreted as a representative sample of people, viewpoints, or danger.
It measures the reach of the selected public-post network under the recorded
search conditions.

### Adversarial review findings and controls

1. Selecting high-exposure posts creates popularity and celebrity bias. Keep
   low, medium, and high exposure strata within each topic and time window.
2. X search ranking is not random. Record query, language filter, date window,
   retrieval time, page range, and the random seed used for within-page choice.
3. Replies to a controversial post are not necessarily abusive. Keep source
   posts, replies, quotes, and native reposts distinct; include non-controversial
   comparison seeds.
4. A published topic list inherits the bias of its source studies. Fix the
   strata before collection and report counts per stratum.
5. One seed is not generalizable. Run each seed under its own checkpoint and
   report seed-specific reach and overlap.
6. Impressions are not interchangeable with public reactions. Store view count
   separately when available; otherwise label the derived value as an
   exposure proxy.

### Confirmed procedure

1. Pre-register topic strata and multiple query sets per stratum.
2. Randomly choose date and time windows.
3. Retrieve public Japanese candidates under the recorded conditions.
4. Sample within result pages using a recorded random seed.
5. Deduplicate same conversation, same author, and duplicate post IDs.
6. Stratify by low, medium, and high exposure; retain comparison seeds.
7. Store the selection reason and all search metadata in the local seed ledger.
8. Run the unchanged explore-v2 protocol per seed and report every ten
   accounts with 100 collected posts each.

### Timeline of this project

- Initial phase: tweet `2099783355248341399` was deliberately chosen as a
  single exploratory seed. It was not randomly sampled.
- Single-seed trial: roughly 70 accounts were reachable, demonstrating the
  scale of one conversation network but not population representativeness.
- Mid-project: exploration-v2 was enabled while preserving the existing
  checkpoint and data.
- Protocol phase: the topic-stratified, random-time-window, exposure-stratified
  seed selection procedure was fixed for future seeds.
- Current snapshot: the explore-v2 checkpoint records 62 processed relation
  posts, 127 queued relation posts, 79 accounts, 4,766 stored posts, 55 account
  scores, and 42 high-confidence accounts.
- Stop decision: repeated v2 cycles on the original seed yielded no new
  accounts, so the original seed's supply was considered exhausted pending a
  new seed.

This section is an additive research record. It does not replace the existing
scoring or collection descriptions above.

### Additional observation-account bias

The collection currently uses the logged-in account `wangoroge333`. X may
personalize search results and visible ranking based on account state, region,
language, session history, eligibility, or temporary display restrictions.
The account can also affect which public relations are visible to the
collector. Therefore, results obtained through this account must not be
described as the distribution of all X users or all Japanese X posts. The
collection account, retrieval time, mode, query, and any restriction or error
must be recorded as observation conditions. A future robustness check should
compare independent, policy-compliant observation contexts where authorized;
until then this is a known limitation of the dataset.

### Local implementation

The reproducible sampler is available locally as:

```bash
uv run jigoku seed generate --per-topic 1 --random-seed 20260917
```

It searches one bounded date/query stratum per configured topic, removes native
reposts, samples low/middle/high exposure strata with a deterministic random
seed, and stores the selected tweet ID and selection metadata in the local
`seed_candidates` table. Search results are not a random sample of X; the
command is a documented, bounded approximation. The resulting IDs can then be
passed one at a time to the unchanged `protocol seed` command.
