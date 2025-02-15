# Dataset HTML attributes

You can store non-html information in the element `data-*` attributes and access them via `dataset` property.

## Convert dataset to a plain object

📺 Watch this recipe explained at [Confirm All HTML data- Attributes At Once By Using The dataset Property](https://youtu.be/t8BSY2czges).

<!-- fiddle Using data attributes -->

```html
<article
  id="electric-cars"
  data-columns="3"
  data-index-number="12314"
  data-parent="cars"
>
  All about electric cards
</article>
```

All `data-*` properties are collected in a single object. Property names are camel-cased, all values are strings. We can confirm a single property

```js
cy.get('article#electric-cars')
  .should('have.prop', 'dataset')
  .its('indexNumber')
  .should('equal', '12314')
// equivalent to checking the "data-*" attribute
cy.get('article#electric-cars').should(
  'have.attr',
  'data-index-number',
  '12314',
)
```

Let's try confirming all `data-*` attributes. Directly comparing property with an object does not work

```js skip
// 🚨 INCORRECT
// cannot compare dataset object with a plain object
cy.get('article#electric-cars').should('have.prop', 'dataset', {
  columns: '3',
  indexNumber: '12314',
  parent: 'cars',
})
```

Instead, yield the `dataset` value and convert into a plain object first before using `deep.equal` assertion.

```js
cy.get('article#electric-cars')
  // yields "DOMStringMap" object
  .should('have.prop', 'dataset')
  // which we can convert into an object
  .then(JSON.stringify)
  .then(JSON.parse)
  // and compare as plain object
  .should('deep.equal', {
    columns: '3',
    indexNumber: '12314',
    parent: 'cars',
  })
```

**Tip:** you can use [cy-spok](https://github.com/bahmutov/cy-spok) to confirm object properties.

<!-- fiddle-end -->

## Query commands

Converting a browser object like `DOMStringMap` to a plain object using `JSON.parse(JSON.stringify(x))` is pretty common. If we use `cy.then(JSON.stringify).then(JSON.parse)` commands we are breaking retries because `cy.then` is not a query command. I suggest using my plugin [cypress-map](https://github.com/bahmutov/cypress-map) and its query commands to convert `DOMStringMap` to a plain JavaScript object.

<!-- fiddle Convert dataset with retries -->

In the example below, one of the data attributes is set after a delay.

```html
<article
  id="electric-cars"
  data-columns="3"
  data-index-number="loading..."
  data-parent="cars"
>
  All about electric cards
</article>
<script>
  // force the tests to retry by adding "data-index-number" attribute
  // after some delay
  setTimeout(() => {
    document
      .getElementById('electric-cars')
      .setAttribute('data-index-number', '12314')
  }, 1500)
</script>
```

We can keep querying the DOM and converting to a plain object until the assertion passes. Query commands `cy.toPlainObject`, `cy.map`, and `cy.print` are from [cypress-map](https://github.com/bahmutov/cypress-map) plugin.

```js
cy.get('article')
  .should('have.prop', 'dataset')
  // "cy.toPlainObject" comes from cypress-map
  .toPlainObject()
  // convert the property "columns" to a number
  // leaving the rest of the properties unchanged
  .map({
    columns: Number,
  })
  .print('dataset %o')
  .should('deep.equal', {
    columns: 3,
    indexNumber: '12314',
    parent: 'cars',
  })
```

We can even map some property values to convert them before the assertion. For example, we can convert the number of columns from the string "3" to the number 3. Again, `cy.map` from `cypress-map` is our friend here:

<!-- fiddle-end -->

## See also

- [Using data attributes](https://developer.mozilla.org/en-US/docs/Learn/HTML/Howto/Use_data_attributes)
- [data-\*](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/data-*)
