---
title: 'GSoC 2019: CGAL - Shape Regularization'
date: 2019-09-05 18:07:34
tags:
---
### Project Overview
I participated in 2019 [Google Summer of Code (GSoC)](https://summerofcode.withgoogle.com/) by creating the [Shape regularization](https://cgal.geometryfactory.com/~danston/Shape_regularization/index.html) package for the [Computational Geometry Algorithms Library (CGAL)](https://www.cgal.org/) project by implementing the Generalized Global Regularization algorithm from the article ["KIPPI: Kinetic Polygonal Partitioning of Images"](https://hal.inria.fr/hal-01740958/file/kippi.pdf) written by Jean-Philippe Bauchet and Florent Lafarge at [Inria](https://www.inria.fr/en). Development for the package for CGAL was carried out under the supervision of [Dr. Dmitry Anisimov](https://www.anisimovdmitry.com/). The package comes with classes that work on a given set of 2D segments.

### About GSoC
[Google Summer of Code 2019](https://summerofcode.withgoogle.com/) was a 12 week, full time, remote, paid internship with regular evaluation of the participant's work every 4 weeks. I designed, implemented, tested and documented the package under the direction of my mentor [Dr. Dmitry Anisimov](https://www.anisimovdmitry.com/). 


### Shape Regularization Overview
The algorithm works as follows:

1. Computes neighbors of each item in the data set;
2. Computes objective function values among these neighbors;
3. Solves the quadratic programming problem utilizing the [OSQP](https://osqp.org/docs/) solver;
4. Sorts items into groups based on the regularized objective function values;
5. Updates items within each regularized group.

If the item type is a 2D segment, the package provides a few classes to reorient and realign them:
* Delaunay neighbor query – finds the neighbors of each segment by constructing a Delaunay triangulation using the CGAL::Delaunay_triangulation_2 class
* Angle regularization – re-orients segments to preserve parallelism and orthogonality.
* Ordinate regularization – re-aligns segments to preserve collinearity.

![](/images/angle_reg.png)
![](/images/ordinate_reg.png)

### Usage Example:
``` C++
#include <CGAL/Simple_cartesian.h>
#include <CGAL/Shape_regularization.h>
 
using Kernel = CGAL::Simple_cartesian<double>;
 
using FT        = typename Kernel::FT;
using Point_2   = typename Kernel::Point_2;
using Segment_2 = typename Kernel::Segment_2;
using Segments  = std::vector<Segment_2>;
 
using NQ = CGAL::Shape_regularization::Segments::Delaunay_neighbor_query_2<Kernel, Segments>;
using AR = CGAL::Shape_regularization::Segments::Angle_regularization_2<Kernel, Segments>;
using QP = CGAL::Shape_regularization::OSQP_quadratic_program<FT>;
using Regularizer = 
  CGAL::Shape_regularization::QP_regularization<Kernel, Segments, NQ, AR, QP>;
 
int main() {
 
  Segments segments = {
    Segment_2(Point_2(0.0, 0.0), Point_2(1.0, -0.1)),
    Segment_2(Point_2(1.0, 0.0), Point_2(1.0,  1.0)),
    Segment_2(Point_2(1.0, 1.0), Point_2(0.0,  1.1)),
    Segment_2(Point_2(0.0, 1.0), Point_2(0.0,  0.0))
  };
 
  NQ neighbor_query(segments);
  neighbor_query.create_unique_group();
  AR angle_regularization(
    segments, CGAL::parameters::all_default());
  angle_regularization.create_unique_group();
  
  QP quadratic_program;
  Regularizer regularizer(
    segments, neighbor_query, angle_regularization, quadratic_program);
  regularizer.regularize();
 
  std::cout << "* number of modified segments = " << 
    angle_regularization.number_of_modified_segments() << std::endl;
}

```

### Skills Applied
Technologies used: [C++11](https://en.cppreference.com/w/cpp/11), [git](https://git-scm.com/), [CMake](https://cmake.org/), [Doxygen](http://www.doxygen.nl/), [OSQP solver](https://osqp.org/docs/).
Concepts used: [Object-Oriented Design](https://en.wikipedia.org/wiki/Object-oriented_design), [Generic Programming](https://en.wikipedia.org/wiki/Generic_programming), [Quadratic Programming](https://en.wikipedia.org/wiki/Quadratic_programming).

### Links for Further Reading
Feel free to take a look at my [code](https://github.com/CGAL/cgal-public-dev/tree/gsoc2019-Shape_regularization-sytov/Shape_regularization), [GSoC2019 proposal](https://summerofcode.withgoogle.com/archive/2019/projects/6276807777583104/), [final submission and commit history](https://github.com/CGAL/cgal/issues/4170), [final report](https://docs.google.com/document/d/17Wvyb1ZiP8J-myLg9d0d7_X83xvxLeOOG13WVJyYwNM/edit) and the [documentation](https://cgal.geometryfactory.com/~danston/Shape_regularization/index.html).


<!-- Test:
<div class="table">
    <img src="/images/Seneca_blue_300.png" style="float: left;" width="35%"> 
    <img src="/images/Seneca_blue_300.png" style="float: left;" width="35%">
</div> -->
