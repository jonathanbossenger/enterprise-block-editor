# Anatomy of a Block

Before diving into the process of building custom blocks, it's important to understand the structure and components of a block. In this lesson, we'll explore the key parts of a block, how to register a block and the different components of a block, and how to use block attributes and block supports to enhance the block's functionality.

## Block Registration

Block registration involves defining the block's metadata, settings, and behavior and requires two parts:

- Registering the block’s code with WordPress using the relevant PHP function(s)
- Registering the block in the editor with the JavaScript `registerBlockType` function.

## PHP block registration

There are a few ways available to register blocks, which has changed in recent versions of WordPress.

### register_block_type

The original way recommended to register blocks was to use the `register_block_type` function, and pass it the path to the block's directory as an argument. 

```php
add_action( 'init', 'create_block_my_custom_block_block_init' );
function create_block_my_custom_block_block_init() {
	register_block_type( __DIR__ . '/build/my-custom-block' );
}
```

If you are using a version of WordPress older than 6.7, you will need to use this method.

This would parse the block's `block.json` file to register the block metadata and then register the block itself.

The `block.json` file contains all the necessary information about the block, including its name, title, description, attributes, and the paths to the scripts and styles that make up its functionality and styling.

```
{
	"$schema": "https://schemas.wp.org/trunk/block.json",
	"apiVersion": 3,
	"name": "create-block/my-custom-block",
	"version": "0.1.0",
	"title": "My Custom Block",
	"category": "widgets",
	"icon": "smiley",
	"description": "Example block scaffolded with Create Block tool.",
	"example": {},
	"supports": {
		"html": false
	},
	"textdomain": "my-custom-block",
	"editorScript": "file:./index.js",
	"editorStyle": "file:./index.css",
	"style": "file:./style-index.css",
	"viewScript": "file:./view.js"
}
```

Let's break down the key properties in this file:

* `$schema`: The URL of the schema that defines the structure of the `block.json` file.
* `apiVersion`: The version of the block API that the block uses.
* `name`: A unique identifier for the block, typically prefixed with your plugin or theme name.
* `version`: The version number of the block.
* `title`: The human-readable name of the block, displayed in the block inserter.
* `category`: The category in which the block will appear in the block inserter (e.g., "common", "formatting", "layout", "widgets", "embed").
* `icon`: The icon to display for the block in the block inserter. This can be a Dashicon or a custom SVG.
* `description`: A brief description of the block's purpose.
* `supports`: An object that defines the features the block supports, such as HTML editing or alignment options.
* `textdomain`: The text domain for the block, used for localization.
* `editorScript`: The path to the JavaScript file that contains the block's editor functionality.
* `editorStyle`: The path to the CSS file that contains the block's editor styles.
* `style`: The path to the CSS file that contains the block's styles.
* `viewScript`: The path to a JavaScript file that can contain additional front-end functionality.

You’ll notice that the `editorScript` is pointing to the `index.js` file in the build directory. This file generated during the build process contains the minified and optimized version of the code in the `src` directory.

### register_block_type_from_metadata and the blocks-manifest.php file

In WordPress 6.7 the `register_block_type_from_metadata` was added, to improve the performance of block type registration, which uses a new `blocks-manifest.php` file. The `blocks-manifest.php` file is generated during the build process and contains all the metadata from the `block.json` files for all blocks in the plugin, combined in a single PHP array.

```php
<?php
// This file is generated. Do not modify it manually.
return array(
	'my-custom-block' => array(
		'$schema' => 'https://schemas.wp.org/trunk/block.json',
		'apiVersion' => 3,
		'name' => 'create-block/my-custom-block',
		'version' => '0.1.0',
		'title' => 'My Custom Block',
		'category' => 'widgets',
		'icon' => 'smiley',
		'description' => 'Example block scaffolded with Create Block tool.',
		'example' => array(
			
		),
		'supports' => array(
			'html' => false
		),
		'textdomain' => 'my-custom-block',
		'editorScript' => 'file:./index.js',
		'editorStyle' => 'file:./index.css',
		'style' => 'file:./style-index.css',
		'viewScript' => 'file:./view.js'
	)
);
```

The changes to the block registration process were part of a larger effort in WordPress 6.7 to improve the performance and usability of block development in WordPress. 

In the previous method, the metadata was loaded into memory by parsing the `block.json` file, which was inefficient, especially for large projects with many blocks. Using `register_block_type_from_metadata` which simply includes the `blocks-manifest.php` PHP file speeds up the process of loading the block metadata into memory.

`register_block_type_from_metadata` accepts two arguments, the path to the plugins build directory and the path to the `blocks-manifest.php` file, but it is used to register a single block type. Therefore, if you have multiple blocks, you need to loop through the array of block types in the `blocks-manifest.php` file and register each block type using `register_block_type`.

