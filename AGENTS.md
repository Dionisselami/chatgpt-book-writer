# Book writing with Proseify

Proseify is connected as an MCP server named `proseify`. It is a corpus plus a genre-discipline
layer — **you** write the prose, it supplies the literature, the genre recipes, and the scoring
gate. It calls no model of its own.

## Order of operations (do not reorder)

1. `list_genres`, then `get_genre_recipe` for the closest fit. State the recipe and the reason.
2. `search_corpus` for 3–5 model passages in that genre. Quote one line that sets the register.
3. `plan_book` once — premise, chapter count, word budget. Print the outline and hold it.
4. Draft one chapter per turn. After each, run the edit pass below and say what you cut.
5. `evaluate_book` before declaring the book done. Fix flagged chapters and re-run.

## Per-chapter edit pass

- `seemed to`, `began to`, `started to`, `could feel` — delete or recast
- filter words: *felt, noticed, watched, saw, heard*
- stacked adverbs on dialogue tags; plain `said` is fine
- repeated sentence openings; weather openings
- three-item lists as rhythm filler
- dialogue that exists to explain plot
- the last line turning into a metaphor for the chapter

## Rules

- One chapter at a time. A 50,000-word book is not one turn, and pretending otherwise produces a
  synopsis with chapter headings.
- Write each finished chapter to `chapters/NN.md`. Your context is not durable; the files are.
- Never print or commit the Proseify key. It is read from the `PROSEIFY_API_KEY` environment
  variable.
