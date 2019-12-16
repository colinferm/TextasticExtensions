# Code Completion Documentation
Code completion is, in theory, simple but, in actuality, rather powerful in Textastic.

To generate a unique UUID, please see the developer's [documentation](https://github.com/blach/Textastic-Customization).

# File Format
```
{
	"description": "",
	"uuid": "",
	
	"completionSets": [
		{
			"name": "aribitrary-set",
			"defaultAppend": " type=\"${1:value}\" $0 />",
			"strings": [
				"script",
				{
					"string": "ref"
					"append": ">\n$0\n</ref>\n"
				}
			]
		}
	],
	contexts": [
		{
			"description": "",
			"scope": "(text.html | source.wiki) - source - comment - string",
			"pattern": "<([a-zA-Z0-9]*)",
			"completionCaptureIndex": 1,
			"completionSetNames": [
				"aribitrary-set"
			]
		}
	]
}
```

# Keys
**description** - String - Describing what your code completion does.

**uuid** - String - A UUID Generated as Per the Docs

**completionSets** - Array - of completion objects.

*completion*.**name** - String - An arbitrary name for this set.

*completion*.**defaultAppend** - String - What to append if the strings below offer nothing.

*completion*.**strings** - Array - of either string literals or appending descriptions.

*appending*.**string** - String - What to autocomplete, functionally the same as just a string would be.

*appending*.**append** - String - What to append after the autocomplete.

*appending*.**replace** - String - What to replace with matched string with (works in the JavaScript example but doesn't seem to work in HTML tags.

**contexts** - Array - of context objects in which to fire the autocomplete.

*context*.**description** - String - An arbitrary name for this context.

*context*.**scope** - String - a definition of the scope in which this should execute. See the discussion below.

*context*.**pattern** - String - a javascript regular expression to pattern match what to autocomplete.

*context*.**completionCaptureIndex** - Integer - Which captured set to pass onto the completion sets.

*context*.**completionSetNames** - Array - names that corresponde to what has been defined in *completion.name*.

In either *defaultAppend* or *context.append*, ```$0``` is where the cursor should end up at the end. ```$1``` or ```${1:value}``` is where the cursor should go in squential order when tabbing through the defined fields, with or without placeholder text. Should the variable be reused in the replacement, all places where it is used will be updated when the first is changed.

## Completion Set Names
Some clever naming of the completion sets can allow them to be called contextually. For the sake of discussion, let's assume you want to auto-complete HTML attribute values. If you have the pattern:

```
"pattern": "<([a-zA-Z0-9]*)(?:\\s+[a-zA-Z0-9-:_]+\\s*=\\s*(?:'.*'|\".*\"))*(?:\\s([a-zA-Z0-9-]+)\\s*=\\s*(?:[\"'])([a-zA-Z0-9-]*))",
```

This results in three capture groups. The first would be the tag name, the second would be the attribute and the third would be what it attempts to autocomplete. A developer can be clever if they only want specific values for specific attributes for specific tags. One *could* set up the *completionSetName* to carry these groups so it will dynamically look to the right set for autocomplete values.

For example: (prefacing the names with a namespace because it's always polite)

```
tags.link.attr
tags.link.rel.values
tags.img.attr
tags.img.class.values
```

This way, if certain values should populate into the *class* attribute but not, say, the width attribute, the *completionSetNames* should be as simple as:

```"tags.$1.$2.values"```

And if an ```<img class="``` is encountered and a set is defined as ```tags.img.class.values```, it will feed autocomplete values from that set and no others.

# Discussion
The two main things to discuss are *replace* in the *appending* object and *scope* in the *context* object.

## Replace
If someone looks at the js.json file in the original [documentation](https://github.com/blach/Textastic-Customization), the "forin" function serves as the example for how "replace" should work. It overwrites that key that triggered the "completion" and inserts the associated text in its place. Unfortunately, when it comes to HTML tags, this developer has not been successful in getting it to work.

## Scope
Scope remains a bit of a black box. Without official documentation from the developer, the best this developer can do is to guess as to the meanings and describe what has been discovered.

For the sake of this discussion, let's assume this is the scope line:
```(text.html | source.wiki) - source - comment - string```

Scope seems to begin with the type of source that it affects. This, for instance:

```(text.html | source.wiki)``` tells the autocorrect code that the patterns should be activated on HTML and Mediawiki documents. It is important to note that while HTML documents are "text.html", most other types of text Textastic can edit are defined as "source.extension", so "source.js" or "source.objc".

It is unknown how many pipes can be placed between scope sources.

```source - comment - string``` seem to describe *where* the autocomplete regular expression will activate. But how this is fundamentally different than ```meta.tag string.quoted``` remains unknown.

However, should autocompletions not be activating and the regular expression is confirmed to be valid, check the scope and experiment with values as seen in the samples.