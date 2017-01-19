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
Extensions are great! Avoiding Util classes is great! The standard for now is to declare extensions for a class in a file with the convention of `<ClassName>Extensions.kt` For example, all `Activity` extensions should be in a file `ActivityExtensions` so that it is obvious and easy to find where the extensions is coming from.

This rule does not apply for library projects, as all extensions will probably just be within the same `extensions.kt` file.

## Avoiding interfaces for Listeners
As of yet, Kotlin does not support SAM-conversions for Kotlin. SAM-conversions only work for Java methods. Which is why you can do things like:
```kotlin
stepperAdults.setOnClickListener { 
	//do things            
}
```
but for a Kotlin defined interface, you would have to do:
```kotlin
stepperAdults.setOnClickListener(object : View.OnClickListener {
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
