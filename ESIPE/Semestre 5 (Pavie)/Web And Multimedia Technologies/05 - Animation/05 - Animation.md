[[Web And Multimedia Technologies]]
17/10/2024
****

It is important to understand that what we refer to as "animations" is different from videos.
Usually, animations are artificially-made.

****
## Bitmap Animation

**Animated GIFs** are composed by a sequence of ordinary still GIF images. Each image has a timer which indicates for how long it should be displayed (this allows slow animations, but also fast animations).
It is the most common format for bitmap animations.
	*As we mentioned, GIF files were made to be small. Nowadays though, most GIF files are very heavy. This behaviour is due to the fact that they are animated GIFs that contains a large sequence of still GIF images, which ends up forming a heavy GIF file...*

You cannot really know in advance if a GIF is animated or not. You have to open it on the browser.


**Tweening** is a mechanism that automatically inserts a series of frames between an **initial frame** and a **final frame**.
This allows us to manage the position and opacity of an object between two states (initial and final frames).
	*This allows smoother animations in case of movements, for instance*
![[tweening.png]]


****
## Vector Animation

Adobe flash was allowing the creation of sophisticated vector animations (and even entire websites), where **objects** could be programmed via the **ActionScript language**.

Flash has vanished, but Adobe Animate is now here to help web designers building HTML animations. Google Web Designer is the free alternative of Adobe Animate.


Unlike bitmap animations which are a simple sequence of bitmap images, a vector animation is more of a **description**.
This description codes the shapes (properties of a circle, a square...), the **keyframes** and **Frames Per Second (FPS)**.

![[keyframes.png]]
