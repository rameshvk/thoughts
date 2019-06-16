# Engineering or What I now know but wish I knew then

Engineering is as much about people as about technology.   Discussions of technical tradeoffs that do not factor in how people work are incomplete and possibly dangerous

## Communication is transfer of data, emotions, meaning and purpose

Good communation is not simply being accurate or honest -- it is about saying things to maximize transfer of knowledge (data), emotions (how one feels about it), meaning (how important something is) and purpose (what role does that play in the overall scheme).

Because everyone interprets the same words differently, much of communication involves establishing a rapport and often involves changing the way one says things based on the receiving party.   Having empathy helps with communication because an empath can figure out a phrasing that will be understood the way intended.  Cross-cultural differences require both sides to make an effort to be more explicit about meaning and emotions as the social cues may not translate well.

## No total order of solutions

The idea of "optimal" solutions is deeply entrenched but all solutions have a profile on mulitple axes and total order rarely exists: one solution is better on some and worse on other axes.   Often, individual measures are quite subjective too (such as maintenance cost).

## Optimal solutions are often non-linear with requirements.

When requirements change a little, the optimal solution (disregarding sunk cost) often involves drastic changes to the prior optimal solution.  Engineering is the art of finding solutions that are nearly optimal but which allow linear incremental changes with requirements.

## No placeholders for the future

The future is always murky and elaborate placeholders for future items (based on "modularity", "extensibility" or "configurability") often end up not being used and eventually becoming a drag on all code change.

## Composition as a defence against change

A highly composable solution would be easy to take apart and put-back together in a different configuration (or with a different component).  This is often a good defence against change as every configuration remains close to optimal at that current point in time.
