# Block Variations

## Introduction

In this lesson, we will explore block variations within the WordPress Block Editor. Block variations allow developers to create multiple versions of a block with different preset attributes, inner blocks, and configurations. By the end of this lesson, you will understand how to create block variations, manage reusable configurations, and modify variations dynamically using JavaScript and WordPress hooks.

## Understanding Block Variations

Block variations provide a mechanism for defining different versions of an existing block without creating entirely new blocks. This approach is widely used in core WordPress blocks, such as the Embed block, which has variations for different content providers (e.g., YouTube, Twitter, and Vimeo) based on the `providerNameSlug` attribute.

By defining variations, developers can enhance the usability of blocks by providing preconfigured versions suited for specific use cases.

## Defining Block Variations

A block variation is defined using an object with specific properties that determine its behavior and appearance within the Block Editor. Some of the key properties include:

- **name**: A unique, machine-readable identifier for the variation.
- **title**: The human-readable name of the variation displayed in the editor.
- **description**: A brief explanation of the variation's purpose.
- **icon**: The icon used to represent the variation in the editor.
- **attributes**: An object defining default attributes for the variation.
- **innerBlocks**: An array specifying the initial configuration of nested blocks.
- **scope**: An array indicating where the variation is applicable (e.g., inserter, transforms).
- **isDefault**: A boolean indicating if this variation should be the default.
- **isActive**: A function or array used to determine if the variation is active based on attributes.

For more details, refer to the [Block Variations API](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-variations/) documentation.

## Creating Block Variations

To create a block variation, use the [`registerBlockVariation`](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-variations/#creating-a-block-variation) function from the `@wordpress/blocks` package. The following example registers a "`Spacer 180`" variation for the `'core/spacer'` block:

```javascript
wp.blocks.registerBlockVariation("core/spacer", {
  name: "wpviplearn/spacer",
  title: wp.i18n.__("Spacer 180", "example-block-variation"),
  scope: ["block", "inserter", "transform"],
  attributes: {
    height: "180px",
  },
  isActive: (blockAttributes) =>
    blockAttributes.height && "180px" === blockAttributes.height,
});
```

## Managing Reusable Configurations

Reusable configurations streamline the development process by allowing developers to define and register block variations that can be used across multiple projects. This improves consistency and reduces redundancy.

For example, developers can create reusable functions for registering variations dynamically:

```javascript
const sharedAttributes = {
  align: "wide",
  className: "custom-block-style",
};

const createVariation = (
  blockName,
  variationName,
  title,
  customAttributes = {},
  customInnerBlocks = []
) => {
  wp.blocks.registerBlockVariation(blockName, {
    name: variationName,
    title,
    attributes: {
      ...sharedAttributes,
      ...customAttributes,
    },
    innerBlocks: customInnerBlocks,
  });
};
```

## Modifying Existing Block Variations Dynamically

Dynamic modification of block variations allows developers to adjust variations based on specific conditions or user interactions. This can be done using JavaScript and WordPress hooks.

### Unregistering a Block Variation

To remove a block variation, use [`unregisterBlockVariation`](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-variations/#removing-a-block-variation):

```javascript
function unregisterSocialLinkVariations() {
  const allowedVariations = [
    "wordpress",
    "facebook",
    "x",
    "linkedin",
    "github",
    "gravatar",
  ];

  const allVariations = wp.data
    .select("core/blocks")
    .getBlockVariations("core/social-link");

  allVariations.forEach(function (variation) {
    if (allowedVariations.indexOf(variation.name) === -1) {
      wp.blocks.unregisterBlockVariation("core/social-link", variation.name);
    }
  });
}
```

## Practical Example: Creating a Custom Block Variation for the Quote block

The following example registers a “Random Quote” variation of the core Quote block:

```javascript
const quoteApiEditorVariationSettings = {
  name: "quote-random-editor",
  description:
    'A "core/quote" block variation that displays a random quote from a local JSON file',
  title: "Quote Random",
  scope: ["block", "inserter", "transform"],
  keywords: ["quote"],
  icon: "universal-access",
  attributes: {
    namespace: "quote-random-editor",
  },
  isActive: ["namespace"],
};

registerBlockVariation("core/quote", quoteApiEditorVariationSettings);
```

To modify existing block variations dynamically, you can use [`blocks.registerBlockType`](https://developer.wordpress.org/block-editor/reference-guides/filters/block-filters/#blocks-registerblocktype) filter.

**Note**: Check out the following examples and their live demos of block variations:

- [block-variations](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/tree/trunk/examples/block-variations) ([live demo](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/Automattic/wpvip-learn-enterprise-block-editor/refs/heads/trunk/examples/block-variations/_playground/blueprint.json))
- [block-variations-quote](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/tree/trunk/examples/block-variations-quote) ([live demo](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/Automattic/wpvip-learn-enterprise-block-editor/refs/heads/trunk/examples/block-variations-quote/_playground/blueprint.json))
- [unregister-block-variations](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/tree/trunk/examples/unregister-block-variations) ([live demo](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/Automattic/wpvip-learn-enterprise-block-editor/refs/heads/trunk/examples/unregister-block-variations/_playground/blueprint.json))
- [modify-block-variation](https://github.com/Automattic/wpvip-learn-enterprise-block-editor/tree/trunk/examples/modify-block-variation) ([live demo](https://playground.wordpress.net/?blueprint-url=https://raw.githubusercontent.com/Automattic/wpvip-learn-enterprise-block-editor/refs/heads/trunk/examples/modify-block-variation/_playground/blueprint.json))

## Conclusion

Block variations are a powerful way to extend and customize blocks in the WordPress Block Editor. By understanding the [`registerBlockVariation`](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-variations/#registerblockvariation) API, managing reusable configurations, and dynamically modifying variations, developers can create more efficient, reusable, and flexible block configurations. Leveraging these techniques will enhance the editing experience and improve the maintainability of WordPress projects.

For further details, refer to the [WordPress Developer Documentation on Block Variations](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-variations/).  