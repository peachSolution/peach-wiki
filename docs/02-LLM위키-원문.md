# LLM Wiki 원문

> 원문 문서. `peach-wiki` 설계의 최상위 아이디어 출처다.
> 관련 해설: [패턴 분석](03-LLM위키-패턴-분석.md), [시스템 구성도](06-LLM위키-시스템-구성도.md), [종합 분석](05-qmd와-LLM위키-종합분석.md)
> 출처: 초기 분석 문서에서 이관

---

## 원본 주소

- Andrej Karpathy 원문 Gist: <https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f>

A pattern for building personal knowledge bases using LLMs.

This is an idea file, it is designed to be copy pasted to your own LLM Agent (e.g. OpenAI Codex, Claude Code, OpenCode / Pi, or etc.). Its goal is to communicate the high level idea, but your agent will build out the specifics in collaboration with you.

## The core idea

Most people's experience with LLMs and documents looks like RAG: you upload a collection of files, the LLM retrieves relevant chunks at query time, and generates an answer. This works, but the LLM is rediscovering knowledge from scratch on every question. There's no accumulation. Ask a subtle question that requires synthesizing five documents, and the LLM has to find and piece together the relevant fragments every time. Nothing is built up. NotebookLM, ChatGPT file uploads, and most RAG systems work this way.

The idea here is different. Instead of just retrieving from raw documents at query time, the LLM incrementally builds and maintains a persistent wiki — a structured, interlinked collection of markdown files that sits between you and the raw sources. When you add a new source, the LLM doesn't just index it for later retrieval. It reads it, extracts the key information, and integrates it into the existing wiki — updating entity pages, revising topic summaries, noting where new data contradicts old claims, strengthening or challenging the evolving synthesis. The knowledge is compiled once and then kept current, not re-derived on every query.

This is the key difference: the wiki is a persistent, compounding artifact. The cross-references are already there. The contradictions have already been flagged. The synthesis already reflects everything you've read. The wiki keeps getting richer with every source you add and every question you ask.

You never (or rarely) write the wiki yourself — the LLM writes and maintains all of it. You're in charge of sourcing, exploration, and asking the right questions. The LLM does all the grunt work — the summarizing, cross-referencing, filing, and bookkeeping that makes a knowledge base actually useful over time. In practice, I have the LLM agent open on one side and Obsidian open on the other. The LLM makes edits based on our conversation, and I browse the results in real time — following links, checking the graph view, reading the updated pages. Obsidian is the IDE; the LLM is the programmer; the wiki is the codebase.

This can apply to a lot of different contexts. A few examples:

- Personal: tracking your own goals, health, psychology, self-improvement — filing journal entries, articles, podcast notes, and building up a structured picture of yourself over time.
- Research: going deep on a topic over weeks or months — reading papers, articles, reports, and incrementally building a comprehensive wiki with an evolving thesis.
- Reading a book: filing each chapter as you go, building out pages for characters, themes, plot threads, and how they connect.
- Business/team: an internal wiki maintained by LLMs, fed by Slack threads, meeting transcripts, project documents, customer calls.
- Competitive analysis, due diligence, trip planning, course notes, hobby deep-dives.

## Architecture

There are three layers:

- Raw sources — your curated collection of source documents. These are immutable.
- The wiki — a directory of LLM-generated markdown files. The LLM owns this layer entirely.
- The schema — a document that tells the LLM how the wiki is structured, what the conventions are, and what workflows to follow.

## Operations

### Ingest

You drop a new source into the raw collection and tell the LLM to process it. An example flow: the LLM reads the source, discusses key takeaways with you, writes a summary page in the wiki, updates the index, updates relevant entity and concept pages across the wiki, and appends an entry to the log.

### Query

You ask questions against the wiki. The LLM searches for relevant pages, reads them, and synthesizes an answer with citations. Good answers can be filed back into the wiki as new pages.

### Lint

Periodically, ask the LLM to health-check the wiki. Look for contradictions between pages, stale claims, orphan pages, important concepts mentioned but lacking their own page, missing cross-references, and data gaps.

## Indexing and logging

Two special files help the LLM (and you) navigate the wiki as it grows.

- `index.md` is content-oriented. It is a catalog of everything in the wiki.
- `log.md` is chronological. It is an append-only record of what happened and when.

## Optional: CLI tools

At some point you may want to build small tools that help the LLM operate on the wiki more efficiently. A search engine over the wiki pages is the most obvious one. `qmd` is a good option: it's a local search engine for markdown files with hybrid BM25/vector search and LLM re-ranking, all on-device.

## Why this works

The tedious part of maintaining a knowledge base is not the reading or the thinking — it's the bookkeeping. Updating cross-references, keeping summaries current, noting when new data contradicts old claims, maintaining consistency across dozens of pages. Humans abandon wikis because the maintenance burden grows faster than the value. LLMs don't get bored, don't forget to update a cross-reference, and can touch many files in one pass. The wiki stays maintained because the cost of maintenance is near zero.

The human's job is to curate sources, direct the analysis, ask good questions, and think about what it all means. The LLM's job is everything else.
