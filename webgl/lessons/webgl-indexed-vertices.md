Title: WebGL2 Indexed Vertices
Description: How to use gl.drawElements
TOC: Indexed Vertices (gl.drawElements)

This article assumes you've at least read
[the article on fundamentals](webgl-fundamentals.html). If
you have not read that yet you should probably start there.

This is short article to cover `gl.drawElements`. There are 2
basic drawing functions in WebGL. `gl.drawArrays` and `gl.drawElements`.
Most of the articles on the site that explicitly call one or the other
call `gl.drawArrays` as it's the most straight forward.

`gl.drawElements` on the other hand uses a buffer filled with
vertex indices and draws based on that.

Let's take the example that draws rectangles from
[the first article](webgl-fundamentals.html) and make it use
`gl.drawElements`

In that code we created a rectangle from 2 triangles, 3 vertices
each for a total of 6 vertices. 

Here was our code that provided 6 vertex positions

```js
  var x1 = x;
  var x2 = x + width;
  var y1 = y;
  var y2 = y + height;
  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([
     x1, y1,   // vertex 0
     x2, y1,   // vertex 1
     x1, y2,   // vertex 2
     x1, y2,   // vertex 3
     x2, y1,   // vertex 4
     x2, y2,   // vertex 5
  ]), gl.STATIC_DRAW);
```

We can instead use data for 4 vertices

```js
  var x1 = x;
  var x2 = x + width;
  var y1 = y;
  var y2 = y + height;
  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([
     x1, y1,  // vertex 0
     x2, y1,  // vertex 1
     x1, y2,  // vertex 2
     x2, y2,  // vertex 3
  ]), gl.STATIC_DRAW);
```

But, then we need to add another buffer with indices because WebGL still
requires that to draw 2 triangles we must tell it to draw 6 vertices in total.

To do this we create another buffer but we use a different binding point.
Instead of the `ARRAY_BUFFER` binding point we use the `ELEMENT_ARRAY_BUFFER`
binding point which is always used for indices.

```js
// create the buffer
const indexBuffer = gl.createBuffer();

// make this buffer the current 'ELEMENT_ARRAY_BUFFER'
gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, indexBuffer);

// Fill the current element array buffer with data
const indices = [
  0, 1, 2,   // first triangle
  2, 1, 3,   // second triangle
];
gl.bufferData(
    gl.ELEMENT_ARRAY_BUFFER,
    new Uint16Array(indices),
    gl.STATIC_DRAW
);
```

Like all data in WebGL we need a specific representation for the 
indices. We convert the indices to unsigned 16 bit integers with
`new Uint16Array(indices)` and then upload them to the buffer.

It's important to note that unlike the `ARRAY_BUFFER` binding point
which is global state, the `ELEMENT_ARRAY_BUFFER` binding point is
part of the current vertex array. 

In the code we created and bound a vertex array and then setup
the index buffer. That means, like the attributes, anytime we
bind this vertex array the index buffer will be also be bound.
See [the article on attributes](webgl-attributes.html) for more info.

At draw time we call `drawElements`

```js
// Draw the rectangle.
var primitiveType = gl.TRIANGLES;
var offset = 0;
var count = 6;
-gl.drawArrays(primitiveType, offset, count);
+var indexType = gl.UNSIGNED_SHORT;
+gl.drawElements(primitiveType, count, indexType, offset);
```

We get the same results as before but we only had to supply data for 4
vertices instead of 6. We still had to ask WebGL to draw 6 vertices but let
us reuse data for 4 vertices through the indices.

{{{example url="../webgl-2d-rectangles-indexed.html"}}}

Whether you use indexed or non indexed data is up to you.

It's important to note that indexed vertices won't usually let you make a cube
with 8 vertex positions because generally you want to associate other data with
each vertex, data that is different depending on which face that vertex position 
is being used. For example if you wanted to give each face of the cube a different
color you'd need to provide that color with the position. So even though the
same position is used 3 times, once for each face a vertex touches, you'd still
need to repeat the position, once for each different face, each with a different
associated color.

Valid types for `indexType` above are `gl.UNSIGNED_BYTE`
where you can only have indices from 0 to 255, in which case you'd use `new Uint8Array(indices)`,
`gl.UNSIGNED_SHORT` where the maximum index is 65535 which we used above, 
and `gl.UNSIGNED_INT` which has a maximum index of 4294967296 and where you'd use
`new Uint32Array(indices)`.

<div class="webgl_bottombar">
<p>
Note: Above we bind the `indexBuffer` to the `ELEMENT_ARRAY_BUFFER` binding point when
putting the indices in the buffer then we bind it again later. Why bind it twice?
</p>
<p>
This is only to show a pattern.
Typically you'd be drawing more than a single thing so you'd have multiple
index buffers, one for each thing you want to draw. At init time you'd create
these buffers and put data in them. At render time, before you draw each individual
thing you'd need to bind the correct buffer. So, the code here follows that pattern
even though it's only drawing one thing.
</p>
</div>
