# Kotlin Guidelines
---------------------------

The Kotlin guidelines build off of the Android guidelines. Read those first.

## lateinit + ButterKnife
We use `lateinit` to get around null safety checks in Kotlin. It would look like this:

```kotlin
@BindView(R.id.toolbar) lateinit var toolbar: Toolbar
```
Note that we allow single line initialization here unlike in the Android Java guidelines due to the fact that the Kotlin formatter allows this.

## companion object
In order to still allow for static instance methods for `Activity`s and `Fragment`s, we use a companion object:

```kotlin
companion object {
    const val KEY_NAME = "name"

    fun newIntent(context: Context, name: String): Intent {
        val intent = Intent(context, AboutActivity::class.java)
	intent.putExtra(KEY_NAME, name)
        return intent
    }
}
```
and
```kotlin
companion object {

	const val KEY_MODE = "mode"

	fun newInstance(int mode): SomeFragment {
		val fragment = SomeFragment()
		val args = Bundle()
		args.putInt(KEY_MODE, mode)
		fragment.arguments = args
		return fragment
	}
}
```
Anything you would normally consider adding as a static variable in Java should be put in this companion object. This way, in Java, you can refer to it as you would a static variable or method. This companion object definition should go at the top of the Kotlin class due to the fact that it is similar to static variables and methods.

## Typecast + Fragment
Often times, you want to see if a fragment exists and created it if it does not. Make sure to use "safe" type casing when doing so. For example:
```kotlin
var feedFragment: FeedFragment? = supportFragmentManager.findFragmentByTag(TAG_FEED_FRAGMENT) as? FeedFragment
if (feedFragment == null) {
    feedFragment = FeedFragment.newInstance()
    supportFragmentManager.beginTransaction()
            .replace(R.id.root_fragment, feedFragment, TAG_FEED_FRAGMENT)
            .commit()
}
```

## Extensions
Extensions are great! Avoiding Util classes is great! The standard for now is to declare extensions for a class in a file with the convention of `<ClassName>.kt` within a package called `extension`. This file should only contain extension methods for the class the name correstponds to. 

This rule does not apply for library projects, as all extensions will probably just be within the same `extensions.kt` file.

### Data Classes + Extensions
If you have a model that acts just as a data holder, such as a model from an API, you should avoid putting custom methods in this classes, and instead add them as extensions. This easily seperates the model and its data from custom business logic.

## Avoiding interfaces for Listeners
As of yet, Kotlin does not support SAM-conversions for Kotlin. SAM-conversions only work for Java methods. Which is why you can do things like:
```kotlin
stepper.setOnClickListener { 
	//do things            
}
```
but for a Kotlin defined interface, you would have to do:
```kotlin
stepper.setOnClickListener(object : View.OnClickListener {
    override fun onClick(v: View?) {
	//do things
    }
})
```
For this reason, it is best to have methods accepting interfaces accept Lamda's instead:
```kotlin
fun setOnStepListener(listener: (count: Int) -> Unit) {
	onStepListener = listener
}

//later, to invoke the listener...
onStepListener?.invoke(nextAmount)
```
This way, the consumer code will look more like:
```kotlin
stepper.setOnStepListener { count ->
	//do something
}
```

## Ternary Operators
Since Kotlin has no such thing as Ternary operators `a ? b : c`, instead you can just do `if-else` flow all in one line:
```kotlin
view.visibility = if (loggedIn) View.VISIBLE else View.GONE
```
Keep in mind that if the flow extends beyond a simple assignment, you should use actual `if-else` flow on multiple lines.

## Null Checks
If you have a block of code where you do one thing if the value is null, and another thing if not null, you will likely need to create another refrence to that variable. For example:
```kotlin
if (user != null) {
	//IDE will give you an error about the potential of user to be null here
	println(user.name)
} else {
	println("no user found")
}
```
This is due to the fact that between the null check and the code within the if, user could be reassigned to null. There are a few solutions to this.
1. Reassignment to val
```kotlin
//rename to a different variable name
val currentUser = user
if (currentUser != null)
//.. etc
```
2. Run block
If you only care about is doing something in the non-null case, you can do a `run` block:
```kotlin
user?.let {
	println(it.name)
}
```
Notice the user of `it` in this case to refer to the non null variable.
