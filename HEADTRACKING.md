# 2 - Vocable AAC: head tracking

<!-- wp:image {"align":"center"} -->
<figure class="wp-block-image aligncenter"><img src="https://www.vocable.app/static/assets/showcase-1.png" alt="Select preset phrases from multiple categories to have conversations with loved ones."/></figure>
<!-- /wp:image -->

<p align="center">Vocable AAC</p>

<!-- wp:columns -->
<div class="wp-block-columns"><!-- wp:column -->
<div class="wp-block-column"><!-- wp:post-navigation-link {"type":"previous","label":"\u0026lt;\u0026lt; Previous Post"} /--></div>
<!-- /wp:column -->

<!-- wp:column -->
<div class="wp-block-column"><!-- wp:post-navigation-link {"textAlign":"right","label":"Next Post \u0026gt;\u0026gt;"} /--></div>
<!-- /wp:column --></div>
<!-- /wp:columns -->

<!-- wp:paragraph -->
<p>So I know you're all wondering how Vocable AAC works and how does it follow your face. You'll need to know a bit of Android development (kotlin) in order to follow some of the below, but if you don't understand it that's okay you can still enjoy the story and I'll put in a little separator for you to know when you can pick up reading again if you're not into the technical details.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>To be frank, on iOS, I have no idea. I think they used ARKit, and that's all I really know. I know one of the developers who worked on this also works on droids, like you know the ones from Star Wars? I know they put in the same technology that allows their BB-8 droid to move and stop gracefully into the app so that when you move your face it will also stop in a way that is pleasant and not so herky jerky. I know that they worked for a long time on head tracking so make it smooth and beautiful.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>If you want to take a look and figure out how iOS does head tracking you can look at their codebase yourself at their iOS <a rel="noreferrer noopener" href="https://github.com/willowtreeapps/vocable-ios" target="_blank">GitHub</a>.</p>
<!-- /wp:paragraph -->

<!-- wp:separator {"className":"is-style-wide"} -->
<hr class="wp-block-separator is-style-wide"/>
<!-- /wp:separator -->

<!-- wp:paragraph -->
<p>I can tell you how it was built on Android because I helped make that. If you want to follow along and take a look at the code you can find that here on the Android <a href="https://github.com/willowtreeapps/vocable-android" target="_blank" rel="noreferrer noopener">Github</a>. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Basically we used ARCore's <a rel="noreferrer noopener" href="https://developers.google.com/ar/develop/java/augmented-faces" target="_blank">Augmented Faces</a>. We used Google's documentation to help us put augmented faces into the app and then we created a <code>BaseActivity.kt</code> that handles creating a <code>PointerView.kt</code> (the little orange circle) and attaches it to a <code>FaceTrackingViewModel.kt</code> That viewModel is in charge of most of the stuff from Augmented Faces and it lets us know where the tip of the person's nose is on the screen.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":51,"sizeSlug":"large","linkDestination":"none"} -->
<figure class="wp-block-image size-large"><img src="https://spudsdevblog.files.wordpress.com/2021/10/googs.png?w=824" alt="" class="wp-image-51"/></figure>
<!-- /wp:image -->

<!-- wp:code -->
<pre class="wp-block-code"><code>val pose = augmentedFace.getRegionPose(AugmentedFace.RegionType.NOSE_TIP)</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>We use a function called <code>updatePointer(x: FLoat, y: Float)</code> to update the pointer when the viewModel tells us the person's nose has moved. The <code>BaseActivity.kt</code> has the current view that the user is on and lets us know if the pointer is within the view with <code>findIntersectingView()</code> and <code>viewIntersects(view1: View, view2: View): Boolean</code></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>The thing is, we got all this and when we went to test it, it didn't really work. It was very herky jerky and it only sort of followed where your face was, but it didn't do a very good job and the pointer was going all over the place. We worked on this for maybe a week before we asked others for help. We were kind of worried because if we didn't figure out how to make it smoother and more reliable we maybe wouldn't have an Android app. Another coworker figured it out:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>val zAxis = pose.zAxis
val x = zAxis&#91;0]
var y = zAxis&#91;1]
val z = <strong><span style="text-decoration:underline;">-zAxis&#91;2]</span></strong></code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>The bold, underlined piece is what we were missing. So we get the pose's z axis:</p>
<!-- /wp:paragraph -->

<!-- wp:quote -->
<blockquote class="wp-block-quote"><p>[A zAxis] returns a 3-element array containing the direction of the transformed Z axis.</p><cite>google developer documentation</cite></blockquote>
<!-- /wp:quote -->

<!-- wp:paragraph -->
<p>and we flip the third part of that so that it's actually facing the screen instead of away from it. Then we use <a rel="noreferrer noopener" href="https://developers.google.com/sceneform/reference/com/google/ar/sceneform/math/Vector3#public-static-vector3-lerp-vector3-a,-vector3-b,-float-t" target="_blank">interpolation</a> to get the new vector and pass it to the viewModel so that it can be observed and we can move the pointer.</p>
<!-- /wp:paragraph -->

<!-- wp:separator {"className":"is-style-wide"} -->
<hr class="wp-block-separator is-style-wide"/>
<!-- /wp:separator -->

<!-- wp:image {"align":"center","id":52,"sizeSlug":"large","linkDestination":"none"} -->
<figure class="wp-block-image aligncenter size-large"><img src="https://spudsdevblog.files.wordpress.com/2021/10/vocable-1.png?w=591" alt="" class="wp-image-52"/><figcaption>We flipped the z of the z axis vector... kinda confusing but promise it makes sense</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph {"align":"left"} -->
<p class="has-text-align-left">This makes sense because the Augmented Faces technology is the same technology that allows you to put stuff on your face in pictures like:</p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"center","id":53,"sizeSlug":"large","linkDestination":"none"} -->
<figure class="wp-block-image aligncenter size-large"><img src="https://spudsdevblog.files.wordpress.com/2021/10/aumentedfaces.png?w=664" alt="" class="wp-image-53"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Also, they say in their documentation: </p>
<!-- /wp:paragraph -->

<!-- wp:quote -->
<blockquote class="wp-block-quote"><p>The positive Z-axis (Z+) points out from the face towards the nose</p></blockquote>
<!-- /wp:quote -->

<!-- wp:paragraph -->
<p>This was a little confusing to us, but now instead of using the positive z-axis, we're using the negative so that it faces towards the screen. Our app is saved, we can now do a little bit of clean up on head tracking and we have Vocable AAC for Android.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"align":"left"} -->
<p class="has-text-align-left">If you want, you can install it <a rel="noreferrer noopener" href="https://play.google.com/store/apps/details?id=com.willowtree.vocable&amp;hl=en_US&amp;gl=US" target="_blank">here</a> on your Android device or if you have an iOS device you can get it <a rel="noreferrer noopener" href="https://apps.apple.com/us/app/vocable-aac/id1497040547" target="_blank">here</a>. </p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"center","id":407,"sizeSlug":"full","linkDestination":"none"} -->
<figure class="wp-block-image aligncenter size-full"><img src="https://spudsdevblog.files.wordpress.com/2021/10/pxl_20211231_053833748.mp_.jpg" alt="" class="wp-image-407"/><figcaption>Sleepy pep</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph {"align":"center"} -->
<p class="has-text-align-center">I'll talk about our process and why we did what we did for the features we have on Vocable AAC currently in my next blog post. See you there :]</p>
<!-- /wp:paragraph -->
