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

