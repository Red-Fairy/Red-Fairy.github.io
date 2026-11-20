---
layout: research-note
title: Depth from a sideways step
description: A pencil and a doorframe drift apart when you move your head. How much of that motion can a camera predict?
topic: Camera geometry
thumbnail: /assets/img/blog/parallax.png
template_preview: true
sitemap: false
---
{% comment %}
To use this note as a starting point, copy it to _posts/YYYY-MM-DD-your-title.md.
Remove template_preview and sitemap. Update the title, thumbnail, and article text. Set math: true for TeX equations.
{% endcomment %}

Hold a pencil at arm’s length and line it up with a distant doorframe. Close one eye. Now slide your head a little to the right, keeping your gaze pointed straight ahead. The pencil slips to the left of the frame. Both objects stayed still, yet their alignment changed.

That difference in apparent motion is **parallax**. We can predict its size with a pinhole camera and one sideways step.

<figure>
  <img src="{{ page.thumbnail | relative_url }}" width="960" height="480" alt="An overlay of two camera views. A nearby sphere shifts 80 pixels to the left, while a distant cube shifts 20 pixels in the same direction.">
  <figcaption>Two views overlaid. Dashed outlines mark the first view. Filled shapes mark the view after the camera moves 10 cm to the right.</figcaption>
</figure>

## From a step to a pixel shift

Put a point at horizontal position *X* and depth *Z* in camera coordinates. With focal length *f* measured in pixels, the point projects to *fX/Z* pixels from the image center. Move the camera right by *B* and keep its orientation fixed. The point’s horizontal position in the new camera frame becomes *X − B*, while its depth stays the same.

Subtract the first image position from the second and the horizontal position *X* drops out.

<div class="equation" role="math" aria-label="Horizontal pixel shift equals negative focal length times camera translation divided by depth">Δ<i>u</i> = −<i>fB</i> / <i>Z</i></div>

Take an 800-pixel focal length and a 0.10 m step. A point 1 m away moves 80 pixels to the left. A point 4 m away moves only 20 pixels. The **60-pixel difference** is the change in alignment you saw with the pencil and doorframe.

## What changes if the camera turns?

Try the thought experiment again with the camera rotating around its optical center. Two points that start on the same viewing ray stay on the same ray as each other. They can travel across the image together, but rotation alone cannot separate them by depth.

That gives a simple check for a generated camera move. Find a near edge that overlaps a distant one. During a sideways translation, watch how the overlap opens up. Even when the whole image drifts smoothly, that small patch can tell you whether the motion is consistent with the scene’s depth.

## Further reading

For the full projection equation, including focal length and the image center, see [OpenCV’s camera model](https://docs.opencv.org/4.x/d9/d0c/group__calib3d.html).
