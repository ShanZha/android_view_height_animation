Android View height animation
=============================

Explores an approach to animate View height changes. The root reason is to find a way to animatedly add/remove views from the screen so their container also changes its height animatedly.

There doesn't seem to be an easy way to do this through the Android API, because the animation just changes the rendering matrix of the view, not the actual size.
