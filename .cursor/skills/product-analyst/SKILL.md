---
name: product-analyst
description: States the one file the product analyst may write and repeats the source and conflict rules for that pass. Read this before writing docs/product/spec.md.
disable-model-invocation: true
---

# Product analyst

The only file this agent may write is docs/product/spec.md. Replace its body with the English product description. Cover the functional spec, UML stored as Markdown text, and identity management. Link to architecture.md, error-handling.md, and concurrency.md. Do not create the other three files.

Write only when the prompt says formalizer started the pass and includes one input path ending in .xml or .txt.

## Sources

Before writing, read this skill, .cursor/rules/product-analyst.mdc, then that input, docs/Unimap_2.0_презентація.pdf, docs/Дипломна_робота_Зозуля_Олег.pdf, docs/Презентація_дипломна_робота_Зозуля_Олег.pdf, docs/product/spec.md, docs/product/roadmap.md, docs/github.txt, and the public repository named in docs/github.txt.

Do not edit the PDFs, roadmap.md, or github.txt, and do not paste the thesis or the slides verbatim.

## Conflicts

A conflict is a disagreement about the final product among docs/Unimap_2.0_презентація.pdf, docs/Дипломна_робота_Зозуля_Олег.pdf, docs/Презентація_дипломна_робота_Зозуля_Олег.pdf, and the current spec draft, or a disagreement among the four English files. A difference between the public repository and the Unimap 2.0 presentation is not a conflict.
