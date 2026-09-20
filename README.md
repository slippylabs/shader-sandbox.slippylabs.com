# Shader Sandbox

Write a GLSL fragment shader and see it immediately — time, mouse and resolution uniforms, compile errors on the right line, presets and a share link. Runs entirely in your browser.

**Live:** <https://shader-sandbox.slippylabs.com/>

## What it does

- A live WebGL2 fragment shader editor, compiling as you type.
- Five uniforms and nothing else: `u_time`, `u_resolution`, `u_mouse`, `u_frame`, `fragColor`.
- Eight presets, from a two-line gradient to a raymarched sphere with a soft shadow, each written to be read.
- Compile errors listed with the line number **of your source**, not of the assembled file.
- Render scale from 0.25× to 2× supersampled, a pixel probe under the cursor, PNG export and a share link that carries the whole shader in the URL.

## How it works

Everything the page adds sits in one prelude — `#version`, two precision statements, the four uniforms and the output — and its length is the only number that matters afterwards, because the driver reports errors against the assembled file. That length is **derived from the prelude string** rather than written down, so the two cannot drift apart.

The prelude ends with `#line 1`, which on a conforming driver resets the counter so errors already come back relative to your source. Not every driver honours it, so the parser computes the shifted reading too and uses whichever one lands inside the source you can actually see.

A shader that fails to compile does **not** replace the running program. The new program is only swapped in once it has linked, so a typo leaves the last working picture on screen instead of a black rectangle.

Two details that catch people: the scene is drawn with one oversized triangle rather than two (no seam down the diagonal), and the mouse uniform is flipped into GL's bottom-left origin — passing the DOM's y straight through is the most common reason a shader reacts to the pointer upside down.

## Verification

The rendering claim is checked against a **CPU reference**. `verify_shader.py` compiles five shaders whose output is computable in closed form through the page's own WebGL path, reads the framebuffer back, and compares every pixel with the same maths in numpy:

| Shader | Pixels | Worst channel error |
|---|---|---|
| uv gradient | 100,925 | 0.50/255 |
| flat colour | 100,925 | 0.00/255 |
| circle mask | 100,925 | 0.00/255 |
| checkerboard | 100,925 | 0.00/255 |
| smoothstep ramp | 100,925 | 0.50/255 |

Half a quantisation step is exact agreement. (Headless Chromium renders this through SwiftShader, so it is a real GLSL ES 3.00 driver, not a stub.)

The error mapping is checked against **real driver output**: shaders with deliberate mistakes on known lines are compiled, and the reported line must land on the offending line and inside the user's source. A driver that ignores `#line` is simulated too. Plus: 14 share-link round trips including unicode and every base64 padding boundary, all eight presets compiling, and a confirmation that a failed compile leaves the previous image untouched.

**80 checks.**
