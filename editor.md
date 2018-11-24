# A Modern Rich Editor

Rich text is text which has formatting (bold/italics), has embedded
content such as images, tables and equations.  This document lists
some issues that such an editor should deal with.

## Unicode

1. [Unicode BIDI Algorithm](http://unicode.org/reports/tr9/).
Unicode text has a lot of directionality options. A good text
editor would support all of them but the real issue is when directions
are combined. Standard BIDI algorithms do not help with Top-To-Bottom
layout options though.

2. [Unicode Text Segmentation](http://unicode.org/reports/tr29/)
defines the standard for grouping code points but that does not handle
all the complexity of ligatures, say in Indic languages (or Arabic)
where the position of vowels needs algorithmic rendering.

3. [Harfbuzz](https://harfbuzz.github.io/) does shaping to deal with
standard font files. Typesetting math equations don't actually fit
into this though it is a similar problem.

## Shaping

The basic act of shaping is mapping a sequence of codepoints (or
bytes) into a compossition visual rendering of glyphs where the
individual glyphs are based on the code points somewhat loosely.

1. The mapping of code point to glpyh is contextual (though only based
on adjacent codepoints)

2. The actual position of the glyph is also contextual. For example,
vowels in hindi can be moved to the left as much as to the right.

3. The logical "grapheme" boundary depends on the use. Character
navigation (such as with cursors) may take one set of boundariess but
a "backspace" operation may choose anotheer.  For example, an emoji
with a skin-color variant may be treated as an atomic entity for
cursor navigation but backspace can simply remove the skin-variant
(allowing choosing a different variant).   Similarly, integral signs
with limits on them can be treated as simple single "graphemes" though
backspace may take the user into the individual limit edit experience.

## Overlays

A rich text editor often has typographic selections. An example is the
"underline" for clickable links or a tooltip indicator. While these
can be edited, they are effectively dependent properties.  Another
example is the red squiggly line indicating grammatical or spelling
errors.

1. Collaborative cursors, carets
2. Hyperlink indicators
3. Tooltips
4. Comment highlights
5. Syntax errors
6. User highlights

## Navigation

1. All individual parts of the text should be navigable by a sequence
of logical "Next" actions starting from the "start" of the
document.

2. The concrete key/action to which "Next" maps (such as Right Arrow
or enter) can vary based on the "context"




