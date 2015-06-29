# ShadowedObjects #

ShadowedObjects is a library that adds object-level undo capabilities to
your custom business objects.

It requires no particular base class.

It can go to any depth, works with collections and dictionaries, and objects can be fully or parially "undone"

It works with any arbitrary class.



## Examples ##

### Simple Properties
```csharp
[Shadowed]
public class MyBusinessObject
{
    [Shadowed]
    public virtual string Name { get; set; }
}

var myObj = ShadowedObject.Create<MyBusinessObject>();
myObj.Name = "Pinky";
myObj.BaselineOriginals();
myObj.Name = "Brain";
myObj.ResetToOriginal(o => o.Name);
Assert.IsTrue(myObj.Name == "Pinky"); //true
```

### Sample Unit Tests
See Unit Test file for many more examples

```csharp
		[TestMethod]
		public void RecursionResetToOriginalTest()
		{
			var A = ShadowedObject.Create<TestLevelA>();
			A.NestedAs = new Collection<TestLevelA>();
			A.B = ShadowedObject.Create<TestLevelB>();
			A.B.Bstuff = 2;

			A.BaselineOriginals();
			A.B.BaselineOriginals();

			A.B.Bstuff = 3;

			//This WON'T work without recursion because the B here still is the original B, nothing to reset.
			//A.ResetToOriginal(a => a.B);
			A.ResetToOriginal();

			Assert.IsTrue(A.B.Bstuff == 2);
		}


		[TestMethod]
		public void CollectionResetToOriginalTest()
		{
			var A = new TestLevelA();
			A.NestedAs = new Collection<TestLevelA>();

			A = ShadowedObject.CopyInto(A);

			A.BaselineOriginals();

			A.NestedAs.Add(new TestLevelA());
			A.NestedAs.Add(new TestLevelA());

			A.ResetToOriginal(a => a.NestedAs[0]);

			Assert.IsTrue(A.NestedAs.Count < 1);
		}
```
