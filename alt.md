# Experiment: Can the Taste Profile Tell "Intense Rock" from "Chill Lofi"?

## The question

Prompt used:

> "My taste profile is `{favorite_genre: pop, favorite_mood: happy, target_energy: 0.75, target_valence: 0.65, likes_acoustic: False}`. My scoring gives mood +2, genre +1, energy closeness up to 2, valence closeness up to 1 and acoustic fit up to 0.5. Using my `songs.csv`, will these preferences let the system clearly tell 'intense rock' (Storm Runner) apart from 'chill lofi' (Library Rain, Midnight Coding)? Or is the profile too narrow? Show the actual scores, explain which features create or shrink the gap, and suggest changes if the profile can't tell them apart."

## Finding: the features work, but the profile is narrow

When the profile asks for rock or lofi directly, the recommender separates them clearly:

| Profile | Storm Runner (rock, intense) | Library Rain (lofi, chill) |
|---|---|---|
| rock / intense / energy 0.9 | **6.41** (1st) | 1.87 (8th) |
| lofi / chill / energy 0.35 | 1.81 (9th) | **6.43** (1st) |

With the pop `TASTE_PROFILE`, they end up close together: Storm Runner scores **2.96** (5th) and Library Rain **2.22** (9th), a gap of only **0.74**.

Why the gap is small:

- Neither song is pop or happy, so neither gets genre or mood points. Those are the largest parts of the score.
- The energy target (0.75) sits between rock (0.91) and lofi (0.35), so energy separates them by only 0.48 points.
- The valence target (0.65) is closer to lofi (0.60) than to rock (0.48), so valence works slightly against the separation.

The profile only describes what the user likes. Every song that isn't pop and happy gets squeezed into a narrow band of scores.

## Experiment: make energy matter more

I tried two changes to the energy scoring and compared them with the original. Everything else stayed the same.

| Version | Change | Storm Runner | Library Rain | Rock vs. lofi gap |
|---|---|---|---|---|
| **A. Baseline** | Energy weight 2.0, linear closeness `1 − \|d\|` | 2.96 | 2.22 | 0.74 |
| **B. Heavier energy** | Energy weight 4.0 | 4.64 | 3.42 | 1.22 |
| **C. Steeper curve** | Gaussian closeness `exp(−d² / (2 · 0.2²))` | 2.60 | 1.31 | **1.29** |

*Version C changes the shared `closeness` function, so valence is also scored with the steeper curve.*

### Full rankings for `TASTE_PROFILE`

| Rank | A. Baseline | B. Energy ×2 | C. Gaussian |
|---|---|---|---|
| 1 | Sunrise City 6.08 | Sunrise City 7.94 | Sunrise City 5.93 |
| 2 | Rooftop Lights 5.15 | Rooftop Lights 7.12 | Rooftop Lights 5.05 |
| 3 | Gym Hero 3.99 | Gym Hero 5.63 | Gym Hero 3.64 |
| 4 | Night Drive Loop 3.23 | Night Drive Loop 5.23 | Night Drive Loop 3.12 |
| 5 | Storm Runner 2.96 | Storm Runner 4.64 | Storm Runner 2.60 |
| 6 | Midnight Coding 2.40 | Midnight Coding 3.73 | Midnight Coding 1.56 |
| 7 | Focus Flow 2.35 | Focus Flow 3.65 | Focus Flow 1.50 |
| 8 | Coffee Shop Stories 2.24 | Coffee Shop Stories 3.48 | Coffee Shop Stories 1.34 |
| 9 | Library Rain 2.22 | Library Rain 3.42 | Library Rain 1.31 |
| 10 | Spacewalk Thoughts 2.10 | Spacewalk Thoughts 3.16 | Spacewalk Thoughts 1.17 |

## What I learned

- **The order didn't change in any version.** The baseline already ranks the songs in a sensible order. The changes only affect how *confident* the separation is.
- **Version C works best.** It nearly doubles the rock vs. lofi gap. It does this by pushing far-away songs (all the calm songs) much lower, while barely touching songs near the target. Version B also widens the gap, but it inflates every score, so the gap grows less in proportion.
- **Weights can't fix a narrow profile.** The calm songs (lofi, jazz, ambient) are still bunched together at the bottom in every version. Telling them apart would need richer preferences, such as several favorite genres, disliked genres, or partial credit for related moods like chill / relaxed / focused.

## Decision

The main recommender keeps the **baseline** (Version A). It's simpler to explain, and it already produces the same ranking. Version C would be the choice if later features need a stronger penalty for songs whose energy is far from the target.

## How to reproduce

From the project root:

```python
import math
import src.recommender as R
from src.main import TASTE_PROFILE

songs = R.load_songs("data/songs.csv")
prefs = R._profile_to_prefs(R.UserProfile(**TASTE_PROFILE))

R.WEIGHTS["energy"] = 4.0                                     # Version B
# R.closeness = lambda v, t: math.exp(-(v - t)**2 / (2 * 0.2**2))  # Version C

for song, score, _ in R.recommend_songs(prefs, songs, k=10):
    print(f"{score:.2f}  {song['title']}")
```
