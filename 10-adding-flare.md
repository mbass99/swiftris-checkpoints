>“It feels like one of those nights, we won't be sleeping.”<br>
>-- Taylor Swift, **22**

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/swiftris-final.gif)</center>

## Adding Flair

You've made it. It's a long and winding road filled with perilous pot holes and unnecessary pop culture references. At Bloc we know that programming is hard, but we try to make it fun and worth your effort.

After you complete this chapter, you'll have a functional version of Swiftris. Show it to your friends and family and maybe then they'll realize you are more than just free tech-support for their beige '98 Gateway desktop which you should've put out of its misery three service packs ago.

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/10-miserable-and-magical.jpg)</center>

Let's start by adding some flair to the screen. Even though Swiftris is calculating your score and level, it sure isn't showing it to you. Open `Main.storyboard` and re-shape your layout by selecting the following options:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/10-miserable-and-magical-xcode-1.png)</center>

Add a **View** object to your existing `View` by dragging it from the objects list. Click on the "Attributes Inspector" button and set its background to `default`, which is transparent. Click on the "Size Inspector" button and match the width, height and location attributes as shown below:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/10-miserable-and-magical-xcode-2.png)</center>

Add an **Image View** to the **View** element that you just created by dragging it into its visible area. In the "Size Inspector", set the image view's location to `0, 0` and have its height and width match the **View**'s: `84` points wide and `100` points tall. In the "Attributes Inspector", set the image view's **Image** property to `whitebg.png` as shown below:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/10-miserable-and-magical-xcode-3.png)</center>

Add a **Label** element to your view. In the "Size Inspector", set its location to `7, 20` and its width and height to `70, 21`. Match the label's properties with the following in the "Attributes Inspector":

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/10-miserable-and-magical-xcode-4.png)</center>

Copy the label you just created and place it right beneath the previous. In the "Size Inspector", set its location to `0, 45` and its size to `84, 39`. In the "Attributes Inspector", set its text to `999` and match its properties with the following image. Make sure that your layout hierarchy looks just like ours:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/10-miserable-and-magical-xcode-5.png)</center>

Select the **View** object you created a couple steps ago which now holds an image and two labels, copy it by pressing <key>⌘ + C</key> and paste it by pressing <key>⌘ + V</key>. Position this new **View** at `224, 237`. Change the `SCORE` label to `LEVEL` and both of the label colors to the following:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/10-miserable-and-magical-color-2.png)</center>

Your final storyboard should look something like this:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/10-miserable-and-magical-xcode-6.png)</center>

Open the **Assistant Editor** by clicking on the butler icon. Create an outlet for the first `999` label by <key>Ctrl + dragging</key> it from the storyboard editor into the properties section of `GameViewController`. Your outlet popup window should match the following:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/10-miserable-and-magical-xcode-7.png)</center>

Make sure to select the **Weak** option for **Storage**. Press connect. Repeat this step with the second `999` label and title it, `levelLabel`. Run Swiftris and you should see some great looking score and level labels ripe for the augmenting!

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/10-miserable-and-magical-preview-1.png)</center>

### Blow It Up

To make Swiftris truly great, it will need one thing: animations so explosive you have to walk away from them without looking back. Let's write some code which will drop our fallen blocks and blast destroyed blocks into the nebulous void:

