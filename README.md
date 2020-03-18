# Crate xml_dom

A Rust crate providing a reasonably faithful implementation of the  W3C 
[Document Object Model Core, Level 2](https://www.w3.org/TR/DOM-Level-2-Core).

![MIT License](https://img.shields.io/badge/license-mit-118811.svg)
![Minimum Rust Version](https://img.shields.io/badge/Min%20Rust-1.38-green.svg)
[![crates.io](https://img.shields.io/crates/v/upnp-rs.svg)](https://crates.io/crates/xml_dom)
[![docs.rs](https://docs.rs/xml_dom/badge.svg)](https://docs.rs/xml_dom)
[![GitHub stars](https://img.shields.io/github/stars/johnstonskj/rust-xml_dom.svg)](https://github.com/johnstonskj/rust-xml_dom/stargazers)

This crate provides a trait-based implementation of the DOM with minimal changes to the style
and semantics defined in the Level 2 specification. The specific mapping from the IDL in the
specification is described in the documentation, however from a purely style point of
view the implementation has the following characteristics:

1. It maintains a reasonable separation between the node type traits and the tree implementation
   using opaque an `RefNode` reference type.
1. Where possible the names from IDL are used with minimal conversion, however some redundant
   suffixes (`_data`, `_node`) have been reduced for brevity/clarity.
1. This leads to a replication of the typical programmer experience where casting between the
   node traits is required. This is supported by the `xml_dom::convert` module.

# Example

```rust
use xml_dom::*;
use xml_dom::convert::*;

// Bootstrap; get an instance of `DOMImplementation`. The mechanism for this is
// intentionally undefined by the specification.
let implementation = get_implementation();

// Create a `DocumentType` instance.
let document_type = implementation
    .create_document_type(
        "html",
        Some("-//W3C//DTD XHTML 1.0 Transitional//EN"),
        Some("http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd"),
    )
    .unwrap();

// Create a new `Document` using the document type defined above. Note that this 
// also has the side-effect of creating the document's root element named "html".
let mut document_node = implementation
    .create_document("http://www.w3.org/1999/xhtml", "html", Some(document_type))
    .unwrap();

// Cast the returned document `RefNode` into a `RefDocument` trait reference
let document = as_document_mut(&mut document_node).unwrap();

// Fetch the document's root element as a node, then cast to `RefElement`.
let mut root_node = document.document_element().unwrap();
let root = as_element_mut(&mut root_node).unwrap();

// Create an `Attribute` instance on the root element.
root.set_attribute("lang", "en");

// Create two child `Element`s of "html".
let _head = root.append_child(document.create_element("head").unwrap());
let _body = root.append_child(document.create_element("body").unwrap());

// Display as XML.
let xml = document_node.to_string();
println!("document 2: {}", xml);
```

This should result in the following XML; note that formatting was added for this document, the provided 
implementation of `Display` for `RefNode` does not format the output.
 
```xml
<!DOCTYPE 
  html 
  PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" 
  SYSTEM "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html lang="en">
  <head></head>
  <body></body>
</html>
```

## Changes

**Version 0.1.3** (_in progress_)

* More unit tests overall, especially for append/insert/replace child
* Add support for xml declaration, not reusing processing instruction
* Support the last Level 2 _extended interfaces_.
* Implement an options capability to turn on processing behaviors.

**Version 0.1.2**

* Focus on feature completion:
  * implement all core trait features
  * implement extended trait features for currently supported traits
  * unescaping text
  * refactor `NodeImpl` for extended traits
* Unit tests, lot's of unit tests

**Version 0.1.1**

* Focus on API, separate the traits from implementation more cleanly.
* Better `Display` formatting
* Better `append_child` rule support
* Have support for namespace resolution
* Have support for text escaping on setting values
* More examples, fleshing out more of the common methods.
* Note, this is NOT YET ready for production usage.

**Version 0.1.0**

* Focus on modeling as traits, not all methods actually implemented.
* Note, this is NOT YET ready for production usage.

## TODO

1. Currently does not support `DocumentFragment`, `Entity`, `EntityReference`, or `Notation`.
1. Not intending to be schema-aware, so `Document::get_element_by_id` always returns `None`.
1. A lot of required methods are still `unimplemented!()`.
1. Intend to add `reader` feature to de-serialize using crate `quick_xml`.
