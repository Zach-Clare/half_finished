This is what's known as a software paper, or a metapaper, so there are some slightly different rules. It's mainly to fill the gap between methodology chapters and papers that describe algorithms and procedures.

The usual length is 2 - 10 pages. The length itself doesn't matter too much, but will depend on the size of the program being discussed.

We want to cover the main following sections:
- [[Abstract]]
- [[Problem Area|Description of the problem it solves]]
- [[Predecessor Software|Similar or predecessor software]]
- [[Architecture|Architecture of the software]]
- [[Methods and Techniques|Methods and techniques implemented (links and references, along with any meaningful changes)]]
- Testing and validation
- Further links to documentation and installation guidance (appendix?)
- [[Performance]]
- [[Conclusion]]
- [[Use Cases]]

So overall, there isn't a lot to cover. I've set up preparation so that each subheading is an obsidian note with some discussions on key points. The actual paper will be written in LaTeX format on Overleaf

~~We should discuss tradeoffs of the uniform grid.~~ (Architecture: Input)
We should also talk about performance probably.
And also we should absolutely have sample renderings.
We could discuss the trilinear sampling method more?
We could also go more in depth on pixel ray generation.
~~Touch on how it is all through command line, so it can be used with popular machine learning tools in python easily.~~
~~Fix the Re display in the intro.~~
~~The introduction is a bit everywhere, we should talk less about the Level 4 pipeline specifically and more about viewing output of emissivity models, empirical and MHD. The level4 discussion is best placed in the use cases section. Actually, do we want to discuss it at all? Maybe we don't, saying that the renderer needs to be fast so it can be part of an ML process is enough.~~
I NEED TO CREDIT ANDY FOR THE SUN-LINE ROTATION CODE