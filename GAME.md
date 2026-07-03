# 🦁🐊 Ezekiel's Animal & Reptile Sound Safari

A party game for Ezekiel's 6th birthday. Kids split into teams and take turns guessing
animals by their **sound** or **picture**, with silly **dance / freeze / funny-face breaks**
in between. Runs about **15–20 minutes**.

## ▶️ How to run it

It's a single self-contained web page — **no internet needed** at the party (all sounds and
pictures are bundled in `assets/`).

- **Easiest:** double-click `index.html` to open it in Chrome/Safari/Edge, then press the
  full-screen key (**F11** on Windows, **⌃⌘F** on Mac). Plug the laptop into the TV via HDMI.
- **Or deploy it:** drag this whole folder onto [netlify.com/drop](https://app.netlify.com/drop),
  or point your existing Netlify site at this repo (it serves `index.html` at the root).

> Tip: click/tap once on the page so the browser allows sound to play, then you're set.

## 🌴 Party background screen (leave it on the TV)

`background.html` is a full-screen animated safari scene ("EZEKIEL IS 6!") that plays soft
**ambient animal sounds** on a loop — perfect to leave running before and after the game for
atmosphere. Open `background.html`, tap once to start the sounds, and hit **▶ Start the Game**
when you're ready. The game's own start and end screens share the same scene and ambience
(with a 🔊/🔇 toggle top-corner), so the vibe is continuous. Sounds automatically go quiet
during the guessing rounds so the animal clips are clear.

## 🎮 How to play (for the grown-up host)

1. Set the number of teams (up to 6) and turns per team, rename teams if you like, and
   (optional) paste the kids' names in the box — press **Make Teams** and the game randomly
   splits everyone into the teams and shows you each roster. **Shuffle again** until it looks
   fair, then **Start**. (The party guest list is already pre-filled — just edit if someone's away.)
2. Teams take turns. The team whose turn it is gets first guess.
3. Press **Play Sound** (sound round) or just show the picture (picture round). Let the kids shout!
4. Press **Reveal Answer**.
5. **Tap the team that got it right** to give +1 (any team can steal!), or tap **No one**.
6. Every 3 turns there's a **silly break** — dance party, freeze dance, funny faces, charades…
7. At the end, a winner is crowned with confetti. 🏆

**Keyboard shortcuts:** `Space` = play / reveal / continue · `R` = replay sound ·
`1`–`6` = give a team the point · `0` = no one.

## 🐾 What's included

13 animals (incl. 2 reptiles — snake & crocodile, for our reptile keeper!): dog, cat, cow,
duck, pig, lion, tiger, elephant, monkey, zebra, hyena, snake, crocodile — each with a real
recorded sound and a picture, plus a fun fact.

## ✏️ Easy things to customise (in `index.html`)

- **Team names / mascots:** the `TEAM_PRESETS` list, or just rename them on the setup screen.
- **Animals & fun facts:** the `ANIMALS` list.
- **Break activities:** the `BREAKS` list.

## 🎵 Asset credits

Bundled sounds & images are from these public GitHub projects:

- Animal photos & several sounds — [CoderAvi/Animal-Sound-Game](https://github.com/CoderAvi/Animal-Sound-Game)
- Cow / duck / monkey sounds & cartoons — [Piotr-Mularski/animals-soundboard](https://github.com/Piotr-Mularski/animals-soundboard)
- Crocodile sound — [sahliaziz/clavier-interactif-projet](https://github.com/sahliaziz/clavier-interactif-projet)
- Crocodile picture — [guladam/guess_what](https://github.com/guladam/guess_what)

The party music and sound-effect "dings" are generated live in the browser (Web Audio) — no files.

🎂 Happy 6th Birthday, Ezekiel!
