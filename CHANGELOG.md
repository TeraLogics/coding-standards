# **Coding Standards and Practices Changelog**

## 2016-04-01
* Updated **Promise Structure**.
  * Added a section on conditionally executed subordinate promise chains.
* Updated **DAL** Layer and **Controller** Layer
  * Moved responsibility for string-to-type conversion from query/param inputs to **controller** layer.
  * Removed references to that kind of validation and conversion from **DAL** layer.
  * Indicated that **DAL** layer should almost always perform strict type validation.
  * Fixed some more description-then-example ordering.
* Added **Route Parameters and Query Parameters**.
  * Explains motivation and execution of string-to-type conversion in the **controller** layer.
* Updated **SQL Query Formatting and Execution**.
  * Added some more notes about composing queries.

## 2016-03-30
* Inverted the example/description order to place the example below the description.
* Changed **`===` and `!==`** to **Equality Operators `===` and `!==`**.
* Updated **Single-Quoted Strings**.
  * Added back-ticked string information.
* Added **Truthy and Falsy**.
* Added **Casting To Boolean With `!!` and `!`**.
* Updated **Accessing Globals**.
  * Added more information about ES6 Promise overwriting.
* Added **Logging Errors**.
* Added **Promises**.
  * Moved **Promise Structure** into **Promises**.
* Added **The Promise Constructor vs. The Deferred Pattern**.
* Added **The Explicit Construction Anti-Pattern**.
* Added **Starting A Promise Chain**.
* Added **HTTP Verbs**.

## 2016-03-18
* Added **Single-Quoted Strings**.
* Added **Indentation Character**.
* Added **`===` and `!==`**.
* Added **Identifier Naming**.
* Updated **Accessing Globals**.
  * Split content up in to server-side and client-side sections.
* Added **Promise Structure**.
* Added **Data Retrieval**.
* Added **Pagination and Sorting**.
* Added **Validating the `sortby` Parameter**.
* Added **Transactionalized SQL**.
* Added **To Do** section to track future work.

## 2016-03-17
* Added introduction and link to RFC2119.
* Added **Header Comment**.
* Added **`use strict`**.
* Added **Accessing Globals**.
* Added **Using `require` In Node**.
* Added **Using `parseInt`**.
* Added **Constructing and Raising Errors**.
* Updated **Route** Layer.
  * Added more middleware information.
* Updated **API Documentation**.
  * Added links to other wiki pages.
* Updated **Controller** Layer.
  * Simplified and clarified final catch block.
* Added **Empty Result vs. Not Found**.

## 2015-10-29
* Initial commit.
