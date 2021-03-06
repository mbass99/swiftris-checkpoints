>“Any intelligent fool can make things bigger, more complex, and more violent. It takes a touch of genius -- and a lot of courage -- to move in the opposite direction.”<br>
>-- Albert Einstein

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/09-honestys-too-much-final.gif)</center>

## Touch Me, Move Me

*Let's get physical*, dear reader. Now that you've had your fill of vaguely touch-related references, it's time to touch the screen. No, take your fingers off of your computer monitor, we meant your iPhone's screen. Swiftris does not force the user to idly watch as shapes descend with no purpose. That's modern art and, as Indiana Jones would say, "It belongs in a museum!"

To make Swiftris interactive, we'll use three `UIGestureRecognizer` objects. They will inform our `GameViewController` when a user interaction occurs, specifically those which Swiftris may take advantage of. Let's begin by adding an `UITapGestureRecognizer` to our view. Open `Main.storyboard`. Your screen should resemble the following:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/09-honestys-too-much-xcode-1.png)</center>

Under the objects browser, find the **Tap Gesture Recognizer** object and drag it into the **Game View Controller Scene** entry. Refer to the screenshot below:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/09-honestys-too-much-xcode-2.png)</center>

Open the **Assistant Editor** by pressing the little bow-tie butler icon, if the butler does his job your screen should resemble ours:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/09-honestys-too-much-xcode-3.png)</center>

In this mode, we'll be able to attach the tap gesture recognizer to actual functions inside of our `GameViewController`. Do so by holding <key>Ctrl</key> and click-dragging from the **Tap Gesture Recognizer** entry towards an empty spot in `GameViewController`:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/09-honestys-too-much-xcode-4.png)</center>

Your **Connection** should be of type **Action**, name the function `didTap`, and change `AnyObject` to `UITapGestureRecognizer`. Press connect. This should add the following function signature to your `GameViewController` class:

```swift(GameViewController.swift)
    @IBAction func didTap(sender: UITapGestureRecognizer) {

    }
```

The gesture recognizer will invoke this function when it recognizes a tap. Return to the **Standard Editor** and <key>right-click</key> on the **Tap Gesture Recognizer** entry. The following window should appear:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/09-honestys-too-much-xcode-5.png)</center>

Click and drag beginning at the location specified in the screen cap. This will assign the `GameViewController` as the tap gesture recognizer's delegate, much like how `GameViewController` is the `Swiftris` class' delegate. Let's prepare `GameViewController` to handle some tapping:

```swift(GameViewController.swift)
-class GameViewController: UIViewController, SwiftrisDelegate {
+class GameViewController: UIViewController, SwiftrisDelegate, UIGestureRecognizerDelegate {
```

```swift(GameViewController.swift)
 @IBAction func didTap(sender: UITapGestureRecognizer) {
+    swiftris.rotateShape()
 }
```

Run Swiftris and click on the screen or tap your iPhone if you're running it on a device. Watch your hard work come together in rotational bliss:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/09-honestys-too-much-demo-1.gif)</center>

### Things Are Panning Out

Repeat the steps above verbatim except this time, use a **Pan Gesture Recognizer**. Then, make sure your `didPan(UIPanGestureRecognizer)` function matches the following:

```swift(GameViewController.swift)
    var scene:GameScene!
    var swiftris:Swiftris!
// #1
+   var panPointReference:CGPoint?
```

```swift(GameViewController.swift)
    @IBAction func didPan(sender: UIPanGestureRecognizer) {
// #2
+        let currentPoint = sender.translationInView(self.view)
+        if let originalPoint = panPointReference {
// #3
+            if abs(currentPoint.x - originalPoint.x) > (BlockSize * 0.9) {
// #4
+                if sender.velocityInView(self.view).x > CGFloat(0) {
+                    swiftris.moveShapeRight()
+                    panPointReference = currentPoint
+                } else {
+                    swiftris.moveShapeLeft()
+                    panPointReference = currentPoint
+                }
+            }
+        } else if sender.state == .Began {
+            panPointReference = currentPoint
+        }
    }
```

Our pan detection logic is straight-forward. Every time the user's finger moves more than 90% of `BlockSize` points across the screen, we'll move the falling shape in the corresponding direction of the pan. At **#1** we keep track of the last point on the screen at which a shape movement occurred or where a pan begins.

At **#2** we recover a point which defines the translation of the gesture relative to where it began. This is not an absolute coordinate, just a measure of the distance that the user's finger has traveled.

At **#3** we check whether the `x` translation has crossed our threshold - 90% of `BlockSize` - before proceeding.

If it has, we check the velocity of the gesture at **#4**. Velocity will give us direction, in this case a positive velocity represents a gesture moving towards the right side of the screen, negative towards the left. We then move the shape in the corresponding direction and reset our reference point.

Run Swiftris now and go nuts... just nuts. Click (or tap) and drag your shape wildly. Combine this experience with a loud witch-like cackle to optimize your joy.

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/09-honestys-too-much-demo-2.gif)</center>

### Swipe for Your Life

Once again, we'll have to repeat the wonderful steps for implementing a new recognizer. Add a **Swipe Gesture Recognizer** to your view, set the **Game View Controller** as its `delegate` and add an **Action** named `didSwipe`. We need to perform one extra step. Highlight the **Swipe Gesture Recognizer** in your storyboard editor and make sure **Down** is the selected **Swipe** option.

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/09-honestys-too-much-xcode-6.png)</center>

Make the following changes to `GameViewController`:

```swift(GameViewController.swift)
    @IBAction func didSwipe(sender: UISwipeGestureRecognizer) {
+        swiftris.dropShape()
    }

// #5
+    func gestureRecognizer(gestureRecognizer: UIGestureRecognizer, shouldRecognizeSimultaneouslyWithGestureRecognizer otherGestureRecognizer: UIGestureRecognizer) -> Bool {
+        return true
+    }

// #6
+    func gestureRecognizer(gestureRecognizer: UIGestureRecognizer, shouldBeRequiredToFailByGestureRecognizer otherGestureRecognizer: UIGestureRecognizer) -> Bool {
+        if gestureRecognizer is UISwipeGestureRecognizer {
+            if otherGestureRecognizer is UIPanGestureRecognizer {
+                return true
+            }
+        } else if gestureRecognizer is UIPanGestureRecognizer {
+            if otherGestureRecognizer is UITapGestureRecognizer {
+                return true
+            }
+        }
+        return false
+    }
```

```swift(GameViewController.swift)
    func gameShapeDidDrop(swiftris: Swiftris) {
// #7
+        scene.stopTicking()
+        scene.redrawShape(swiftris.fallingShape!) {
+            swiftris.letShapeFall()
+        }
    }
```

At **#5**, `GameViewController` will implement an optional delegate method found in `UIGestureRecognizerDelegate` which will allow each gesture recognizer to work in tandem with the others. At times, a gesture recognizer may collide with another.

Sometimes when swiping down, a pan gesture may occur simultaneously with a swipe gesture. In order for these recognizers to relinquish priority, we will implement another optional delegate method at **#6**. The code performs `is` conditionals. These conditionals check whether the generic `UIGestureRecognizer` parameters is of the specific types of recognizers we expect to see. If the check succeeds, we execute the code block.

Our code lets the pan gesture recognizer take precedence over the swipe gesture and the tap to do likewise over the pan. This will keep all three of our recognizers from bickering with one another over who's the prettiest API in the room.

At **#7** we stop the ticks, redraw the shape at its new location and then let it drop. This will in turn call back to `GameViewController` and report that the shape has landed. Run Swiftris and launch those blocks to inconceivable speeds!
