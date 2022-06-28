# Canvas HyperTxt 🚀📐✍

A zero dependency featherweight library to layout text on a canvas.

## Who is this for?

This library is inspired by the excellent [canvas-txt](https://canvas-txt.geongeorge.com) but focuses instead on being a part of the rendering pipeline instead of drawing the text for you. This offers greater flexibility for those who need it. Additionally canvas-hypertxt focuses on layout performance, allowing for much faster overall layout and rendering performance compared to the original library. Some sacrifices are made in the name of performance (justify).

## Comparison vs canvas-txt

While canvas-hypertxt does not set out to be a drop in replacement to canvas-txt, we can drag race them.

All tests were done with 5000 iterations. To ensure a fair comparison times include font rendering time.

|                       | canvas-txt | canvas-hypertxt | canvas-hypertxt w/ HyperWrapping |
| --------------------- | ---------: | --------------: | -------------------------------: |
| 20 char (no wrapping) |   0.05 sec |        0.05 sec |                         0.05 sec |
| 100 char              |   0.59 sec |        0.11 sec |                         0.11 sec |
| 300 char              |   2.63 sec |        0.40 sec |                         0.26 sec |
| 600 char              |   6.17 sec |        0.81 sec |                         0.47 sec |
| 1000 char             |  11.19 sec |        1.43 sec |                         0.77 sec |
| 1800 char (overflow)  |  22.47 sec |        2.29 sec |                         1.19 sec |

Benchmark code can be found [here](https://github.com/glideapps/canvas-hypertxt/blob/main/src/stories/benchmark.stories.tsx).

## How is this so much faster?

Canvas-txt is an excellent library but takes a very inefficient approach to finding wrap points. It is clearly not written with performance in mind, but rather with features and bundle size. Overall it is a fantastic library of raw speed is not important to your use case.

## HyperWrapping

One of the major items introduced by `canvas-hypertxt` is the concept of hyper wrapping. When enabled the font engine will train a weighting model to provide estimates for string sizes. Once the model is sufficiently trained it will perform string wrapping without calling `ctx.measureText` once. This leads to massive performance gains at the cost of accuracy. In practice with most fonts and text bodies, once trained the hyper wrap guesses will be withing 1% of the actual measured size. A buffer is added to ensure the text wraps slightly too early instead of clipping.

The end result is text that is correctly wrapped the vast majority of the time, with a very small number of errors where the text wraps too early by a single word. The performance gains in the measure pass are over 100x, with hyper wrapped text basically having zero cost vs unwrapped text of the same size.

## What am I missing using canvas-hypertxt?

You miss some features.

### Managed rendering

canvas-txt will rneder your string for you, figure out line heights, etc. With canvas-hypertxt this is on you. It's not hard to do but it is a bit of extra lifting. You could easily wrap the canvas-hypertxt split function to do the same thing canvas-txt does.

### Justify

I didn't feel like implementing it, patches welcome. This will 100% be slower that non-justified text due to the large amounts of string manipulation and extra measurement required. If you need justification canvas-txt can do it.

### Debug mode

Because canvas-hypertxt doesn't actually render the text, it also can't render debug boxes for you.

### Automatic text alignment

Text alignment with canvas-hypertxt is not as simple as setting a flag, though it's close. Set the `ctx.textAlign` appropriately and change the `x` value you pass to `fillText` to correspond to the left, center, or right edge of the bounding box.

### Why are these all missing?

canvas-txt is intended to be used as part of larger libraries which need wrapping text. In these cases text-alignment or actual text rendering is handled by existing functions which may provide additional functionality. By not rendering the text for the consumer, more flexibility is granted, however it comes at the cost of simplicity.
