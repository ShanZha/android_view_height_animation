# Android View height animation

Explores an approach to animate `View` height changes. The root intent was to find a way to animatedly add/remove views from the screen so their container also changes its height animatedly.

## The issue

There doesn't seem to be an easy way to do this through the Android API, because the animation from the API (`ScaleAnimation`, `TranslateAnimation`) just changes the rendering matrix of the view, but not the actual size.

## The solution

There is a custom animation class that does the trick: [ChangeYSizeAnimation](https://github.com/vitkhudenko/android_view_height_animation/blob/master/src/vit/android/test/height/animation/ChangeYSizeAnimation.java)

Similar to native animations that change some view property, it changes height and top/bottom margins for the view. Each change ends with a `View.requestLayout()` call.

`ChangeYSizeAnimation` accepts `HeightSpec` and `MarginSpec` instances as parameters. Those incapsulate the target height and top/bottom margins respectively.


### Collapse animation

Lets look at collapse animation first, because it is a bit easier. We know that on the end of the collapse animation each child being removed should have 0 height and 0 top/bottom margins.

So we create corresponding specs and start `ChangeYSizeAnimation` for each view being removed. They change their size and thus the size of the container also changes producing the collapsing effect.

On finish of the collapsing animation we remove the views from the container.


### Expand animation

This is a bit trickier than collapsing, because we somehow need to know the target height for views being added. Those may not be constants, e.g. if view height is declared as `"wrap_content"`.

The trick to find the future height was found on StackOverflow: http://stackoverflow.com/a/4950968/247013

There is also one extra point - unless we do something then on finish of the expanding animation the view being added is left with the constant height, say 73 px, even if initial intent was to give it a "floating" height ("wrap_content" in layout xml).

That is why there are also 2 sublclasses of `HeightSpec` - `MatchParentHeightSpec` and `WrapContentHeightSpec`. Unlike base `HeightSpec` they instruct to apply `ViewGroup.LayoutParams.MATCH_PARENT` or `ViewGroup.LayoutParams.WRAP_CONTENT` height on the animation end.

## Limitations

1. Because unlike the built-in animations we do change the view hierarchy, then I am sure this approach "eats" more CPU/battery. On some less powerfull devices (or if there are hundreds of child views) it probably could be non-smooth. I have not investigated what's the number of views to get this kind of issue.

2. Another point to mention is that this project has been created without any support for horizontally oriented `ViewGroup`.