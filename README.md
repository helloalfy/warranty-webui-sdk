<p align="center">
  <img src="https://assets.helloalfy.com/logo.svg" height="90px"/>

  <h1 align="center">Alfy Warranty WebUI SDK</h1>
</p>
<br />
<h2>Installation</h2>
<p>Add the scripts to your page</p>

```html
<script src="https://cdn.jsdelivr.net/gh/helloalfy/warranty-webui-sdk@latest/dist/alfy-webui-sdk.min.js" charset="utf-8"></script>

// Production
<script>Alfy.config({ storeId: '<YOUR_ALFY_STORE_ID>' })</script>

// Demo
<script>Alfy.config({ storeId: '<YOUR_ALFY_STORE_ID>', env: 'development' })</script>
```

## Setting things up
<p>
	To display offers on your webpage you will need: 
	<dl>
		<dt>Your Alfy Store ID</dt>
		<dd>
			This will be provided by your Alfy account manager
		</dd>
		<dt>A product ID </dt>
		<dd>This is the unique product identifier that you had specified in your product catalog upload.  We will use this identifier to know which warranty offer to display.  Generally this is the product ID or similar, however the choice of <b>product ID</b> is up to a merchant to decide.  If you are having trouble finding out what your <b>product productId</b> is, please contact your Alfy account manager or contact us through <a href="https://www.helloalfy.com/#comp-k88mfgn5">Alfy Portal</a>.<br>
		:warning: Your product catalog must be synchronized with Alfy regularly for this to work properly.  
		</dd>
</dl>
</p>

