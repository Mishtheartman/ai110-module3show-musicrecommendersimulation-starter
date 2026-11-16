# 🎵 Music Recommender Simulation

## Project Summary

In this project you will build and explain a small music recommender system.

Your goal is to:

- Represent songs and a user "taste profile" as data
- Design a scoring rule that turns that data into recommendations
- Evaluate what your system gets right and wrong
- Reflect on how this mirrors real world AI recommenders

Replace this paragraph with your own summary of what your version does.

---

## How The System Works

### Data

Each `Song` has: `id`, `title`, `artist`, `genre`, `mood`, `energy`, `tempo_bpm`, `valence`, `danceability`, and `acousticness`. The catalog in `data/songs.csv` has 30 songs across 16 genres and 13 moods.

A user taste profile stores a favorite `genre`, a favorite `mood`, and a `target_energy` between 0 and 1.

### Algorithm recipe

Every song in the catalog is scored against the user's profile. The score has three parts, for a maximum of 4.0:

| Rule | Points |
|---|---|
| Genre exactly matches | +2.0 |
| Mood exactly matches | +1.0 |
| Energy closeness: `1.0 - abs(song.energy - target_energy)` | 0.0 to 1.0 |

`score_song` returns `(score, reasons)`, where each reason is a short string such as `"genre match (+2.0)"` or `"energy 0.82 vs your target 0.80 (+0.98)"`.

### Ranking

1. Score every song.
2. Round each score to 2 decimals, so floating-point noise cannot reorder songs that are effectively tied.
3. Sort by score (highest first), then by smaller energy gap.
4. Return the top `k` songs as `(song, score, explanation)`, where the explanation joins the reasons with `"; "`.

### Why these weights

- **Genre (2.0) counts most.** It is the most stable signal of taste. A lofi fan rarely wants metal, whatever their mood.
- **Mood (1.0) counts half as much.** Mood changes with the situation, so it adjusts the ranking without overriding genre.
- **Energy (up to 1.0) is continuous.** It separates songs that the two yes/no rules score the same.
- **The ranges overlap on purpose.** A perfect energy match (+1.0) is worth as much as a mood match, so a song with the right energy can beat one with the right mood but the wrong energy.

### Predicted behavior (hand calculation)

| `pop / happy / 0.8` | Score | `lofi / chill / 0.4` | Score |
|---|---|---|---|
| Sunrise City | 3.98 | Midnight Coding | 3.98 |
| Golden Hour Drive | 3.98 | Library Rain | 3.95 |
| Gym Hero | 2.87 | Focus Flow | 3.00 |
| Neon Heartbeat | 2.85 | Rainy Window Tape | 2.93 |
| Rooftop Lights | 1.96 | Late Night Bars | 1.90 |

### Expected biases and limitations

- **It will over-prioritize genre.** A genre match alone is worth 2.0, which is the most that mood and energy can add together. A song in the right genre with the wrong mood and wrong energy can therefore outrank a song that matches mood and energy perfectly. Rooftop Lights (indie pop, happy, energy 0.76) scores 1.96 for a `pop / happy / 0.8` user, below Gym Hero (pop, intense) at 2.87.
- **Near-genres get no credit.** Matching is exact, so "indie pop" earns nothing for a "pop" fan, and the same goes for "indie rock" and "rock". Moods work the same way: "chill", "relaxed" and "focused" are close in feel but score as unrelated.
- **Small catalogs favor some tastes.** Most genres have only one or two songs, so a user who likes country or metal gets one strong match, then weak ones. Pop and lofi fans get fuller lists.
- **It ignores most of the data.** Tempo, valence, danceability and acousticness are in the CSV but never scored, so two songs that feel very different can score the same.
- **Energy treats "too high" and "too low" alike.** A song 0.1 above the target scores the same as one 0.1 below it, even though those can feel different to a listener.
- **Ties happen.** Sunrise City and Golden Hour Drive both score 3.98 for the pop/happy profile and are both 0.02 from the target energy. After rounding, the stable sort keeps file order, so Sunrise City comes first. That ordering is arbitrary.

I'll test these biases in the Experiments section by changing the weights and comparing how the rankings shift.

---

## Getting Started

### Setup

1. Create a virtual environment (optional but recommended):

   ```bash
   python -m venv .venv
   source .venv/bin/activate      # Mac or Linux
   .venv\Scripts\activate         # Windows
   ```

2. Install dependencies

```bash
pip install -r requirements.txt
```

3. Run the app:

```bash
python -m src.main
```

### Running Tests

Run the starter tests with:

```bash
pytest
```

You can add more tests in `tests/test_recommender.py`.

---

## Sample Recommendation Output

Output of `python -m src.main` for the default `pop / happy / 0.8` profile:

```
Loaded songs: 30

============================================================
Profile: genre=pop, mood=happy, energy=0.8
Top 5 recommendations
============================================================

1. Sunrise City by Neon Echo
   Genre: pop | Mood: happy
   Score: 3.98 / 4.00
   Why:
     - genre match (+2.0)
     - mood match (+1.0)
     - energy 0.82 vs your target 0.80 (+0.98)

2. Golden Hour Drive by Sunny Circuit
   Genre: pop | Mood: happy
   Score: 3.98 / 4.00
   Why:
     - genre match (+2.0)
     - mood match (+1.0)
     - energy 0.78 vs your target 0.80 (+0.98)

3. Gym Hero by Max Pulse
   Genre: pop | Mood: intense
   Score: 2.87 / 4.00
   Why:
     - genre match (+2.0)
     - energy 0.93 vs your target 0.80 (+0.87)

4. Neon Heartbeat by Velvet Static
   Genre: pop | Mood: romantic
   Score: 2.85 / 4.00
   Why:
     - genre match (+2.0)
     - energy 0.65 vs your target 0.80 (+0.85)

5. Rooftop Lights by Indigo Parade
   Genre: indie pop | Mood: happy
   Score: 1.96 / 4.00
   Why:
     - mood match (+1.0)
     - energy 0.76 vs your target 0.80 (+0.96)
```

The results match the hand calculation above, including the two quirks: the Sunrise City / Golden Hour Drive tie, and Rooftop Lights (indie pop, happy) ranking below two pop songs with the wrong mood.

**Screenshot or video** *(optional)*: <!-- Insert a screenshot or demo video link here -->

---

## Experiments You Tried

Use this section to document the experiments you ran. For example:

- What happened when you changed the weight on genre from 2.0 to 0.5
- What happened when you added tempo or valence to the score
- How did your system behave for different types of users

---

## Limitations and Risks

Summarize some limitations of your recommender.

Examples:

- It only works on a tiny catalog
- It does not understand lyrics or language
- It might over favor one genre or mood

You will go deeper on this in your model card.

---

## Reflection

Read and complete `model_card.md`:

[**Model Card**](model_card.md)

Write 1 to 2 paragraphs here about what you learned:

- about how recommenders turn data into predictions
- about where bias or unfairness could show up in systems like this