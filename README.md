# Default Instance Test

On macOS Catalina (10.15.5 and earlier), there seems to be an issue in which the default instance of a variable font cannot be selected in apps such as TextEdit & Keynote that use CoreText and system APIs to handle fonts. This repo is a way to investigate & provide a simple test case for this issue.

Questions to answer are:
- Is this issue reproducible with an extremely simple font?
- Does the issue still occur if the default instance is “Regular”? E.g. is this only happening if the default instance is something like “Display Bold”?

## The Problem

### Summary

Variable fonts are composed of two or more *sources* (AKA “masters”) which allow intermediate styles to be derived via on-the-fly interpolation of coordinate points. They also usually have several to many *named instances* which are instances predefined by the font designer as useful locations within the designspace (available stylistic possibilities) of the font.

A simple example would be a font with sources at Light and ExtraBold weights, yielding a weight range of 300–800, and instances at Light, Regular, Medium, SemiBold, Bold, and ExtraBold.

A non-variable, “static” font records letter outlines as a series of mathematical curves with control points on a coordinate grid, in what is called the `glyf` table. A variable font also records points in the `glyf` table for the *default instance* – which is the style which will render even in older software that can’t display variable fonts. And then, a variable font also has tables recording the *deltas* of points, or the direction and amount they move by to render other sources. Intermediate instances are derived based on the default instance plus a factor of the deltas.

So, in the case of the two-source, Light to ExtraBold variable font, the default instance *must* be either Light or ExtraBold. Some (but not all) fonts will add an extra source at Regular, to have better control over the design, and so that the Regular style can be rendered in legacy software. However, this is not and should not be a requirement of a variable font, as each additional source adds filesize, and a primary benefit of the variable font format is reduction in filesize for the web.

Unfortunately, it seems that in system font handling macOS, only variable fonts with a default instance of “Regular” allow their default instance to be selected. If a default instance is anything else (e.g. Light), it cannot be selected from the font style menu.

### Examples

**Example:** [Recursive](https://github.com/arrowtype/recursive) has a default instance of `Sans Linear Light`. However, attempting to select this style instead results in selection of `Sans Casual Medium`.

![TextEdit failing to work for Recursive](readme-assets/textedit-recursive.gif)

**Example:** [Commissioner](https://github.com/kosbarts/Commissioner/blob/df81f836a18b8dca0e553e8fc55fe19de4935839/fonts/variable/Commissioner%5BFLAR%2CVOLM%2Cslnt%2Cwght%5D.ttf) has a default instance of `Thin`. Attempting to select this results in a selection of `Flair Medium`.

![TextEdit failing to work for Commissioner](readme-assets/textedit-commisioner.gif)

**Example:** [Name Sans](https://www.futurefonts.xyz/arrowtype/name-sans) (not open source, but does have a free trial font which can be tested) has a default instance of `Display Bold`. Attempting to select this style results in `Display Medium`.

![TextEdit failing to work for Name Sans](readme-assets/textedit-namesans.gif)

**Example of working font w/ Regular instance:** [Inter](https://github.com/rsms/inter) does work, likely due to the default instance being “Regular.”

![TextEdit working for Inter](readme-assets/textedit-inter.gif)