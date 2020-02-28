# Follow me on [Github](https://github.com/darktheft) 
# Subscribe me on [Youtube](https://www.youtube.com/channel/UCdbUomS4yFXN9D3_ZLtJJUg)
![youtube](https://github.com/darktheft/FlashLogin/blob/master/donation.png)
## on [twiter](https://twitter.com/iamdarktheft)
## on [instagram](https://www.instagram.com/iamdarktheft)

![FlashLogin](https://github.com/darktheft/FlashLogin/blob/master/1_dbZwGGyhE5pk_KgNZ_GW8w.gif)

## May be Transition framework?
Android Transition framework works great but we need something other behavior.
We need to start transition animation for selected recycler item, translation for toolbar,
alpha and scale for recycler in RecyclerFragment at the same time. And we want to get it working at least on API 19. 
The most simplest solution is to using ViewPropertyAnimator and android transition tricks.

## All by steps
It’s pretty simple:
 #### Follow me 
*1  Calculate selected recycler card end position (its position on DerailsFragment);
 
2  Save card current position and pass it to DetailsFragment as arguments (if we want revers transition on API < 21);
3   Create a copy of selected recycle card (lets name it new card);
4   Make selected recycler card invisible;
5   Add new card (from step 3) to the root layout of RecyclerFragment;
6   Animate all other views and translate new card;
7   Create fragment transaction and show DetailsFragment when animation ends;
8   Animate DetailsFragment views.

## Views animation
For toolbar animation we’ll create a helper view above toolbar in RecyclerFragment layout file and set its initial translation by Y out of the screen. This view will be animated into DetailsFragment toolbar container (blue color on above screenshot) with ViewPropertyAnimator.

```
<View
    android:id="@+id/details_toolbar_helper"
    android:layout_width="wrap_content"
    android:layout_height="@dimen/details_toolbar_container_height"
    android:background="@color/colorPrimary"
    app:layout_constraintTop_toTopOf="parent"/>
// In RecyclerFragment
details_toolbar_helper.translationY = -details_toolbar_helper.height
```
![intro](https://miro.medium.com/max/360/1*lzbPi7Nih8ICfJQQl0wfQw.gif)

BottomNavigationView and RecyclerView are animated with ViewPropertyAnimator in the same way but without helper views, it’s very simple (just scale, fade and translate).

## Transition trick

In simple words, android transition framework when starts a shared element transition just creates copy of transition views content (like print screen) and put them into ImageViews, then adds these ImageViews at the same positions into the root overlay layer of target fragment (or activity) and starts translate animations.

For us it’s not a case, because when android transition framework starts transition it destroys all other views and we can not animate them. But we want simultaneously animations: animate recycler, toolbar, navigation bar and card transition at the same time.

We can do our animations by adding selected recycler item’s view into the RecyclerFragment root view overlay layer and then translate it. But as per ViewGroupOverlay add(view: View) method documentation:

```
If the view has a parent, the view will be removed from that parent before being added to the overlay.
```
But for RecyclerView it’s not working. Selected view will not be removed from recycler after adding it into overlay.

![intro2](https://miro.medium.com/max/360/1*hA3JPVQEp4tgPRlVYYrYxQ.gif)

What we have

![intro3](https://miro.medium.com/max/360/1*7jfUEBJg8Y7TfORDYjXv8w.gif)

To achieve the expected result we just copy content (like print screen) from the selected recycler item, add put that print screen into ImageView and setup its layout parameters and position.

```
private fun createCopyView(view: View): View {
    val copy = copyViewImage(view)

    // On preLollipop when we create a copy of card view's content its shadow is copied too
    // and we do not need additional card view. 
  
    return (if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
        CardView(view.context).apply {
            cardElevation = resources.getDimension(R.dimen.card_elevation)
            radius = resources.getDimension(R.dimen.card_corner_radius)
            addView(copy)
        }
    } else {
        copy
    }).apply {
        layoutParams = view.layoutParams
        layoutParams.height = view.height
        layoutParams.width = view.width
        x = view.x
        y = view.y + toolbar.height
    }
}

private fun copyViewImage(view: View): ImageView {
    val copy = ImageView(view.context)

    val bitmap = Bitmap.createBitmap(view.width, view.height, Bitmap.Config.ARGB_8888)
    val canvas = Canvas(bitmap)
    view.draw(canvas)

    copy.setImageBitmap(bitmap)
    return copy
}
```

```
Why we can’t just translate a selected view without creating a copy?
Because the recycler will also be animated and all its views become invisible and we can’t see the selected item transition animation.
```

After that we can add a new copied view into the RecyclerFragment root view and translate it to the target position.


```

override fun onClick(view: View) {
    val fragmentTransaction = initFragmentTransaction(view)
    val copy = createCopyView(view)
    root.addView(copy)
    view.visibility = View.INVISIBLE
    startAnimation(copy, fragmentTransaction)
}
```

And this is what we have now:

![in](https://miro.medium.com/max/360/1*VPAGfUJsuZBpHudiR7LPwg.gif)

Not bad!

## We’re in the homestretch

The last step is to begin fragment transaction as animator end action:

```
.withEndAction {
    fragmentTransaction?.commitAllowingStateLoss()
}
```

```
Why do we use commitAllowingStateLoss?
Because if there is an orientation change during the animation, we’ll get an IllegalStateExсeption. This is a good article about this.
```
All that’s left to do is run the view animation in DetailsFragment. It’s very easy to do with ViewPropertyAnimator in way like in RecyclerFragment.

For the returning transition we at first animate all views and then the android transition framework transit our shared elements.

### And let’s play all animations together:

![in](https://miro.medium.com/max/320/1*qrJlj35yyqNTdY2Q0wbLfA.gif)

Not exactly the same as on the UI template created by Ivan Parfenov, but I think looks nice.

# Follow me on [Github](https://github.com/darktheft) 
                                                
## on [twiter](https://twitter.com/iamdarktheft)
## on [instagram](https://www.instagram.com/iamdarktheft)