```php
add_action( 'init', 'create_block_my_custom_block_block_init' );

function create_block_my_custom_block_block_init() {
	wp_register_block_metadata_collection( __DIR__ . '/build', __DIR__ . '/build/blocks-manifest.php' );
	$manifest_data = require __DIR__ . '/build/blocks-manifest.php';
	foreach ( array_keys( $manifest_data ) as $block_type ) {
		register_block_type( __DIR__ . "/build/{$block_type}" );
	}
}
````

If the WordPress version of the site is 6.7, you should use this method.

### wp_register_block_metadata_collection

In WordPress 6.8, the `wp_register_block_metadata_collection` function was introduced to simplify the block registration process. This function allows you to register all blocks in a plugin or theme with a single function call.

`register_block_type_from_metadata` accepts the same two arguments as `register_block_type_from_metadata` , the path to the plugins's build directory and the path to the `blocks-manifest.php` file. However, you don't need the loop to register each block type or the additional calls to `register_block_type`.

```php
add_action( 'init', 'create_block_my_custom_block_block_init' );

function create_block_my_custom_block_block_init() {
	wp_register_block_types_from_metadata_collection( __DIR__ . '/build', __DIR__ . '/build/blocks-manifest.php' );
}
```

If the WordPress version of the site is 6.8 or newer, you only need to use this function to register your blocks. 

### create-block and backward compatibility

In order to support older versions of WordPress, the `create-block` package was updated to scaffold blocks using both methods, with WordPress version checks in place to only use specific functionality in the right context.

## JavaScript block registration

Once the blocks metadata and source code is registered with WordPress, the next step is to register the block in the editor using JavaScript. This is done using the `registerBlockType` function from the `@wordpress/blocks` package.

`registerBlockType` is typically called in the main script for your block, usually `index.js`. At a minimum, `registerBlockType` needs two things:

* The block name
* A JavaScript object containing properties for the edit and save functionality of the block

Below is an example of the `index.js` file generated by `create-block`:

```javascript
/**
 * Registers a new block provided a unique name and an object defining its behavior.
 *
 * @see https://developer.wordpress.org/block-editor/reference-guides/block-api/block-registration/
 */
import { registerBlockType } from '@wordpress/blocks';

/**
 * Lets webpack process CSS, SASS or SCSS files referenced in JavaScript files.
 * All files containing `style` keyword are bundled together. The code used
 * gets applied both to the front of your site and to the editor.
 *
 * @see https://www.npmjs.com/package/@wordpress/scripts#using-css
 */
import './style.scss';

/**
 * Internal dependencies
 */
import Edit from './edit';
import save from './save';
import metadata from './block.json';

/**
 * Every block starts by registering a new block type definition.
 *
 * @see https://developer.wordpress.org/block-editor/reference-guides/block-api/block-registration/
 */
registerBlockType( metadata.name, {
	/**
	 * @see ./edit.js
	 */
	edit: Edit,

	/**
	 * @see ./save.js
	 */
	save,
} );
```

You will see that this code imports the `style.scss`, the `Edit` component, `save` function and the metadata in `block.json` from their respective files in order to apply the styles and functionality to the block being registered.

This is a feature of modern JavaScript called [module bundling](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules). It allows you to separate logical functions and components into separate files. You can then import the code from those files to use it. It also allows you to import code from external libraries, in this case the `@wordpress/blocks` package.

#### The `edit` Component

The `edit` component defines the interface that users interact with when editing the block in the editor.

This component is a JavaScript function that returns a React element, describing how the block should appear and behave.

Here's an example of an `edit` component generated by create-block:

```javascript
/**
 * Retrieves the translation of text.
 *
 * @see https://developer.wordpress.org/block-editor/reference-guides/packages/packages-i18n/
 */
import { __ } from '@wordpress/i18n';

/**
 * React hook that is used to mark the block wrapper element.
 * It provides all the necessary props like the class name.
 *
 * @see https://developer.wordpress.org/block-editor/reference-guides/packages/packages-block-editor/#useblockprops
 */
import { useBlockProps } from '@wordpress/block-editor';

/**
 * Lets webpack process CSS, SASS or SCSS files referenced in JavaScript files.
 * Those files can contain any CSS code that gets applied to the editor.
 *
 * @see https://www.npmjs.com/package/@wordpress/scripts#using-css
 */
import './editor.scss';

/**
 * The edit function describes the structure of your block in the context of the
 * editor. This represents what the editor will render when the block is used.
 *
 * @see https://developer.wordpress.org/block-editor/reference-guides/block-api/block-edit-save/#edit
 *
 * @return {Element} Element to render.
 */
export default function Edit() {
	return (
		<p { ...useBlockProps() }>
			{ __(
				'My Custom Block – hello from the editor!',
				'my-custom-block'
			) }
		</p>
	);
}
```

### The `save` Function

The `save` function is responsible for defining the block's output on the site's front end. This function is also a JavaScript function that returns a React element, describing how the block should be rendered.

Here's an example of a `save` function generated by create-block:

```javascript
/**
 * React hook that is used to mark the block wrapper element.
 * It provides all the necessary props like the class name.
 *
 * @see https://developer.wordpress.org/block-editor/reference-guides/packages/packages-block-editor/#useblockprops
 */
import { useBlockProps } from '@wordpress/block-editor';

