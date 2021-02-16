# Mixed Reality Toolkit Hand Pose Detector for HoloLens 2

This is a very small hand pose detector for use with the HoloLens 2 and the [Mixed Reality Toolkit](https://github.com/Microsoft/MixedRealityToolkit-Unity).
The code is very minimal and the approach is extremely simpleminded; my only claim is that it works well enough for
prototyping.

## The code

This repository consists of a complete Unity MRTK project for HoloLens 2. The MRTK
documentation above describes how to set up MRTK on your system; I won't duplicate
that information here. I will say that if you pull, build, and run this, you should
see something similar to [this very brief demo video](https://twitter.com/MCSpaceCadet/status/1361515238219083776).

Almost all of this repository is the MRTK code itself; the only actual new code here is
in [the HandPose directory](Assets/HandPose), in particular the [HandPoseClassifier.cs](Assets/HandPose/HandPoseClassifier.cs) class.

## The poses

The hand poses this library currently attempts to detect are:

- _Open_: All fingers spread, as if waving.
- _Closed_: All fingers in a fist.
- _PointingIndex_: Pointing with index finger only.
- _PointingIndexAndMiddle_: Pointing with index and middle fingers (or at least, extending both index and middle fingers).
- _PointingMiddle_: Pointing with middle finger only. (This is both rude and happens to be unreliable, so maybe best to just
  ignore this.)
- _Flat_: All fingers (and thumb) flat.
- _Bloom_: All fingertips together, pointing upwards, palm below.

Recognition of all of these is best if all of your fingers are visible, though the code makes some attempt to guess
correctly if your closed fist is facing away from the HoloLens device (e.g. with the back of your hand towards your
face, fingers occluded).

Experimentation in the course of developing this code showed that various variations of these gestures wouldn't work; for
example, the HoloLens 2 can't see the "Vulcan salute" gesture (index and middle fingers together, ring and pinky fingers
together), and nor can it reliably tell the difference between extending your index finger only, and your middle finger
only.

## The very basic algorithm

The algorithm is terribly simple and in fact could be slightly simpler -- some of the code here was experimental and is
only barely used. The core idea is:

- Calculate finger pose, where finger may be "Extended", "NotExtended", or "Unknown".
  - This is done by calculating how linear the finger's joint positions are, simply by projecting the
    normalized vectors between each adjacent finger bone (e.g. determining the joint angles).
- Calculate, for each finger, how aligned it is with a ray from the eye to that finger's knuckle.
  - Fingers which are aligned with the eye->knuckle ray are almost certainly occluded by the palm.
- Calculate, for each pair of fingers, how co-linear they are.
  - This is done by projecting the normalized knuckle->fingertip vectors of both fingers onto each other.
- Calculate how far apart the fingertips are (in total), how far apart the knuckles are, and how far above
  the knuckles the fingertips are (as a ratio of the inter-knuckle distance).
  - This is used for detecting the "bloom" gesture.

Given these finger poses, eye->finger occlusions, and finger co-linearities, we calculate the poses above quite
straightforwardly.

## Known issues

The approach here uses ratios between different parts of the user's hand, instead of absolute measurements, and
so should be more robust in the presence of different hand sizes. Nonetheless, the tuning constants here are
potentially fragile and I haven't done extensive testing.

As with all gestural interfaces, accessibility is a problem; someone with limited hand mobility or missing digits
may be unable to make these gestures at all.

The whole approach here is fundamentally limited in that it is not Bayesian and has no current notion of partial
hand poses (e.g. "mostly open" or "almost pointing") -- poses are either classified or not, with no intermediate
probabilities. In general, deep learning would probably be a better way to solve this whole problem, but at least
this method is sufficient for my current prototyping with the HoloLens 2.
