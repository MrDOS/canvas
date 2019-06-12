# Canvas <a name="canvas"></a> [![GoDoc](http://godoc.org/github.com/tdewolff/canvas?status.svg)](http://godoc.org/github.com/tdewolff/canvas)

Canvas is a common vector drawing target that can output SVG, PDF, EPS and raster images (which can be saved as PNG, JPG, ...). It can parse SVG path data or LaTeX into paths and has a wide range of path manipulation functionality (such as flattening, stroking and dashing). Text can be displayed using embedded fonts (TTF, OTF or WOFF) or by converting them to outlines and can be aligned and indenting within a rectangle.

![Example](https://raw.githubusercontent.com/tdewolff/canvas/master/example/example.png)

**Figure 1**: top-left you can see text being fitted into a box and their bounding box (orange-red), the spaces between the words on the first row are being stretched to fill the whole width. You can see all the possible styles and text decorations applied. Also note the typographic substitutions (the quotes) and ligature support (fi, ffi, ffl, ...). Below the text box, the word "stroke" is being stroked and drawn as a path. Top-right we see a LaTeX formula that has been converted to a path. Left of that we see ellipse support showcasing precise dashing, notably the length of e.g. the short dash is equal wherever it is (approximated through arc length parametrization) on the curve. It also shows support for alternating dash lengths, in this case (2.0, 4.0, 2.0) for dashes and for spaces. Note that the dashes themselves are elliptical arcs as well (thus exactly precise even if magnified greatly). In the bottom-right we see a closed polygon of four points being smoothed by cubic Béziers that are smooth along the whole path, and next to it on the left an open path.

**Terminology**: a path is a sequence of drawing commands (MoveTo, LineTo, QuadTo, CubeTo, ArcTo, Close) that completely describe a path. QuadTo and CubeTo are quadratic and cubic Béziers respectively, ArcTo is an elliptical arc, and Close is a LineTo to the last MoveTo command and closes the path (sometimes this has a special meaning such as when stroking). A path can consist of several path segments by having multiple MoveTos, Closes, or the pair of Close and MoveTo. Flattening is the action of converting the QuadTo, CubeTo and ArcTo commands into LineTos.

### Articles
* [Numerically stable quadratic formula](https://math.stackexchange.com/questions/866331/numerically-stable-algorithm-for-solving-the-quadratic-equation-when-a-is-very/2007723#2007723)
* [Quadratic Bézier length](https://malczak.linuxpl.com/blog/quadratic-bezier-curve-length/)
* [Bézier spline through open path](https://www.particleincell.com/2012/bezier-splines/)
* [Bézier spline through closed path](http://www.jacos.nl/jacos_html/spline/circular/index.html)
* [Point inclusion in polygon test](https://wrf.ecse.rpi.edu/Research/Short_Notes/pnpoly.html)

My own

* [Arc length parametrization](https://tacodewolff.nl/posts/20190525-arc-length/)

Papers

* [M. Walter, A. Fournier, Approximate Arc Length Parametrization, Anais do IX SIBGRAPHI (1996), p. 143--150](https://www.visgraf.impa.br/sibgrapi96/trabs/pdf/a14.pdf)
* [T.F. Hain, A.L. Ahmad, S.V.R. Racherla, D.D. Langan, Fast, precise flattening of cubic Bézier path and offset curves, Computers & Graphics 29 (2005). p. 656--666](https://www.sciencedirect.com/science/article/pii/S0097849305001287?via%3Dihub)
* [M. Goldapp, Approximation of circular arcs by cibic polynomials, Computer Aided Geometric Design 8 (1991), p. 227--238](https://www.sciencedirect.com/science/article/abs/pii/016783969190007X)
* [Drawing and elliptical arc using polylines, quadratic or cubic Bézier curves (2003), L. Maisonobe](https://spaceroots.org/documents/ellipse/elliptical-arc.pdf)

## Status
### Targets
| Feature | Image | SVG | PDF | EPS |
| ------- | ----- | --- | --- | --- |
| Transparency | yes | yes | yes | - |
| Draw path fill | yes | yes | yes | yes |
| Draw path stroke | yes | yes | yes | no |
| Draw path dash | yes | yes | yes | no |
| Embed fonts | - | yes | no | no |
| Draw text | - | yes | no | no |
| Draw image | no | no | no | no |

### Path
| Command | Flatten | Stroke | Length | SplitAt |
| ------- | ------- | ------ | ------ | ------- |
| LineTo  | yes     | yes    | yes    | yes     |
| QuadTo  | yes (CubeTo) | yes (CubeTo) | yes | yes (GL5 + Chebyshev10) |
| CubeTo  | yes     | yes    | yes (GL5) | yes (GL5 + Chebyshev10) |
| ArcTo   | yes | yes | yes (GL5) | yes (GL5 + Chebyshev10) |

* Ellipse => Cubic Bézier: used by rasterizer and PDF targets (Maisonobe)

NB: GL5 means a Gauss-Legendre n=5, which is an numerical approximation as there is no analytical solution. Chebyshev is a converging way to approximate a function by an n=10 degree polynomial. It uses the bisection method as well to determine the polynomial points.


## Planning
Features that are planned to be implemented in the future. Also see the TODOs in the code.

General

* Fix slowness, transparency and colors in the rasterizer
* Allow embedding raster images?

Fonts

* **Font embedding for PDFs and EPSs**
* Compressing fonts and embedding only used characters
* Use ligature tables
* Support WOFF2 font format
* Support Type1 font format?
* Support font hinting (for the rasterizer)

Paths

* Support Winding and EvenOdd fill rules
* Simplify polygons using the Ramer-Douglas-Peucker algorithm
* Intersection function between line, Bézier and ellipse and between themselves (for path merge, overlap/mask, clipping, etc.)
* Implement Bentley-Ottmann algorithm to find all line intersections (clipping)

Optimization

* Avoid overlapping paths when offsetting in corners (needs path intersection code)
* Approximate Béziers by elliptic arcs instead of lines when stroking, if number of path elements is reduced by more than 2 times (unsure if worth it)

Far future

* Support fill gradients and patterns (hard)
* Load in PDFs, SVGs and EPSs and turn to paths/texts
* Load in Markdown/HTML formatting and turn into texts


## Canvas
``` go
c := canvas.New(width, height float64)
c.SetColor(color color.Color)
c.DrawPath(x, y, rot float64, path *Path)
c.DrawText(x, y, rot float64, text *Text)

c.WriteSVG(w io.Writer)
c.WriteEPS(w io.Writer)
c.WritePDF(w io.Writer)
c.WriteImage(dpi float64) *image.RGBA
```

Canvas allows to draw either paths or text. All positions and sizes are given in millimeters.

## Text
![Text Example](https://raw.githubusercontent.com/tdewolff/canvas/master/example/text_example.png)

``` go
dejaVuSerif, err := canvas.LoadFontFile("DejaVuSerif", canvas.Regular, "DejaVuSerif.ttf")  // TTF, OTF or WOFF

ff := dejaVuSerif.Face(size float64)
ff.Info() (name string, style FontStyle, size float64)
ff.Metrics() Metrics                          // font metrics such as line height
ff.ToPath(r rune) (p *Path, advance float64)  // convert rune to path and return advance
ff.Kerning(r0, r1 rune) float64               // return kerning between runes

text := NewText(ff, "string")                                            // simple text with newlines
text := NewTextBox(ff, "string", width, height, halign, valign, indent)  // split on word boundaries and specify text alignment
text.Bounds() Rect
text.ToPath() *Path
text.ToSVG(x, y, rot, color) string  // convert to series of <tspan>

richText := NewRichText()                        // allow different FontFaces in the same text block
richText.Add(ff, "string")
text = richText.ToText(width, height, halign, valign, indent)
```


## Paths
A large deal of this library implements functionality for building paths. Any path can be constructed from a few basic operations, see below. The successive commands start from the current pen position (from the previous command's end point) and are drawn towards a new end point. A path can consist of multiple path segments (multiple MoveTos), but be aware that overlapping paths will cancel each other.

``` go
p := &Path{}
p.MoveTo(x, y float64)                                            // new path segment starting at (x,y)
p.LineTo(x, y float64)                                            // straight line to (x,y)
p.QuadTo(cpx, cpy, x, y float64)                                  // a quadratic Bézier with control point (cpx,cpy) and end point (x,y)
p.CubeTo(cp1x, cp1y, cp2x, cp2y, x, y float64)                    // a cubic Bézier with control points (cp1x,cp1y), (cp2x,cp2y) and end point (x,y)
p.ArcTo(rx, ry, rot float64, largeArc, sweep bool, x, y float64)  // an arc of an ellipse with radii (rx,ry), rotated by rot (in degrees CCW), with flags largeArc and sweep (booleans, see https://www.w3.org/TR/SVG/paths.html#PathDataEllipticalArcCommands)
p.Close()                                                         // close the path, essentially a LineTo to the last MoveTo location

p = Rectangle(x, y, w, h float64)
p = RoundedRectangle(x, y, w, h, r float64)
p = BeveledRectangle(x, y, w, h, r float64)
p = RegularPolygon(n int, x, y, r, rot float64)
p = Circle(x, y, r float64)
p = Ellipse(x, y, rx, ry float64)
```

We can extract information from these paths using:

``` go
p.Empty() bool
p.Pos() (x, y float64)         // current pen position
p.StartPos() (x, y float64)    // position of last MoveTo
p.Points() []Point             // positions of all commands
p.CCW() bool                   // true if the last path segment has a counter clockwise direction
p.Interior(x, y float64) bool  // true if (x,y) is in the interior of the path, ie. gets filled
p.Filling() bool               // true if the last path segment is filling
p.Bounds() Rect                // bounding box of path
p.Length() float64             // length of path in millimeters
p.ToSVG() string               // to SVG
p.ToPS() string                // to PostScript
```

These paths can be manipulated and transformed with the following commands. Each will return a pointer to the path.

``` go
p.Copy()
p.Append(q *Path)        // append path q to p
p.Join(q *Path)          // join path q to p
p.Split()                // split the path segments, ie. at Close/MoveTo
p.SplitAt(d ...float64)  // split the path at certain lengths d
p.Reverse()              // reverse the direction of the path

p.Translate(x, y float64)
p.Scale(x, y float64)
p.Rotate(rot, x, y float64)  // with the rotation rot in degrees, around point (x,y)

p.Flatten()                                            // flatten Bézier and arc commands to straight lines
p.Smoothen()                                           // treat path as a polygon and smoothen it by cubic Béziers
p.Stroke(width float64, capper Capper, joiner Joiner)  // create a stroke from a path of certain width, using capper and joiner for caps and joins
p.Dash(d ...float64)                                   // create dashed path with lengths d which are alternating the dash and the space

p.Optimize()  // optimize and shorten path
```


### Path stroke
Below is an illustration of the different types of Cappers and Joiners you can use when creating a stroke of a path:

![Stroke example](https://raw.githubusercontent.com/tdewolff/canvas/master/example/stroke_example.png)


## LaTeX
To generate outlines generated by LaTeX, you need `latex` and `dvisvgm` installed on your system.

``` go
p, err := ParseLaTeX(`$y=\sin\(\frac{x}{180}\pi\)$`)
if err != nil {
    panic(err)
}
```

Where the provided string gets inserted into the following document template:

``` latex
\documentclass{article}
\begin{document}
\thispagestyle{empty}
{{input}}
\end{document}
```


## Example
See https://github.com/tdewolff/canvas/tree/master/example for a working examples.

## License
Released under the [MIT license](LICENSE.md).
