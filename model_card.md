# 🎧 Model Card: Music Recommender Simulation

## 1. Model Name

**VibeScore 1.0**

It scores how well each song matches your vibe. Then it ranks the songs.

---

## 2. Goal / Task

VibeScore suggests songs from a small catalog.

You tell it three things:

- A favorite genre
- A favorite mood
- A target energy level from 0 to 1

It gives every song a score. Then it returns the top 5.

It does not predict what you will like in the future. It only matches songs to what you say you want right now.

---

## 3. Data Used

- The catalog has **30 songs** in `data/songs.csv`.
- It covers **16 genres** and **13 moods**.
- Each song has 10 columns: id, title, artist, genre, mood, energy, tempo, valence, danceability, and acousticness.
- The starter file had 10 songs. I added 20 more made-up songs.
- The numbers are invented. They do not come from a real music service.

**The scoring only uses three columns:** genre, mood, and energy. The other columns are loaded but never used.

**Limits of the data:**

- It is tiny.
- 13 of the 16 genres have only one or two songs.
- More songs have high energy than low energy. 14 songs are at 0.65 or above. 7 songs are at 0.35 or below.
- Energy and acousticness move in opposite directions. The correlation is −0.93.
- There are no lyrics, languages, artist fame, or listening history.

---

## 4. Algorithm Summary

Every song starts at zero. Then it earns points.

- **Genre match: +2.0 points.** The genre must match exactly.
- **Mood match: +1.0 point.** The mood must match exactly.
- **Energy closeness: 0 to +1.0 points.** The closer the song's energy is to your target, the more points it gets. A perfect match gives 1.0.

The best possible score is 4.0.

Genre counts most. Genre is the steadiest sign of taste. Mood changes with the moment, so it counts less. Energy helps sort songs that tie on the other two.

Then the program sorts the songs from highest score to lowest. If two songs tie, the one with the closer energy goes first. It returns the top 5.

Each song also comes with reasons, like "genre match (+2.0)". So you can see why it ranked where it did.

---

## 5. Observed Behavior / Biases

**The main weakness: it trusts genre too much.**

A genre match is worth 2.0 points. That is as much as mood and perfect energy put together. So a song in the right genre often beats a song that fits everything else. I tested a "classical, angry, high-energy" listener. The top pick was a quiet, sad piano piece. It scored 2.25 just for being classical. An angry, high-energy metal song came second at 1.98. This also makes a filter bubble. For pop/happy and lofi/chill, four of the top five songs came from the listener's own genre. Nothing rewards variety.

**Why Gym Hero keeps showing up for happy pop fans.**

Think of the program as a judge giving points. Gym Hero is pop. That earns 2 points. Its energy is close to what a happy pop fan wants. That earns almost 1 more point. It is a workout song, not a happy song. But mood is only worth 1 point. So missing it costs little. Rooftop Lights is happy and has the right energy. But it is labeled "indie pop." The program only counts an exact "pop." So it gets no genre points. Gym Hero scores 2.87. Rooftop Lights scores 1.96.

**Other things I noticed:**

- **No credit for close genres or moods.** "Indie pop" is not "pop." "Chill" is not "relaxed."
- **The small catalog makes sharp drops.** A metal fan gets one perfect match (4.00). The next song scores 0.98.
- **Missing genres fail quietly.** A bossa nova fan gets a top score of only 1.98. Nothing tells them their genre was not found.
- **The `likes_acoustic` setting is never used.** Acoustic fans cannot say what they want.
- **Energy gap is symmetric.** A song 0.1 too loud scores the same as one 0.1 too quiet.

---

## 6. Evaluation Process

**What I tested.** I ran five profiles. For each one I looked at the top 5 songs. I checked three things:

- Does the top song make sense?
- Do the printed reasons explain the score?
- Do the lists change in sensible ways when the profile changes?

I also checked some scores by hand. They matched the program.

