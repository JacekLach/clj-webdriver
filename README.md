# Selenium-WebDriver Support for Clojure

This is a Clojure wrapper around the Selenium-WebDriver library. Credits to [mikitebeka/webdriver-clj][webdriver-orig] for the initial code for this project and many of the low-level wrappers around the WebDriver API.

## Usage

Use/require the library in your code:

    (use 'clj-webdriver.core)

Start up a browser:

    (def b (start :firefox "https://github.com"))

At the moment, the best documentation is the source code itself. While there are many functions in the core namespace, they're mostly short and straightforward wrappers around WebDriver API's. For the task of finding elements on the page, I've added some utility functions at the end of the core namespace.

Here's an example of logging into Github:

    ;; Start the browser and bind it to `b`
    (def b (start :firefox "https://github.com"))
    
    ;; Click "Login" link
    (-> b
        (find-it {:text "Login"})
        click)
    
    ;; Input username/email into the "Login or Email" field
    (-> b
        (find-it {:class "text", :name "login"}) ; use multiple attributes
        (input-text "username"))
    
    ;; Input password into the "Password" field
    (-> b
        (find-it {:xpath "//input[@id='password']"}) ; even in "easy" mode, you still
        (input-text "password"))                     ; have :xpath and :css options
    
    ;; Click the "Log in" button"
    (-> b
        (<find-it :input {:value "Log"}) ; see special <find-it and <find-it> helpers
        click)                         ; also used optional tag arg, :input

The key functions for finding an element on the page are `find-it` and `find-them`. The `find-it` function returns the first result that matches the criteria, while `find-them` returns a vector of all matches for the given criteria. Both support the same syntax and set of attributes.

To demonstrate how to use arguments in different ways, consider the following example. If I wanted to find `<a href="/contact" id="contact-link" class="menu-item" name="contact">Contact Us</a>` in a page and click on it I could perform any of the following:

    (-> b
        (find-it :a)    ; assuming its the first <a> on the page
        click)
    
    (-> b
        (find-it {:id "contact-link"})    ; :id is unique, so only one is needed
        click)
    
    (-> b
        (find-it {:class "menu-item", :name "contact"})    ; use multiple attributes
        click)
    
    (-> b
        (find-it :a {:class "menu-item", :name "contact"})    ; specify tag
        click)
    
    (-> b
        (find-it :a {:text "Contact Us"})    ; special :text attribute, uses XPath's
        click)                               ; text() function to find the element
    
    (-> b
        (find-it :a {:text #"(?i)contact"})  ; use Java-style regular
        click)                               ; expressions
    
    (-> b
        (find-it {:xpath "//a[@id='contact-link']"})    ; XPath query
        click)
    
    (-> b
        (find-it {:css "a#contact-link"})    ; CSS selector
        click)
    
    (-> b
        (<find-it> {:class "-item"})    ; the :class attribute contains
        click)                          ; the string "-item"
    
    (-> b
        (<find-it {:class "menu-"})    ; the :class attribute starts with
        click)                         ; the string "menu-"

As seen above, the `find-it` function also understands `:xpath` and `:css` attributes, in which case it finds the element on the page described by the XPath or CSS query provided. An `IllegalArgumentException` will be thrown if you attempt to use `:xpath` or `:css` in conjunction with other attributes.

So, to describe the general pattern of interacting with the page:

    (-> browser-instance
        (find-it :optional-tag-name {:attribute "value", :attribute "value"})
        (do-something-with-the-element))

## Running Tests

The namespace `clj-webdriver.test.example-app.core` contains a [Ring][ring-github] app (routing by [Moustache][moustache-github]) that acts as my "control application" for this project's test suite. Instead of running my tests against a remote server on the Internet (prone to change, not always available), I've packaged this small web application to be run locally for the purposes of testing.

Due to some Java server/socket issues, you cannot start both this Ring app and the WebDriver browser instance in the test itself (in this situation, the Ring app starts and WebDriver opens the browser, but then a host of errors follow).

Here's how I run these tests:

* Open a terminal and run `lein repl` or `lein swank` at the root of this project
* Evaluate `(use 'clj-webdriver.test.example-app.core 'ring.adapter.jetty)`
* Evaluate `(defonce my-server (run-jetty #'routes {:port 8080, :join? false}))`
* Open a new terminal tab/window and run `lein test` at the root of this project

The last test in the suite closes the WebDriver browser instance; you can stop the Jetty server by evaluating `(.stop my-server)` or just killing the REPL with `Ctrl+C` or `C-c C-c`.

If anyone can figure out how to solve this issue (i.e. be able to run just `lein test` and start both the Ring app and WebDriver browser instance in one go), I'd be most appreciative. Until then, testing multiple server-based apps in separate "sandboxes" is acceptable to me.

## License

Distributed under the Eclipse Public License, the same as Clojure.

[webdriver-orig]: https://github.com/mikitebeka/webdriver-clj
[ring-github]: https://github.com/mmcgrana/ring
[moustache-github]: https://github.com/cgrand/moustache
