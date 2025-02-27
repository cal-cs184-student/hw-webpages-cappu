Team Cappuccino

Lixin Ruan

## Overview:

In this assignment I have implemented a basic rasterizer that can render basic triangles, interpolated color triangles and textured triangles, meanwhile enabled the rasterizer with multiple sampling methods, including multi-level supersampling and multi-level mipmap sampling.

## Task 1:

In this task, the triangles are rasterized with a basic direct sampling, wich means having one sample point per each pixel on the screen. The essence of this process is to do triple **line perpendicular test** and use the positivity of the results to decide wether the point is on the edge, inside the triangle or outside the triangle.

### Main Steps

The algorithm specified in the `RasterizerImp::rasterize_triangle` are designed in following steps:

1. Initialize the variables used in the algorithm
   - `Vector2D v[3]`: An array to store the coordinates of the three vertexes.
   - `size_t hUpperBound, hLowerBound, wLowerBound, wUpperBound`: the boundaries of the traverse area.
2. Calculate the square bounding box for the triangle.
3. Traverse the area by testing each point.
   1. Do the line perpendicular test of the point and each edge with a helper function `float LinePerpendicularTest().`
   2. If the test results includes zeroes, the point is judged to be on edges. 
      1. According to the openGL/Direct3D's rule, judge whether the vertex is belonged to a top-left edge or not, and if it is an end point or not. (the method would be specified later)
      2. Fill the pixel accordingly if it belongs to the top-left edges.
   3. If the test results are all positive or negative, the point is judged to be inside the triangle.
      1. Fill the pixel accordingly if it belongs to the top-left edges.

### Mind the gap!

- The order of the verticies is not a matter since one point's line perpendicular results can be all negative to prove it is in the triangle.(This would happen in clockwise scenario.)

### Key math/code

​	**Line perpendicular test:**

```c++
float LinePerpendicularTest(const Vector2D& p, const Vector2D& p0, const Vector2D& p1) {
    return (p.y - p0.y) * (p1.x - p0.x) - (p.x - p0.x) * (p1.y - p0.y);
  }
```

​	**Top-left check:**

1. Campare the verticies to see if the coresponding edge is horizontal or left.
2. Check if the point is an end point or not.

```c++
if (v[i].y == v[(i + 1) % 3].y) {
  if (points[j].x != v[(i + 1) % 3].x) { //check if it is the end point or not
    fill_pixel(j % static_cast<int>(sqrt(sample_rate)) + w * static_cast<int>(sqrt(sample_rate)), j / static_cast<int>(sqrt(sample_rate)) + h * static_cast<int>(sqrt(sample_rate)), color, true);
    break;
  }
} else if (v[i].x > v[(i + 1) % 3].x) {
  fill_pixel(j % static_cast<int>(sqrt(sample_rate)) + w * static_cast<int>(sqrt(sample_rate)), j / static_cast<int>(sqrt(sample_rate)) + h * static_cast<int>(sqrt(sample_rate)), color, true);
  break;
} else if (v[i].x == v[(i + 1) % 3].x) {
  if (v[i].y > v[(i + 1) % 3].y) {
    fill_pixel(j % static_cast<int>(sqrt(sample_rate)) + w * static_cast<int>(sqrt(sample_rate)), j / static_cast<int>(sqrt(sample_rate)) + h * static_cast<int>(sqrt(sample_rate)), color, true);
    break;
  }
}
```

## Task2

In this task, I have mainly modified `void RasterizerImp::rasterize_triangle` , `void RasterizerImp::fill_pixel()`, `void RasterizerImp::resolve_to_framebuffer()` . The main modification happened in the data structures and down-sampling in resolve function while the algorithm steps remained the same with task1. (For line perpendicular test and edge point check.)

### Modifications

The modification happened in `sample_buffer` . It is resized to `width * height * sample_rate`. 

In `void RasterizerImp::rasterize_triangle` , all of the points are upsampled by adding a sample point traverse inside each actual pixel traverse (of the bound box).  Following steps are added:

1. Calculate segments (the gap between each sample points) in float.
2. Create a local sample point set for each pixel `Vector2D points[sample_rate]`
3. Traverse each point.
4. *Following the steps in Task1*
5. **The fill_pixel invoke is modified to enter the correct position in buffer of each sample point.**

In `void RasterizerImp::fill_pixel()`, I added an input `is_superSample` at the end of parameter list, default value as false, to ensure point rasterization and line rasterization remain normal. Additionaly, for invokations in `rasterize_triangle` where `is_superSample = true`, the `x, y` are inputted in upsampling space. For  `is_superSample = false`, they are in normal pixel space. It is following steps:

1. Check  if `is_superSample = true`
   1. Dirrectly fill the buffer corresponding with input x, y.
2. Else
   1. Calculate the coordinates of all sample points in the corresponding pixel.
   2. Traverse and fill the corresponding buffer position.

In `RasterizerImp::resolve_to_framebuffer()` , it downsamples `sample_buffer` into `rgb_framebuffer_target` in following steps:

1. Traverse pixel sapce.
2. Add color values at all sample points.
3. Devide the sum by sample rate, to get the color of each pixel.

## Task3



## Task4

The essence of barycentric coordinate is the ratio calculated by the distance of the point to the three vertices, making each point inside a triangle locally smooth transisting.



## Task5

### Pixel Sampling

**Nearest Sampling:**

For each sample point in pixel space, find its nearest texel point on the texture and use that color precisely.

**Bilinear Sampling:**

For each sample point in pixel space, sample it with 4 closest texel point on the texture. The colors of 4 texels are processed with 3 times of linear interpolation by creating two Lerps between two pairs of texels, then interpolating between the two Lerps.

### Implementation

The texture sampling is mainly implemented in `Color Texture::sample_bilinear()` and `Color Texture::sample_nearest()`.

Nearest implementation steps:

1. Multiply the uv coordinates with width and height.
2. Round the result and shift the rounding in two axes accordinglly with the relative directrion of uv coords and result.
3. Get the color from texture

Bilinear implementation steps:

1. Multiply the uv coordinates with width and height.
2. Round the result and shift the rounding in two axes accordinglly with the relative directrion of uv coords and result.
3. Get the color sfrom texture

### Task6

### Level Sampling

Is a method which produces multiple texture file with different resolution to filter out high frequency data for minification. (Cases that one sample point gets to many pixels)