For more options that can be passed to the `Alfy.config()` function, see [API Reference](#api-reference)

## Displaying a Product Offer
Place an html element where you would like to see the product page offers with a css selector that you can reference

```html
<div id="alfy-offer"></div>
```
This element will be used as the container for the displayed warranty offer.  If you have specific width or spacing requirements, you can add them directly to this element using css or inline styles.

### Initialization

#### Displaying multiple buttons

```js
Alfy.buttons.render('#alfy-offer', {
  productId: '<PRODUCT_ID>',
})
```

>Note: to query DOM elements, we use the native <b>document.querySelector</b>.  For this reason we recommend using element ids instead of classes for selector references.

#### Displaying a single button 

When clicked, this button displays a modal, allowing the user to select a plan
```js
Alfy.buttons.renderSimple('#alfy-offer', {
  productId: '<PRODUCT_ID>',
})
```

### Accessing the component instance
It's possible to have multiple offers displayed on the same page and each might have their own product productId so each rendered component acts independently of each other. The component instance will be tied to the element/selector passed in during the call to  `Alfy.buttons.render` and will be used for API calls later in this guide.

**To retrieve the component instance:**

```js
const component = Alfy.buttons.instance('#alfy-offer')
```

### Handling product selection changes
If your store has multiple product variants for a single product page (for example, if you have a product that allows a customer to select a color or size option)  you'll need to pass the new **product reference id** to the SDK when the change occurs.  This prevents a customer from accidentally purchasing the wrong warranty for an incorrect product.  

**Example:**

```js
// calls are done through the component instance
const component = Alfy.buttons.instance('#alfy-offer')

component.setActiveProduct('<ANOTHER_PRODUCT_ID>')
```

## Adding a warranty to the cart
Typically you would add the a warranty selection to the cart when they click on "Add to Cart".  You can retrieve the users warranty selection by calling `component.getPlanSelection();`.  

Here is an example of how this might look in a basic store.

```js
$('#add-to-cart').on('click', function (e) {
  e.preventDefault()

  /** get the component instance rendered previously */
  const component = Alfy.buttons.instance('#alfy-offer');

  /** get the users plan selection */
  const plan = component.getPlanSelection();

  if (plan) {
    /** 
     * If a plan is returned from #getPlanSelection, then a user has selected a
     * warranty option.
     * 
     * See the API reference for details on what is included with the "plan"
     * object.
     * 
     * If you are using an ecommerce addon (e.g. Shopify) this is where you
     * would use the respective add-to-cart helper function.
     * 
     * For custom integrations, use the plan data to determine which warranty
     * id to add to the cart and at what price.
     */
     // add plan to cart, then handle form submission
  } else {
     // handle form submission
  }
})
```

## Displaying a Modal Offer
A modal offer can be triggered from anywhere on a page and provides the user with another opportunity to protect their product with a warranty.
<p align="center"><img src="https://storage.googleapis.com/helloalfy-static-assets/modal.webp" /></p>

### Opening the modal offer

```js
Alfy.modal.open({
  productId: '<PRODUCT_ID>',
  /**
   * This callback will be triggered both when a user declines the offer and if
   * they choose a plan.  If a plan is chosen, it will be passed into the
   * callback function, otherwise it will be undefined.
   */
   onClose: function (plan) {
     if (plan) {
        // a user has selected a plan.  Add it to their cart. 
     }
   }
})
```

<br />
<h2>Piecing it all together</h2>
<p>
  Combining all of these tools you should now be able to create a complete end-to-end warranty offer solution on your online store.
</p>
<b>Full Example</b>

```html
<form id="product-form" action="/cart/add">
  <div id="alfy-offer"></div>
  <button id="add-to-cart" type="submit">Add To Cart</button>
</form>
```

```js
/** configure */
Alfy.config({ storeId: '<ALFY_STORE_ID>' });

/** initialize offer */
Alfy.buttons.render('#alfy-offer', {
  productId: '<PRODUCT_ID>'
});

/** a fake implementation of add to cart for this example. */
function addPlanToCart(plan, callback) {}

/** bind to add-to-cart click */
$('#add-to-cart').on('click', function (event) {
  event.preventDefault()

  /** get the component instance rendered previously */
  const component = Alfy.buttons.instance('#alfy-offer');

  /** get the users plan selection */
  const plan = component.getPlanSelection();

  if (plan) {
    /** 
     * Add the warranty to the cart using an AJAX call and then submit the form.
     * Replace this section with your own store specific cart functionality.
     */
    addPlanToCart(plan, function () {
      $('#product-form').submit()
    })
  } else {
    Alfy.modal.open({
      productId: '<PRODUCT_ID>',
      onClose: function (plan) {
        if (plan) {
          addPlanToCart(plan, function () {
            $('#product-form').submit()
          })
        } else {
          $('#product-form').submit();
        }
      }
    });
  }
});
```

## API Reference

#### Alfy.config(config)

```js
Alfy.config({
  /** 
   * Alfy store ID
   * @required 
   */
  storeId: string,
  /**
   * a list of product IDs for the current page.  This will preload
   * the offers for each product to improve performance when switching 
   * productIds
   * @optional
   */
  productIds: string[],
  /**
   * optional theme configuration.  A theme passed in as an argument will
   * override any theme potentially retrieved from the Alfy Merchant configuration.
   * These parameters can be also overridden through widgets dedicated rendering
   * methods options. See the related methods reference for more information.
   * @optional
   */
  theme: {
    primaryColor: string,
    buttons: {
      // ...
    },
    button: {
      // ...
    }
  }
})
```

#### Alfy.getConfiguration(storeId?: string)

```js
Alfy.getConfiguration(
  /** @optional */
  storeId
).then((config) => {
  // ...
})
```

#### Alfy.getOffer(productId)

```js
Alfy.getOffer(
  /** @required */
  productId
).then((offer) => {
  if(offer && offer.plans && offer.plans.length) {
    // ...
  }
})
```

#### Alfy.buttons.render(selector, options)

```js
Alfy.buttons.render('#offer-container', {
  /** @required */
  productId: string,
  /**
   * Callback on plan selection change
   * @optional
   */
  onSelectionChange: (plan: IWarrantyPlan | null) => {},
  /**
   * optional theme configuration. Passed theme will merge and potentially override
   * Alfy Merchant conf and theme options passed to Alfy.config
   * @optional
   */
  theme?: {
     /**
      * for theme primary color for this widget
      * @optional
      */
    primaryColor?: string,
    buttons?: {
      /**
       * use this to force its default bg color
       * @default transparent
       * @optional
       */
      bgColor?: string,
      /**
       * disable headline text above buttons
       * @default false
       * @optional
       */
      disableCaption?: bool,
      /**
       * set button presentation theme
       * @default false
       * @optional
       */
      theme?: 'inline' | 'default',
    }
  }
})
```

#### Alfy.buttons.renderSimple(selector, options)

```js
Alfy.buttons.renderSimple('#offer-container', {
  /** @required */
  productId: string,
  /**
   * Callback on modal closing
   * @optional
   */
  onClose: (plan: IWarrantyPlan | null) => {},
  /**
   * optional theme configuration. Passed theme will merge and potentially override Alfy Merchant conf
   * and theme options passed to Alfy.config
   * @optional
   */
  theme?: {
     /**
      * for theme primary color for this widget
      * @optional
      */
    primaryColor?: string,
    button?: {
      /**
       * enable button slim mode with much thinner padding
       * @default false
       * @optional
       */
      slim?: bool,
      /**
       * enable full-width button instead of auto-width 
       * @default false
       * @optional
       */
      fullWidth?: bool,
    }
  }
})
```


#### Alfy.buttons.instance(selector): ButtonsComponent

```js
const component = Alfy.buttons.instance('#offer-container')
```

#### ButtonsComponent#getActiveProduct(): IProduct | null

```js
component.getActiveProduct()

// ActiveProduct:
{
  // productId
  id: string, 
  // The name of the product
  name: string
}
```

#### ButtonsComponent#setActiveProduct()

```js
component.setActiveProduct('P456789-robot')
```

#### ButtonsComponent#getPlanSelection(): IWarrantyPlan | null

```js
component.getPlanSelection()

// SelectedPlan:
{
  /** 
   * The unique plan identifier for this plan 
   * @example "P12849-laptop"
   */
  id: string,
  /** 
   * The price of the warranty plan
   * @example 119.99
   */
  price: number,
  /** 
   * The coverage term length in months 
   * @example 12
   */
  term: number
}
```

#### Alfy.modal.open(options)

```js
Alfy.modal.open({
  /** @required */
  productId: string,
  /** @optional */
  theme: {
    primaryColor: string
  },
  /**
   * Callback on modal closing
   * @optional
   */
  onClose: (plan: IWarrantyPlan | null) => {}
})
```
