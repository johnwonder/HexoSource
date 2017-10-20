title: scala_mock
date: 2017-03-31 14:04:04
tags: scala
---

## scala mock

读了这篇文章( Mock Objects for Scala and Android)[https://www.artima.com/scalazine/articles/borachio_mock_objects_for_scala.html],对scala mock有点了解了。  

```java
trait Turtle {
  def penUp()
  def penDown()
  def forward(distance: Double)
  def turn(angle: Double)
  def getAngle: Double
  def getPosition: (Double, Double)
}
```

This API is not very convenient. We have no way to move to a specific position, instead we need to work out how to get from where we are now to where we want to get by calculating angles and distances. The diagram below, for example, demonstrates the movements a turtle starting at the origin (0, 0) would need to make to draw a line from (1, 1) to (2, 1).

Turtle diagram

You can see an example of code that performs these calculations here. This isn’t trivial, so we want to test to make sure that it’s doing the right thing. Here’s a test (written with ScalaTest) that creates a mock turtle that pretends to start at the origin (0, 0) and verifies that if we ask the code we’ve just written to draw a line from (1, 1) to (2, 1) it performs the correct sequence of turns and movements:

```java
class MockFunctionTest extends Suite with MockFactory {

  val mockTurtle = mock[Turtle]
  val controller = new Controller(mockTurtle)

  def testDrawLine() {
    inSequence {
      mockTurtle expects 'getPosition returning (0.0, 0.0)
      mockTurtle expects 'getAngle returning 0.0
      mockTurtle expects 'penUp
      mockTurtle expects 'turn withArgs (~(Pi / 4))
      mockTurtle expects 'forward withArgs (~sqrt(2.0))
      mockTurtle expects 'getAngle returning Pi / 4
      mockTurtle expects 'turn withArgs (~(-Pi / 4))
      mockTurtle expects 'penDown
      mockTurtle expects 'forward withArgs (1.0)
    }

    controller.drawLine((1.0, 1.0), (2.0, 1.0))
  }
}
```