| Profile | Genre / mood / energy | Why I tested it |
|---|---|---|
| Default Pop/Happy | pop / happy / 0.8 | The starter example |
| Chill Lofi | lofi / chill / 0.4 | A quiet listener |
| Intense Rock | rock / intense / 0.92 | A loud listener |
| Conflicting Tastes | classical / angry / 0.95 | Genre and mood point different ways |
| Genre Not In Catalog | bossa nova / romantic / 0.5 | No song matches the genre |

The top songs were Sunrise City, Midnight Coding, Storm Runner, Moonlit Sonata Remix, and Velvet Hours. No song won every list.

**What surprised me.**

- Gym Hero showed up in three of the five lists.
- The quiet classical song beat the angry metal song.
- Late Night Bars beat Spacewalk Thoughts by only 0.02 points. A tiny energy gap decided it.

**Comparing each pair of profiles.**

1. **Pop/Happy vs Chill Lofi:** No songs are shared. Pop gets upbeat songs near 0.8 energy. Lofi gets quiet songs near 0.4. Genre points pick the group. Energy points sort inside it. This makes sense.
2. **Pop/Happy vs Intense Rock:** Only Gym Hero is shared. It is pop in one list. It is "intense" with high energy in the other. Rock swaps in Storm Runner and Garage Sunday. This makes sense.
3. **Pop/Happy vs Conflicting Tastes:** Only Gym Hero is shared. The pop list agrees with itself. The conflicting list has no song that fits all three wishes. Its top pick is a quiet classical piece. That does not feel right.
4. **Pop/Happy vs Genre Not In Catalog:** Only Neon Heartbeat is shared. Bossa nova gets no genre points. Its top score is 1.98 instead of 3.98. Mood and energy do all the work. It is a fair fallback, but the user is not told.
5. **Chill Lofi vs Intense Rock:** No songs are shared. Lofi sits near 0.33 to 0.42 energy. Rock sits near 0.85 to 0.97. Lofi stays lofi for four songs. Rock runs out after two. The catalog has more lofi than rock.
6. **Chill Lofi vs Conflicting Tastes:** No songs are shared. The lofi list is calm and makes sense. The conflicting list mixes classical, metal, EDM, pop, and rock. No song fits an angry, loud classical fan.
7. **Chill Lofi vs Genre Not In Catalog:** Late Night Bars is shared. It is chill with middle energy (0.50). Both listeners want calm-ish songs. That fits.
8. **Intense Rock vs Conflicting Tastes:** Three songs are shared: Storm Runner, Gym Hero, and Pulse Machine. There is only one classical song, so genre stops helping. Energy and mood take over. An angry classical fan gets treated like a rock fan.
9. **Intense Rock vs Genre Not In Catalog:** No songs are shared. Rock's top score is 3.99. Bossa nova's is 1.98. The system treats both lists as equally sure.
10. **Conflicting Tastes vs Genre Not In Catalog:** No songs are shared. The energy targets are far apart (0.95 vs 0.5). Most songs score near 1.0 or less. With no genre to help, the system mostly sorts by energy. That is a weakness.

---

## 7. Intended Use and Non-Intended Use

**Intended use:**

- Classroom practice.
- Learning how scoring and ranking work.
- Showing how a simple recommender turns data into a list.
- Small demos where every score can be explained.

**Not intended for:**

- A real music app with real users. The catalog is too small and the data is made up.
- Helping people discover new music. It keeps people inside their own genre.
- Deciding which artists get exposure or money.
- Judging someone's taste.
- Users with unusual or mixed tastes. It cannot handle two genres at once.
- Treating the score as the truth. It only counts three things.

---

## 8. Ideas for Improvement

1. **Give partial credit for close matches.** Give "indie pop" some points for a "pop" fan. Do the same for "chill" and "relaxed." This would fix the Gym Hero and Rooftop Lights problem.
2. **Use more of the data.** Use the `likes_acoustic` setting and the acousticness column. Add valence and tempo too. This would help listeners whose taste is not about energy.
3. **Add variety and honesty.** Make sure the top 5 includes at least one song from outside the user's genre. Also warn the user when their genre is not in the catalog.