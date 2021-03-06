# Compiler

New language features, faster compilation, stricter checking; `javac` saw some changes and it looks like some of them may cause new compilation errors.

## Type Inference

It looks like the Java 9 compiler's type inference works differently in some cases, which can lead to new compile errors.
All files in this project compile fine with Java 8 but fail with Java 9.
Look for comments `// fail` to see what doesn't work and `// pass` for fixes that make it work in Java 9.

Some errors can come from [casts like `(int) Comparable<T>`](src/main/java/wtf/java9/compiler/CastWildcardParam.java):

```java
public void spinnerModel(SpinnerNumberModel spinnerModel) {
	// this could have happened somewhere else:
	// spinnerModel.setMinimum(0);
	// spinnerModel.setMaximum(360);
	// spinnerModel.setValue(45);

	int newValue = (int) spinnerModel.getValue() + 1;
	newValue = Math.min(newValue, (int) spinnerModel.getMaximum()); // fail
	newValue = Math.max(newValue, (int) (Object) spinnerModel.getMinimum()); // pass
	spinnerModel.setValue(newValue);
}
```

Also, some [shenanigans with generic arrays](src/main/java/wtf/java9/compiler/GenericArray.java) stop working:

```java
public static <I> Optional[] lift(I[] array) { ... }
public static <T> T[] flatten(Optional<T>[] array) { ... }

private Stream<T[]> streamSelectionPaths() {
	Object[][] things2d = new Object[0][0];
	return Arrays
			.stream(things2d)
//			.map(thing -> (Optional<T>[]) lift(thing)) // pass
			.map(GenericArray::lift) // fail
			.map(GenericArray::flatten);
}
```

(Last checked: 8u152 and 9.0.1)
