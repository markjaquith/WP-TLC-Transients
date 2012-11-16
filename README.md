# TLC Transients

A WordPress transients interface with support for soft-expiration (use old content until new content is available), background updating of the transients (without having to wait for a cron job), and a chainable syntax that allows for one liners.

## Examples

In this simple example, we're defining a feed-fetching callback, and then using `tlc_transient` with a chain to point to that callback and use it, all in one line. Note that since we haven't used `background_only()`, the initial load of this **will** cause the page to pause.

```php
<?php
// Define your callback (other examples use this)
function my_callback() {
	return wp_remote_retrieve_body(
		wp_remote_get( 'http://example.com/feed.xml', array( 'timeout' => 30 ) )
	);
}

// Grab that feed
echo tlc_transient( 'example-feed' )
	->updates_with( 'my_callback' )
	->expires_in( 300 )
	->get();
?>
```
This time, we'll set `background_only()` in the chain. This means that if there has been a hard cache flush, or this is the first-ever request, it will return false. So your code will have to be written to gracefully degrade if the feed isn't yet available. This, of course, triggers a background update. And once it is available, it will start returning the content.

```php
<?php
echo tlc_transient( 'example-feed' )
	->updates_with( 'my_callback' )
	->expires_in( 300 )
	->background_only()
	->get();
?>
```

We don't have to chain, of course.

```php
<?php
$t = tlc_transient( 'example-feed' );
if ( true ) {
	$t->updates_with( 'my_callback' );
} else {
	$t->updates_with( 'some_other_callback' );
}

$t->expires_in( 300 );
echo $t->get();
?>
```
We can even pass parameters to our callback.

```php
<?php
// Define your callback
function my_callback_with_param( $param ) {
	return str_replace(
		'foo',
		$param,
		wp_remote_retrieve_body( wp_remote_get( 'http://example.com/feed.xml', array( 'timeout' => 30 ) ) ),
	);
}

// Grab that feed
echo tlc_transient( 'example-feed' )
	->updates_with( 'my_callback_with_param', array( 'bar' ) )
	->expires_in( 300 )
	->background_only()
	->get();
?>
```
