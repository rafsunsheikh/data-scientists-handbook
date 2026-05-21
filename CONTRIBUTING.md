# Contributing to the Data Scientist's Handbook

Thank you for your interest in improving this handbook! It is designed to grow over time as the community contributes domain expertise, examples, and corrections.

## Ways to contribute

- **Fill in stubbed sections.** Industry chapters and data-source pages are stubs — pick one you know and write it up.
- **Add a notebook.** Concrete, runnable Python examples are the highest-leverage contribution. Drop them in `notebooks/` and link them from the relevant chapter.
- **Improve existing content.** Fix factual errors, add citations, clarify explanations.
- **Add visualizations.** Each viz chapter benefits from a worked example with code and an image.
- **Translate.** Translations of any chapter are welcome.

## Style guidelines

- **Be concrete.** Prefer "use `pandas.DataFrame.isna().sum()` to count nulls per column" over "check for missing values."
- **Cite primary sources.** When citing a method or industry fact, link the original paper, textbook, regulation, or vendor documentation.
- **Code that runs.** Every code block should be executable as-is. Notebooks must run top-to-bottom on a fresh kernel.
- **Use Python 3.10+** and the libraries listed in `requirements.txt`.
- **Avoid AI-generated boilerplate.** Write from experience; if you used an LLM to draft, edit it down to what you actually know is correct.

## Adding a new chapter

1. Create the markdown file in the appropriate top-level folder.
2. Add a link to it from that folder's `README.md` index.
3. Add a link from the top-level `README.md` if it introduces a new top-level topic.
4. If you add a notebook, place it in `notebooks/<topic>/` and link it from the chapter.

## Pull request checklist

- [ ] Chapter has a clear title and one-paragraph TL;DR at the top.
- [ ] All claims have a citation (link or footnote).
- [ ] Code blocks are tested.
- [ ] New files are linked from the parent index.
- [ ] No large data files committed (>1 MB). Link to the source instead.

## License

By contributing, you agree your contribution is licensed under the MIT License (see `LICENSE`).
