scalafxml
=========

The [scalafx](https://code.google.com/p/scalafx/) library is a great UI DSL that wraps JavaFX classes and provides a nice syntax to work with them from Scala.

This library bridges [FXML](http://docs.oracle.com/javafx/2/fxml_get_started/why_use_fxml.htm) and [scalafx](https://code.google.com/p/scalafx/) by automatically building proxy classes, enabling a more clear controller syntax.

## Status

Currently the proxies are generated runtime. This has a disadvantage of having `scala-compiler.jar` as a dependency.

The library could be extended to use [macro annotations](http://docs.scala-lang.org/overviews/macros/annotations.html) for generating the proxies compile-time. This is an optimization and should not affect the FXML controller codes based on the library.    

## Example

The controller's, referenced from the FXML's through the `fx:controller` attribute, can be implemented as simple Scala classes, getting all the bound controls though the constructor:

```scala
import scalafx.scene.control.TextField
import scalafx.scene.control.Button
import scalafx.scene.control.ListView
import scalafx.event.ActionEvent

class TestController(
    private val input: TextField,
    private val create: Button,
    private val recentInputs: ListView[String],
    private val dep: AnAdditionalDependency) {

	// event handlers are simple public methods:
	def onCreate(event: ActionEvent) {
		// ...
	} 
}
```

Beside the JavaFX controls, additional dependencies can be injected to the controller as well. This injection process is extensible.

The following example uses [SubCut](https://github.com/dickwall/subcut) for injecting the additional dependency:

```scala
object SubCutDemo extends JFXApp {

  implicit val bindingModule = newBindingModule(module => {
    import module._

    bind [AnAdditionalDependency] toSingle(new AnAdditionalDependency("subcut dependency"))
  })

  stage = new JFXApp.PrimaryStage() {
    title = "Test window"
    scene = new Scene(
		FXMLView(
			getClass.getResource("test.fxml"), 
			new SubCutDependencyResolver()))            
  }  
}
```

but it is also possible to simply give the dependencies _by their type_ or _by their name_:

```
object SimpleDemo extends JFXApp {

  stage = new JFXApp.PrimaryStage() {
    title = "Hello world"
    scene = new Scene(
		FXMLView(getClass.getResource("startscreen.fxml"),
		    new DependenciesByType(Map(
		      typeOf[AnAdditionalDependency] -> new AnAdditionalDependency("dependency by type"))))
            
  }  
}
```