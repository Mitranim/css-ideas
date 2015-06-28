# Description

Ideas for useful features missing from the CSS spec.

## `as-if` property

Makes the matched element look _as if_ it also matched the selectors listed
in the property.

Speaking in procedural terms, the rule adds the listed to the currently matched
selector, then recalculates the element's styles. This has the potential to make
several passes, adding more selectors with each pass. User agents will need to
guard against infinite recursion by ignoring duplicates.

Borrowing an example from below:

```css
input[type=checkbox]: checked + label {
  as-if: [theme=primary] .active;
}
```

### Example

Suppose we have a colour theme defined as follows.

```css
[theme=primary] {
  color: blue;
}
[theme=primary] .active {
  color: darkblue;
}
```

We might use it like so:

```html
<nav theme="primary">
  <a class="active">first</a>
  <a>second</a>
</nav>
```

Suppose we want a pair of `input[type=checkbox] + label` to take advantage of
the theme, so that `:checked` makes the label use the already defined `.active`
state.

```html
<form theme="primary">
  <input checked type="checkbox" id="1"><label for="1">I'm selected</label>
  <input type="checkbox" id="2"><label for="2">I'm not</label>
</form>
```

Currently, you have to add an exception to the theme definition, duplicating
the colour:

```css
[theme=primary] {
  input[type=checkbox]:checked + label {
    color: darkblue;
  }
}
```

With the proposal, you can make it behave _as if_ it matched the given selector
combination:

```css
input[type=checkbox]: checked + label {
  as-if: [theme=primary] .active;
}
```

This way, you can reuse rules in cases where duplication is otherwise
unavoidable.

### Considerations

* Additive by nature: additional declarations don't override the previous ones.
* Adding `!important` results in a "final" declaration that prevents further
  additions, but doesn't remove the selectors previously added to the list.

## `!default` rule

Similar to the `!default` feature in Sass. Applies the property if there isn't
an existing rule, or if it's set to `initial`. Mutually exclusive with
`!important`.

### Example

Suppose we have a rule that only works with elements that don't have their
`display` property set to `inline`:

```css
.wide {
  width: 100%;
  height: 2em;
}
```

To apply this to an element in HTML, we need to know its `display` property
in advance. Blindly adding a declaration like `display: block` would break
elements with a different display property, like `display: flex`:

```html
<!-- BAD: we don't know its initial display property -->
<some-element class="wide block">
  <some-child>...</some-child>
  <some-child>...</some-child>
</some-element>

<style>
  .block {display: block}
</style>
```

With the proposal, we can safely specify the default without breaking elements
with their own rules:

```css
.wide {
  width: 100%;
  height: 2em;
  display: block !default;
}
```

```html
<!-- GOOD: the default display property, if any, is preserved -->
<some-element class="wide">
  <some-child>...</some-child>
  <some-child>...</some-child>
</some-element>
```