```swift(GameScene.swift)
// #1
+    func animateCollapsingLines(linesToRemove: Array<Array<Block>>, fallenBlocks: Array<Array<Block>>, completion:() -> ()) {
+        var longestDuration: NSTimeInterval = 0
// #2
+        for (columnIdx, column) in fallenBlocks.enumerate() {
+            for (blockIdx, block) in column.enumerate() {
+                let newPosition = pointForColumn(block.column, row: block.row)
+                let sprite = block.sprite!
// #3
+                let delay = (NSTimeInterval(columnIdx) * 0.05) + (NSTimeInterval(blockIdx) * 0.05)
+                let duration = NSTimeInterval(((sprite.position.y - newPosition.y) / BlockSize) * 0.1)
+                let moveAction = SKAction.moveTo(newPosition, duration: duration)
+                moveAction.timingMode = .EaseOut
+                sprite.runAction(
+                    SKAction.sequence([
+                        SKAction.waitForDuration(delay),
+                        moveAction]))
+                longestDuration = max(longestDuration, duration + delay)
+            }
+        }

+        for rowToRemove in linesToRemove {
+            for block in rowToRemove {
// #4
+                let randomRadius = CGFloat(UInt(arc4random_uniform(400) + 100))
+                let goLeft = arc4random_uniform(100) % 2 == 0

+                var point = pointForColumn(block.column, row: block.row)
+                point = CGPointMake(point.x + (goLeft ? -randomRadius : randomRadius), point.y)

+                let randomDuration = NSTimeInterval(arc4random_uniform(2)) + 0.5
// #5
+                var startAngle = CGFloat(M_PI)
+                var endAngle = startAngle * 2
+                if goLeft {
+                    endAngle = startAngle
+                    startAngle = 0
+                }
+                let archPath = UIBezierPath(arcCenter: point, radius: randomRadius, startAngle: startAngle, endAngle: endAngle, clockwise: goLeft)
+                let archAction = SKAction.followPath(archPath.CGPath, asOffset: false, orientToPath: true, duration: randomDuration)
+                archAction.timingMode = .EaseIn
+                let sprite = block.sprite!
// #6
+                sprite.zPosition = 100
+                sprite.runAction(
+                    SKAction.sequence(
+                        [SKAction.group([archAction, SKAction.fadeOutWithDuration(NSTimeInterval(randomDuration))]),
+                            SKAction.removeFromParent()]))
+            }
+        }
// #7
+        runAction(SKAction.waitForDuration(longestDuration), completion:completion)
+    }
```

This is EPIC… At **#1** you can see that we take in precisely the tuple data which `Swiftris` returns each time it removes a line. This will ensure that `GameViewController` will pass those elements to `GameScene` for them to animate properly.

For the blocks which must now fall to their new locations, we cascade them from left to right. We begin by iterating column by column, block by block at **#2**. We also established a `longestDuration` variable which will determine precisely how long we should wait before calling the `completion` closure.

To keep the blocks from looking robotic, they will fall after one another rather than all at once. At **#3** we wrote code which will produce this pleasing effect for eye balls to enjoy. Based on the block and column indices, we introduce a directly proportional delay.

When removing lines at **#4**, we make their blocks shoot off the screen like explosive debris. To achieve this we will use a `UIBezierPath`. Our arch requires a radius and we've chosen to generate one randomly to introduce a natural variance among the explosive paths. Furthermore, we've randomized whether the block flies left or right.

At **#5** we choose beginning and starting angles. The angles are in radians and if your trigonometry is as rough as ours was when we wrote this, a circle in radian degrees, or *unit circle*, looks like this:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/10-miserable-and-magical-unit-circle.gif)</center>

>Riveting…

When going left, we begin at `0` radians and end at `π`. When going right, we go from `π` to `2π`. Math for the win!

At **#6** we place the block sprite above the others such that they animate above the other blocks and begin the sequence of actions which concludes with removing the sprite from the scene.

Lastly, at **#7** we run the `completion` action after a duration matching the time it will take to drop the last block to its new resting place.

### Sound Is Good

Remember those wonderful mp3 files we imported eons ago? We're going to put those to use. Add a couple lines to your `GameScene` file to support sound playback as well as an amazing theme song.

```swift(GameScene.swift)
        shapeLayer.position = LayerPosition
        shapeLayer.addChild(gameBoard)
        gameLayer.addChild(shapeLayer)
// #8
+       runAction(SKAction.repeatActionForever(SKAction.playSoundFileNamed("Sounds/theme.mp3", waitForCompletion: true)))
    }

// #9
+   func playSound(sound:String) {
+       runAction(SKAction.playSoundFileNamed(sound, waitForCompletion: false))
+   }
```

Our theme song is incredible and we must play it ad infinitum. At **#8** we set up a looping sound playback action of our theme song. At **#9** we added a method which `GameViewController` may use to play any sound file on demand.

Run Swiftris and listen to the soothing tones of soviet propaganda. Workers of the world, *unite!* Note that your blocks won't explode yet; we need to write a little more code in the next section to finish that animation.

>Fun fact, the word **robot** stems from the Russian word for **worker**, **rabotnik**. Languages are fun!

### The Last Piece

We can feel your excitement. We are actually tracking it by a thermal camera we've installed on your computer and beamed directly to Bloc's excitement-tracking servers. Those same servers operate a pair of subwoofers each the size of a racquetball court.

Our entire office gets that boom, boom pow when an eager programmer reaches their ultimate destination. It's pretty intense and yes our liability policy is extensive.

All we need now is to hook up our logic and scene together to make a beautiful game:

```swift(GameViewController.swift)
    func gameDidBegin(swiftris: Swiftris) {
+       levelLabel.text = "\(swiftris.level)"
+       scoreLabel.text = "\(swiftris.score)"
+       scene.tickLengthMillis = TickLengthLevelOne

        // The following is false when restarting a new game
        if swiftris.nextShape != nil && swiftris.nextShape!.blocks[0].sprite == nil {
            scene.addPreviewShapeToScene(swiftris.nextShape!) {
                self.nextShape()
            }
        } else {
            nextShape()
        }
    }
```

When the game begins, we reset the score and level labels as well as the speed at which the ticks occur, beginning with `TickLengthLevelOne`.

```swift(GameViewController.swift)
    func gameDidEnd(swiftris: Swiftris) {
        view.userInteractionEnabled = false
        scene.stopTicking()
+       scene.playSound("Sounds/gameover.mp3")
+       scene.animateCollapsingLines(swiftris.removeAllBlocks(), fallenBlocks: swiftris.removeAllBlocks()) {
+           swiftris.beginGame()
+       }
    }
```

After the game ends, we'll play the designated game over sound; *that's a must*. Then we destroy the remaining blocks on screen before starting a brand new game with no delay. The show must go on.

```swift(GameViewController.swift)
    func gameDidLevelUp(swiftris: Swiftris) {
+        levelLabel.text = "\(swiftris.level)"
+        if scene.tickLengthMillis >= 100 {
+            scene.tickLengthMillis -= 100
+        } else if scene.tickLengthMillis > 50 {
+            scene.tickLengthMillis -= 50
+        }
+        scene.playSound("Sounds/levelup.mp3")
    }
```

Each time the player levels up, we'll decrease the tick interval. At first, each level will decrease it by `100` milliseconds, but as it progresses it will go even faster, and topping off at `50` milliseconds between ticks. Lastly, we play a congratulatory level up sound. We have to reward players with *something*, after all.

```swift(GameViewController.swift)
    func gameShapeDidDrop(swiftris: Swiftris) {
        scene.stopTicking()
        scene.redrawShape(swiftris.fallingShape!) {
            swiftris.letShapeFall()
        }
+       scene.playSound("Sounds/drop.mp3")
    }
```

Just a couple small changes here, but it's the little details in life that matter. Letting the player see and *hear* the result of their efforts as well is like giving them the virtual proud father figure they never had.

```swift(GameViewController.swift)
    func gameShapeDidLand(swiftris: Swiftris) {
        scene.stopTicking()
-       nextShape()
+       self.view.userInteractionEnabled = false
// #10
+       let removedLines = swiftris.removeCompletedLines()
+       if removedLines.linesRemoved.count > 0 {
+           self.scoreLabel.text = "\(swiftris.score)"
+           scene.animateCollapsingLines(removedLines.linesRemoved, fallenBlocks:removedLines.fallenBlocks) {
// #11
+               self.gameShapeDidLand(swiftris)
+           }
+           scene.playSound("Sounds/bomb.mp3")
+       } else {
+           nextShape()
+       }
    }
```

When a shape lands either naturally on its own or after a drop, it's time to check for completed lines. We invoke `removeCompletedLines` at **#10** to recover the two arrays from `Swiftris`. If `Swiftris` removed any lines, we update the score label to represent the newest score and then animate the blocks with our explosive new animation function.

After the animation completes, we perform a *recursive* call at **#11**. A recursive function is one which invokes itself. In Swiftris' case, after the blocks have fallen to their new location, they may have formed brand new lines.

After we remove the first set of lines, we invoke `gameShapeDidLand(Swiftris)` again to detect any new lines. If we find none, we bring in the next shape.

### Thank You!

For sticking with us this long. It's been a doozy, and I'm being earnest when I write that I'm not sure how you put up with it. Oh yeah, the puppy gifs:

<center>![](http://bloc-books.s3.amazonaws.com/swiftris/10-miserable-and-magical-cute-puppy-gifs-towel.gif)</center>

Thanks again and enjoy Swiftris!

#### Even More

Want to learn more? Design your first website, [Jottly](https://www.bloc.io/tutorials/jottly-beginner-s-guide-to-html-css-skeleton-and-animation-css). It's super cool and just as fun!

Or perhaps you'd like to design a [cool music app website](http://bloc-jams.webflow.com/)... with NO code! How is this possible? Check out our [Webflow tutorial](https://www.bloc.io/tutorials/webflow-tutorial-design-responsive-sites-with-webflow) to find out.

If you enjoyed this book, please share it with your friends and followers.
