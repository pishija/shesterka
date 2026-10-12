# Шестерка

A Macedonian 6-letter Wordle clone, inspired by [Zborle](https://zborle.web.app/).

Play at: **https://shesterka.com/**

---

## How to play

Guess the 6-letter Macedonian word in 6 tries.

- 🟩 Right letter, right position
- 🟨 Right letter, wrong position
- ⬛ Letter not in the word

A new word every day. Results can be shared as an emoji grid.

---

## Project structure

```
shesterka/
├── index.html          # entire game (HTML + CSS + JS — no build step)
├── words.js            # word list (ANSWERS + VALID_GUESSES arrays)
├── extract-words.py    # one-time script to regenerate words.js from the dictionary
└── README.md
```

No framework, no dependencies, no build step. `index.html` is served directly.

---

## Deploying to GitHub Pages

1. Push this repo to GitHub.
2. Go to **Settings → Pages**.
3. Set source to `main` branch, root (`/`).
4. The game is live at `https://<username>.github.io/<repo-name>/`.

Optional: add a `CNAME` file with a custom domain (e.g. `shesterka.mk`).

---

## Regenerating the word list

The word list is generated from the [gerazov/dictionary-mk](https://github.com/gerazov/dictionary-mk) Hunspell Macedonian dictionary.

```bash
# 1. Download the dictionary
curl -o mk_MK.dic https://raw.githubusercontent.com/gerazov/dictionary-mk/master/dict-mk/mk_MK.dic

# 2. Run the extraction script (requires Python 3)
python3 extract-words.py

# 3. Curate ANSWERS in words.js
#    Move common, familiar everyday words into the ANSWERS array.
#    Aim for ~500–1000 words. VALID_GUESSES stays as-is.
```

The extraction script outputs all valid 6-letter Macedonian Cyrillic words. The `ANSWERS` array needs manual curation — remove proper nouns and obscure words, keep familiar everyday words.

---

## Word list attribution

The word list is derived from the **OpenOffice Macedonian spellcheck dictionary**, originally developed by the Free/Libre Software Organisation of Macedonia (OSSM) and made available via [`gerazov/dictionary-mk`](https://github.com/gerazov/dictionary-mk).