/**
 * The save function defines the way in which the different attributes should
 * be combined into the final markup, which is then serialized by the block
 * editor into `post_content`.
 *
 * @see https://developer.wordpress.org/block-editor/reference-guides/block-api/block-edit-save/#save
 *
 * @return {Element} Element to render.
 */
export default function save() {
	return (
		<p { ...useBlockProps.save() }>
			{ 'My Custom Block – hello from the saved content!' }
		</p>
	);
}
```

### useBlockProps

You will notice that both the `Edit` component and `save` function use `useBlockProps`. `useBlockProps` is a React hook that allows developers to manage and apply properties to the root element of a block, ensuring consistency between the editor and the front end.

Developers can use `useBlockProps` to add custom attributes (e.g., `className`, inline styles) to the root element of a block. It also consolidates attributes like alignment classes and other editor-specific properties into the block’s wrapper element, making it easier to manage.

### Block Rendering Methods

There are two primary methods to render block content on the site's front end: [static rendering and dynamic rendering](https://developer.wordpress.org/block-editor/getting-started/fundamentals/static-dynamic-rendering/).

#### Static Rendering

Static rendering is the default approach for most blocks in the WordPress Block Editor and is the method used when scaffolding a block with create-block.

With static rendering, the block output (also known as block markup) is generated when the block's `save` function is triggered. This markup is stored directly in the database.

When the post or page that contains the block loads on the front end, the pre-saved markup is retrieved and displayed to the user.

Examples of WordPress core blocks that use static rendering include Paragraph, Heading, and Preformatted blocks.

#### Dynamic Rendering

Dynamic blocks generate their HTML output on the fly when a block is loaded on the front end. Instead of storing the complete HTML in the database, dynamic blocks store a representation of the block (including its attributes) and use server-side PHP code to render the final output when needed.

Examples of WordPress core blocks that use dynamic rendering include Latest Posts, Calendar, and Archives blocks.

You will learn how to create dynamic blocks in Module 5\.

## Block Attributes

Attributes define the data that a block stores and manages. Each attribute has a name, a type, and, optionally, a default value. Attributes can be defined in the `attributes` property of the `block.json` file.

Here are some common attribute types:

* `string`: A text string.
* `number`: A number.
* `boolean`: A true/false value.
* `array`: An array of values.
* `object`: An object containing key-value pairs.

Here's an example of how to define attributes for a block:

```
attributes: {
  content: {
    type: 'string',
    default: 'Hello World!',
  },
  alignment: {
    type: 'string',
    default: 'left',
  },
  imageUrl: {
    type: 'string',
  },
},
```

In this example, we're defining three attributes: `content`, `alignment`, and `imageUrl`. The `content` attribute is a string with a default value of "Hello World\!", the `alignment` attribute is a string with a default value of "left", and the `imageUrl` attribute is a string with no default value.

### Accessing the block’s attributes

Attributes are passed to a block’s Edit component or save function via a `props` object. Both the Edit component and the save function are set up to always accept this props object containing all the properties of the block.  
Developers can then access the content of the attribute by accessing the `attributes` property on the `props` object.

```javascript
export default function Edit( props ) {
	const blockContent = props.attributes.content;
	return (
		<p { ...useBlockProps() }>
			{ __(
				blockContent,
				'my-custom-block'
			) }
		</p>
	);
}
```

Modern JavaScript also supports [object destructuring](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment), which allows you to extract specific properties from an object and assign them to variables. This can make your code more concise and easier to read.

To start, you can destructure the `attributes` property from the `props` object like this:

```javascript
export default function Edit( { attributes } ) {
    const content = attributes.content;
    return (
        <p { ...useBlockProps() }>
            { __(
                content,
                'my-custom-block'
            ) }
        </p>
    );
}
```

You can also destructure the `content` property directly from the `attributes` object like this:

```javascript
export default function Edit( { attributes } ) {
    const { content } = attributes;
    return (
        <p { ...useBlockProps() }>
            { __(
                content,
                'my-custom-block'
            ) }
        </p>
    );
}
```

## Block Supports

The `supports` property allows you to define the common features that a block supports. This can include things like HTML editing, alignment options, custom styles, and more.

Supports are defined as an object containing key-value pairs, where the key is the name of the feature and the value is a boolean or an object containing additional settings.

Here are some common block supports:

* `html`: Whether the block supports HTML editing.
* `align`: Whether the block supports alignment options (e.g., "left", "center", "right", "wide", "full").
* `className`: Whether the block should have a CSS class name automatically generated.
* `customClassName`: Whether the user can add a custom CSS class name to the block.
* `multiple`: Whether multiple instances of the block can be inserted into a post.

Here's an example of how to define block supports in `block.json`:

```
supports: {
  html: false,
  align: true
},
```

In this example, we're disabling HTML editing and enabling alignment options. This will add the alignment toolbar to the block in the editor, allowing users to choose from different alignment options.

Now that you have a better understanding of the anatomy of a block, you're ready to start building your own custom blocks. In the next lesson, we'll explore the block development workflow and how to set up your local environment for block development.  

