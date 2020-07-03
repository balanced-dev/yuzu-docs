# Integration
While usually Yuzu's Delivery side is happy with most situations the Definition can throw at it, there are a few caveats you should be aware of.

Ignoring these could cause the Umbraco developer in the team some headaches when it comes to integration time instead of seamless!

---

## Do not use "name"
However tempting it is to use, this does not play well with Umbraco's document types and so should be avoided.

Try to instead add a prefix to it like "employeeName" or use something like "title" instead.

---

## Do not use a property name that's the same as the block/page name
This one is fairly straightforward: do not use a property within a schema that is the same as the component name.

For example if you had a block called `parNavLinks` do not name the link array `navLinks`...
```json
{
	"id": "/parNavLinks",	
    "$schema": "http://json-schema.org/schema#",
	"type": "object",
	"additionalProperties": false,
	"properties": {
		"navLinks": {
			"type": "array",
            "items": {
                ...
            }
		}
        ...
	}
}
```

---

## Try not to nest properties more than 2 objects deep
With well-structured data, this is rarely an issue. However if you do nest properties within more than 2 sub-objects, without using something like an array to breaking the nesting up, then it can cause issues for the automatic Yuzu Umbraco integration/

Note: sub-block references are not effected by this rule- i.e. they do not count towards the parent's object nesting.

Also, some of the Definition examples break this rule but this is purely to illustrate how Handlebars etc. work and are often not best practice!

---

## Use the enum helper
Although you can render enum values happily without using the dedicated Yuzu helper in the Definition side, it is not possible in the .NET implementation and requires some some logic to work on the Delivery side.

?> Read about the [enum helper](https://yuzudocs.balanced.dev/#/definition/markup?id=enum)