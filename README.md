# KBO Fight-Song Lyrics and Hitters' Performance

[English](README.md) | [繁體中文](README.zh-TW.md)

Meng-Yu Chiu · KAIST HSS510 · Spring 2025

<b>Are KBO fight-song lyrics associated with hitters' performance?</b>

This individual research project was completed for KAIST HSS510, NLP for Humanities and Social Sciences. I linked <b>more than 54,000 player-game records</b> from 2022–2024 with manually compiled fight-song information, designed a lyric score for the baseball-cheering context, and examined its association with batting average.

The work required deciding how to turn lyrics into an interpretable variable, which units to analyze, and how far the statistical results could support the conclusions.

[Notion portfolio: 中文 / English](https://app.notion.com/p/258fea401f5a8004a222c5d5bb58a2c6)

## 1. Research Motivation and Questions

KBO players' fight songs repeatedly call out player names, team names, and baseball terms. I wondered: <b>if the lyrics themselves became measurable data, would they show an association with hitters' performance?</b>

In my literature review, fight songs were often discussed through fan culture and team identity, while research on audiences and performance focused on attendance or home advantage. I wanted to examine the lyrics, testing how a custom lyric score related to batting average and whether home games, crowd size, or song sharing changed that relationship. Team language and batting order provided supplementary questions.

The study uses observational data to identify associations worth investigating. Songs were not randomly assigned to players, so it cannot directly establish whether changing a song improves performance.

## 2. My Role and Responsibilities

This was an individual course project. I designed the research questions and hypotheses, collected and cleaned game data, manually compiled players' fight songs, coded reuse and inheritance, designed the lyric score, conducted the statistical analysis, created figures, and wrote the paper.

I needed to decide which cheering terms to score, how to classify song use, and when to analyze game-level records or aggregated data. Claude Code assisted with implementation; I was responsible for the research definitions, manual judgments, and interpretation.

## 3. Data and Units of Analysis

| Data | Source and contents | Organization and use |
| --- | --- | --- |
| Game context | KBO schedules, stadiums, attendance, and results for 2022–2024 | Align game-level information by gameId; exclude rain-cancelled games without hitter performance data |
| Hitter records | KBO game-data endpoints, accessed by gameId, with at-bats, hits, RBIs, runs, batting average, batting order, and defensive position | Each merged row records one player's appearance in one game; the same player appears repeatedly |
| Fight songs | Manually compiled personal fight-song lyrics, original-song information, reuse and inheritance flags, and shared-player counts | Join to appearances by player and team; team songs played for batters without personal songs were not comprehensively covered |

<b>The 54,000-plus rows are appearances, not independent players or songs.</b> The main score–batting-average analysis first pools the three seasons into player–team groups. A player appearing for different teams contributes separate groups, and batting average is calculated as total hits divided by total at-bats within each group. Other hypotheses use game-level or player-level data as appropriate.

Song use also requires clear definitions. <b>Reuse</b> means retaining a melody while substantially rewriting the lyrics; <b>inheritance</b> means largely retaining the lyrics and mainly replacing the player's name. Shared-player count records how many players use the same song.

## 4. Methods, Tools, and Outputs

| Step | Tools | Output and decisions |
| --- | --- | --- |
| Integrate data | Python, Selenium, BeautifulSoup, requests, pandas | Align schedules, attendance, and hitter records, then join manually compiled song information |
| Quantify lyrics | Kiwi, a custom cheering lexicon, and contextual chant rules | Produce a lyric score that retains baseball vocabulary and chant-pattern features |
| Test hypotheses | scipy, statsmodels | Correlations, regressions, interactions, and group comparisons, with the unit chosen for each question |
| Explore language | POS analysis, KLUE BERT, sentence-transformers, K-means, PCA, UMAP | Describe team language and lyric semantics for exploration, rather than ranking team performance |

I called the custom measure the <b>lyric sentiment score</b>. For example, team names receive 1 point, while hits, home runs, and selected action terms receive 2. Qualifying segments are doubled when they appear in parentheses or near specified punctuation or chant patterns; the final score averages the segments that receive a nonzero score.

The score describes cheering-language features under my lexicon and weighting rules. It does not directly measure players' emotions. Repeated vowels and chants provide contextual cues that should not automatically be discarded as noise.

<details>
<summary>Code and reproducibility</summary>

The public repository contains the research notebook, small data summaries, and selected figures. It does not include the complete game-level model inputs or manually compiled lyric data. Running the scraper or using the small CSVs in data alone will not reproduce the entire analysis.

Before use, adjust Chrome/chromedriver, file paths, and inputs. Song information must be compiled before generating intermediate files such as with\_sentiment\_boost\_rounded.csv and player\_team\_summary.csv. The notebook retains alternative versions and exploratory sections that need to be run according to their data dependencies.

There are also two different embedding paths. The model called KoBERT in the paper is loaded as klue/bert-base in the current code; the team-level version first truncates concatenated text to 512 characters. A separate sentence-transformers exploration operates at lyric level. The supplementary figure below uses the latter; points in the two approaches represent different units and should not be interpreted interchangeably.

```bash
python -m pip install -r requirements.txt
jupyter lab notebooks/KBO_statistics.ipynb
```

See [LICENSE](LICENSE) for the code license. Full lyrics are not included in the public data folder.

[Research notebook](notebooks/KBO_statistics.ipynb) · [Environment requirements](requirements.txt)

</details>

## 5. Key Findings

The numbers below are reported in the June 15, 2025 term paper, with the analysis procedures checked against the public code. Complete model inputs are not included in the repository.

### 1. The lyric score was positively associated with batting average, with limited explanatory power

In the player–team summary data, Pearson r = 0.249 and Spearman ρ = 0.305, both p < .001. In the simple regression, a one-standard-deviation higher score was associated with a 0.0248 higher batting average. However, <b>R² = 0.062: the model explained only 6.2% of the variation in batting average</b>.

After controlling for average attendance and the share of home appearances, the score's coefficient remained positive (β = 0.0244, p < .001). This supports a limited association, without establishing that changing a song improves performance.

![Batting-average distributions for high and low lyric-score groups](figures/batting_by_sentiment_group.png)

Groups are defined by the median player–team mean score. The figure shows distributions; differences between groups are not an estimate of a song intervention.

### 2. There was no evidence that home games or larger crowds amplified the association

| Context | Interaction p-value | Interpretation |
| --- | --- | --- |
| Home game × lyric score | p = .403 | The model using game-level appearances did not detect evidence that home games changed the score–batting-average relationship |
| Crowd group × lyric score | p = .998 | The model grouping players by average attendance did not detect a difference in relationship slopes between high- and low-attendance groups |

In the crowd-group model, the low-attendance coefficient was about −0.0221 (p = .021), but <b>a group difference does not establish that attendance amplifies the lyric association</b>. The interaction result in Table 5 does not support an amplification claim.

![Batting-average distributions by home/away context and lyric-score group](figures/batting_home_vs_away.png)

This is a descriptive comparison by context and score group. Moderation requires an interaction test; it cannot be established from the boxplot alone.

### 3. Reuse, inheritance, and sharing showed no clear performance differences

| Comparison | Reported result |
| --- | --- |
| Reused versus non-reused songs | t = −0.154, p = .8795 |
| Inherited versus non-inherited songs | t = −1.258, p = .2167 |
| Shared-player count and batting average | β = 0.0101, p = .356; R² = 0.002 |

Under this sample and analysis, there was insufficient evidence that song-sharing practices explained differences in batting performance. Non-significant findings do not establish that all song-use practices are equivalent; more data would be needed to investigate that.

<details>
<summary>Supplementary exploration: team language, semantics, and batting order</summary>

The paper also explores team language through common words, POS proportions, and semantics. For example, some teams repeat team and player names more often. These analyses describe lyrics, rather than ranking teams' playing strength.

![Lyric-level UMAP exploration colored by team](figures/team_lyric_style_umap.png)

Each point in this repository figure is a lyric deduplicated by team and lyric text. Color represents the team and point size the custom score; each point is not a team. This sentence-transformers exploration differs from the team-level embedding figures in the paper.

The batting-order relationship was weak (Spearman ρ = −0.036; OLS β = −0.087, R² = 0.003). A significant p-value alone does not make it a strong practical relationship or establish that teams deliberately arrange batting order around lyrical sentiment.

</details>

## 6. Challenges and Lessons

- <b>A context-specific score still needs validation.</b> Fight songs contain baseball terms and repeated chants, so I retained these features through a custom lexicon and contextual weighting. The weights were my choices and have not been validated against human sentiment ratings. The score should currently be treated as an interpretable custom measure, with alternative weights and rating methods still to be compared.
- <b>Appearances, players, and songs are different units.</b> The same song recurs across many appearances, so the main analysis aggregates to player–team level. The current code also fills missing lyrics with none, which receives a score of 0; 0 can also mean no lexicon matches. These cases should not all be treated as low-emotion lyrics. Sensitivity checks for missing-data exclusions and thresholds for total at-bats remain to be done.
- <b>Main effects and interactions answer different questions.</b> A significant low-attendance coefficient does not establish that attendance amplifies the lyric association. Group differences and slope differences need separate interpretations, and the written conclusion must follow the actual test.
- <b>An association does not establish that cheering causes improvement.</b> Player ability, playing opportunities, and song selection may jointly shape the results. The data do not fully record when a player changed songs, which version was actually played, or the shares of home and away supporters. Addressing causality would require better timing information and a different research design, alongside currently unmeasured factors such as melody, volume, or rhythm.
