# Structure of jsonform.js

The entire jsonform.js file is one big method call.

It starts:

    (function(serverside, global, $, _, JSON) {

And ends:

    })((typeof exports !== 'undefined'),
      ((typeof exports !== 'undefined') ? exports : window),
      ((typeof jQuery !== 'undefined') ? jQuery : { fn: {} }),
      ((typeof _ !== 'undefined') ? _ : null),
      JSON);

# The main entrypoint

At approximately line 3780:

    $.fn.jsonForm = function(options) {
        ....
    }


# Main HTML boilerplate for standard inputs

This is the main location in the script that generates HTML divs and inputs and injects class names, ids, etc.

Located at approximately line 211:

    // Twitter bootstrap-friendly HTML boilerplate for standard inputs
    jsonform.fieldTemplate = function(inner) {
      return '<div ' +
        '<% for(var key in elt.htmlMetaData) {%>' +
          '<%= key %>="<%= elt.htmlMetaData[key] %>" ' +
        '<% }%>' +



# Schemas are broken into nodes

A new node is initialized with

    /**
     * Represents a node in the form.
     *
     * Nodes that have an ID are linked to the corresponding DOM element
     * when rendered
     *
     * Note the form element and the schema elements that gave birth to the
     * node may be shared among multiple nodes (in the case of arrays).
     *
     * @class
     */
    var formNode = function () {
        ...
    }

This method can be found around line 1923.


# Loading a value into a form field for DISPLAY only

The method, `getInitialValue`, which is around line 1820, computes the value for a form element.

The source heirarchy is:

     * The "initial" value is defined as:
     * 1. the previously submitted value if already submitted
     * 2. the default value defined in the layout of the form
     * 3. the default value defined in the schema


# Building a lookup key from the Schema

This is done in `getSchemaKey` approximately around line 1704.

The present key format is explained below:

    /**
     * Retrieves the key definition from the given schema.
     *
     * The key is identified by the path that leads to the key in the
     * structured object that the schema would generate. Each level is
     * separated by a '.'. Array levels are marked with []. For instance:
     *  foo.bar[].baz
     * ... to retrieve the definition of the key at the following location
     * in the JSON schema (using a dotted path notation):
     *  foo.properties.bar.items.properties.baz
     *
     * @function
     * @param {Object} schema The JSON schema to retrieve the key from
     * @param {String} key The path to the key, each level being separated
     *  by a dot and array items being flagged with [].
     * @return {Object} The key definition in the schema, null if not found.
     */

# Getting and setting a DATA value for a given `KEY PATH`

See: `jsonform.util.setObjKey`, line 1625

And, see: `jsonform.util.getObjKey`, line 1568

# Emitting HTML for the set of supported element types

The definitions for this are in `jsonform.elementTypes`.

This MASSIVE data structure stretches from line 275 to 1547.

# Underscore being used for templating

Code in the JSON, such as:

    var fieldTemplateSettings = {
        evaluate    : /<%([\s\S]+?)%>/g,
        interpolate : /<%=([\s\S]+?)%>/g
      };

is setting up regular expressions to help the Underscore library process the templates used by JSONForm.

## Here is a detailed explanation of the RegEx logic from ChatGPT:

This JavaScript code defines an object called `fieldTemplateSettings` with two properties: `evaluate` and `interpolate`. Both of these properties have regular expression values. This pattern suggests that the object is intended for settings of a template system, specifically an Underscore.js-like template.

Here's a breakdown:

1. **`fieldTemplateSettings`**: This is the name of the variable holding the object.

2. **`evaluate`**: The regular expression `/<%([\s\S]+?)%>/g` is meant to match text that's enclosed between `<%` and `%>`. This kind of syntax is often used in template engines to denote blocks of code to be evaluated. For example, in Underscore.js templates:
   ```html
   <% if (user) { %>
     Hello, <%= user.name %>!
   <% } %>
   ```
   Here, everything between `<%` and `%>` is a code block that gets executed, but not displayed in the rendered template.

3. **`interpolate`**: The regular expression `/<%=([\s\S]+?)%>/g` is meant to match text that's enclosed between `<%=` and `%>`. This syntax is used in template engines to denote expressions whose results should be displayed in the rendered template. In the example above, `<%= user.name %>` would be replaced with the name of the `user`, assuming `user` is an object passed to the template.

The `[\s\S]+?` in both regular expressions is used to match any character (including newlines) lazily (non-greedy match), ensuring that the shortest possible match is made. The `g` at the end of the regular expressions indicates a global match, meaning it will match all occurrences in a given string, not just the first one.

In context, such settings are typically used with a library (like Underscore.js) that can parse templates and replace these tokens with actual values or execute the embedded JavaScript code.

# Loading the `JSONForm` object into the `WINDOW`

The object is first defined around line 67.

    /**
       * The jsonform object whose methods will be exposed to the window object
       */
      var jsonform = {util:{}};

# Helper methods and helper data are loaded from the opening lines to line 116

1. General helper methods and helper data are loaded from the opening lines to line 116
2. Tab initialization methods and logic go from line 118 to line 207.

# Confusing array logic

1. `truncateToArrayDepth`, line 1733
2. `applyArrayPath`, line 1770


# NODE Logic - Tree Represenation

`formNode` is the class type around which trees are built.

Below are the methods and lines of note:

- `var formNode = function () {`, line 1923
- `formNode.prototype.clone = function (parentNode) { `, line 2025
- `formNode.prototype.hasNonDefaultValue = function () {`, line 2047
- `formNode.prototype.appendChild = function (node) {`, line 2073
- `formNode.prototype.removeChild = function () {`, line 2086
- `formNode.prototype.moveValuesTo = function (node) {`, line 2117 - For `ARRAYS`
- `formNode.prototype.switchValuesWith = function (node) {`, line 2138 - For `ARRAYS`
- `formNode.prototype.resetValues = function () {`, line 2156 - Clear the values from the form. Includes emptying arrays.
- `formNode.prototype.setChildTemplate = function (node) {`, line 2209 - For complex arrays
- `formNode.prototype.render = function (el) {`, line 2687 - Render a node
- `formNode.prototype.getFormValues = function (updateArrayPath) {`, line 2574 to line 2673 - Grab the forms `JSON DATA` for a SPECIFIC node
- `formNode.prototype.setContent = function (html, parentEl) {`, line 2710 - Inject node HTML into the dom
- `formNode.prototype.updateElement = function (domNode) {`, line 2747 - Refresh the dom for a specific node
- `formNode.prototype.generate = function () {`, line 2787 - Generate HTML (**QUESTION:** How does this differ from all the other render related methods? Is this the start method?)
- `$.fn.jsonFormErrors = function(errors, options) {`, line 3710 - Highlight validation errors in the UI

# Calculating initial field values

Around line 2215 to line 2493.

    /**
     * Recursively sets values to all nodes of the current subtree
     * based on previously submitted values, or based on default
     * values when the submitted values are not enough
     *
     * The function should be called once in the lifetime of a node
     * in the tree. It expects its parent's arrayPath to be up to date.
     *
     * Three cases may arise:
     * 1. if the form element is a simple input field, the value is
     * extracted from previously submitted values of from default values
     * defined in the schema.
     * 2. if the form element is an array-like node, the child template
     * is used to create as many children as possible (and at least one).
     * 3. the function simply recurses down the node's subtree otherwise
     * (this happens when the form element is a fieldset-like element).
     *
     * @function
     * @param {Object} values Previously submitted values for the form
     * @param {Boolean} ignoreDefaultValues Ignore default values defined in the
     *  schema when set.
     */
    formNode.prototype.computeInitialValues = function (values, ignoreDefaultValues) { 

# Grab the forms `JSON DATA` for a SPECIFIC node

This is the method that GRABS the form data in a JSON format that should match the ORIGINAL JSON Schema.

This method is defined from line 2574 to line 2673.

    formNode.prototype.getFormValues = function (updateArrayPath) {
        ...
    }


# Inject node (`formNode`) HTML into the dom

Line 2710

    formNode.prototype.setContent = function (html, parentEl) {
        ...
    }


# `formNode` remaining to classify

    formNode.prototype.enhance = function () {


# `FormTree` holds `FormNode`s?

Line 3131

    /**
     * Form tree class.
     *
     * Holds the internal representation of the form.
     * The tree is always in sync with the rendered form, this allows to parse
     * it easily.
     *
     * @class
     */
    var formTree = function () {
      this.eventhandlers = [];
      this.root = null;
      this.formDesc = null;
    };

# Using the `formTree` form layout feature

Line 3257

Also, injects the existing, previously-submitted values that are stored in `this.formDesc.value`

    /**
     * Builds the internal form tree representation from the requested layout.
     *
     * The function is recursive, generating the node children as necessary.
     * The function extracts the values from the previously submitted values
     * (this.formDesc.value) or from default values defined in the schema.
     *
     * @function
     * @param {Object} formElement JSONForm element to render
     * @param {Object} context The parsing context (the array depth in particular)
     * @return {Object} The node that matches the element.
     */
    formTree.prototype.buildFromLayout = function (formElement, context) {
        ...
    }

## Using the provided form `value`

The method `formTree.prototype.buildFromLayout` is at least one of the methods that consumes `value`

### Compute initial values also uses `value`

Line 3516

    /**
     * Computes the values associated with each input field in the tree based
     * on previously submitted values or default values in the JSON schema.
     *
     * For arrays, the function actually creates and inserts additional
     * nodes in the tree based on previously submitted values (also ensuring
     * that the array has at least one item).
     *
     * The function sets the array path on all nodes.
     * It should be called once in the lifetime of a form tree right after
     * the tree structure has been created.
     *
     * @function
     */
    formTree.prototype.computeInitialValues = function () {
      this.root.computeInitialValues(this.formDesc.value);
    };


# Rendering the form tree into the `DOM` ROOT

Line 3528

    /**
     * Renders the form tree
     *
     * @function
     * @param {Node} domRoot The "form" element in the DOM tree that serves as
     *  root for the form
     */
    formTree.prototype.render = function (domRoot) {
        ...
    }

## Compute if the forms schema has AT LEAST ONE required fields

A function to crawl the tree and determine this is at line 3650.

    formTree.prototype.hasRequiredField = function () {
        ...
    }


# Applying a function to an entire form tree with `formTree.forEachElement`

It is a recursive function that applies a function to all nodes in a formTree.

    callback(root.children[i]);

This function is defined at approximately line 3547.

    /**
     * Walks down the element tree with a callback
     *
     * @function
     * @param {Function} callback The callback to call on each element
     */
    formTree.prototype.forEachElement = function (callback) {

# Form validation via `formTree` method `formTree.prototype.validate`

The method that does validation is defined on line 3559

    formTree.prototype.validate = function(noErrorDisplay) {
        ...
    }

## Rendering validation errors `$.fn.jsonFormErrors`

Defined on line 3710.

    $.fn.jsonFormErrors = function(errors, options) {
      $(".error", this).removeClass("error");
      $(".warning", this).removeClass("warning");

      $(".jsonform-errortext", this).hide();

      if (!errors) return;
      ...
  }

# Form submission via `formTree` method `formTree.prototype.submit`

The submission method is defined at line 3597.

    formTree.prototype.submit = function(evt) {
        ...
    }

# Form value extraction at the top-level is triggered off of `jsonForm.getFormValue`

In turn, it calls `form.root.getFormValues`.

This tells us that `form.root` is likely a `formNode` because `getFormValues` is only defined on `formNode`.

The jsonForm.getFormValue function is relatively simple.

It extracts the form element using JQuery, and then recursively processes the underlying data.

Below is the full method:

    /**
     * Returns the structured object that corresponds to the form values entered
     * by the user for the given form.
     *
     * The form must have been previously rendered through a call to jsonform.
     *
     * @function
     * @param {Node} The <form> tag in the DOM
     * @return {Object} The object that follows the data schema and matches the
     *  values entered by the user.
     */
    jsonform.getFormValue = function (formelt) {
      var form = $(formelt).data('jsonform-tree');
      if (!form) return null;
      return form.root.getFormValues();
    };

# `JQuery` exposed functions

## `jsonForm` Create form

This is the main entrypoint for jsonForm defined on line 3780

    $.fn.jsonForm = function(options) {
        ...
    }

## `jsonFormValue` Get the JSON formatted form data entered by the user

    /**
     * Retrieves the structured values object generated from the values
     * entered by the user and the data schema that gave birth to the form.
     *
     * Defined as a jQuery function that typically needs to be applied to
     * a <form> element whose content has previously been generated by a
     * call to "jsonForm".
     *
     * Unless explicitly disabled, the values are automatically validated
     * against the constraints expressed in the schema.
     *
     * @function
     * @return {Object} Structured values object that matches the user inputs
     *  and the data schema.
     */
    $.fn.jsonFormValue = function() {
      return jsonform.getFormValue(this);
    };

## `jsonFormErrors` render validation errors

Defined on line 3710.

    $.fn.jsonFormErrors = function(errors, options) {
        $(".error", this).removeClass("error");
        $(".warning", this).removeClass("warning");

        $(".jsonform-errortext", this).hide();
        if (!errors) return;

        var errorSelectors = [];
        for (var i = 0; i < errors.length; i++) {
            ...
        }
    }
