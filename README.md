# Assigning a mixin will bleed out, only when using native CSS properties

https://github.com/Polymer/polymer/issues/4254

We are experiencing an issue where setting a mixin within a selector seems to be misbehaving: the value seems to be "bleeding out" to other elements that do not match the CSS selector.

Even more baffling, this only seems to happen by using the custom CSS properties.

We recreated a simple tiny repository showing the issue: https://github.com/tegola/polymer-apply-bug-testcase (sorry, it was just way too painful getting this to happen on a CDN).

The short story: `<paper-input>` is configurable by changing the mixin `--paper-input-container-input`. So, in `my-app.html`, we do this:

      /* Add borders and padding, and reduce margin to underline */
        --paper-input-container-input: {
          box-sizing: border-box;
          border: 1px solid var(--paper-grey-400);
          border-bottom: 0;
          padding: 0.5rem;
        };

This should set the mixin in the scope of the `<my-app>` element.
`my-app.html` has a PSK-ish:

      <my-view1 name="view1"></my-view1>

Enter `my-view.html`. HTML-wise, it just has two elements:

    <paper-input label="Normal input"></paper-input>

    <paper-dropdown-menu label="Normal dropdown">
      <paper-listbox class="dropdown-content">
        <paper-item>Item 1</paper-item>
        <paper-item>Item 2</paper-item>
        <paper-item>Item 3</paper-item>
      </paper-listbox>
    </paper-dropdown-menu>

The first `<paper-input>` can be configured, like ANY `<paper-input>`, by setting the mixin `--paper-input-container-input`. However, `<paper-dropdown-menu>` ALSO contains internally a `<paper-input>` element. I want *that* `<paper-input>` element to look a little different. So, I write this in the CSS section of `my-view1.html`:

      /* Input borders in dropdown menus */
      paper-dropdown-menu {
        --paper-input-container-input: {
          box-sizing: border-box;
          padding: 0.5rem;
          border: 1px solid var(--paper-grey-400);
          border-bottom: 0;
          border-right: 0; /* This bleeds out */
          background-color: rgba(255, 0, 0, 0.25); /* This does not */
        };
      }

This should (I hope I am right) set the value for the mixin `--paper-input-container-input` as specified, but _only for elements matching the selector `paper-dropdown-menu`_.
So, the first `<paper-input>` should be untouched.

And yet: 

* Using the custom CSS elements polyfill, it works fine: the normal input has its border.
* Setting `useNativeCSSProperties` to true will show the straight paper input with the missing border on the right
* The issue seems to be there only for the borders. The red transparent background will only happen in `<paper-dropdown-menu>` (as you would expect)

Is this a bug in the native implementation of native CSS properties? Or am I doing something silly?

Safari has the same issue when using native CSS properties.


