# Cross-Domain Communication: Parent window and child iframe

This respository provides you with the example code to enable cross-domain communication between a parent window and child iframe. To learn more about how to use the code and how to enable cross-domain communication (for this specific case), view my Medium article below:

[https://medium.com/@crookse/cross-domain-communication-parent-window-and-child-iframe-aaf90fcb3e26](https://medium.com/@crookse/cross-domain-communication-parent-window-and-child-iframe-aaf90fcb3e26)

The article shows you how to enable cross-domain communication by allowing a child iframe to send data to its parent window. It does not show you how to perform communication from the parent window to the child iframe. If you're interested in performing parent-to-child communication, then read the section below.

## Sending data from the parent window to the child iframe

The parent window can send its child iframes messages. It takes the same code as child-to-parent communication, but differs in three ways:
* The code is reversed
* The `targetWindow.postMessage()` call is called from the parent window
* The `targetWindow` in the `targetWindow.postMessage()` call is the child iframe's window object. For example, `parent.postMessage()` is used in child-to-parent communication and (assuming the use of jQuery) `$("iframe")[0].contentWindow.postMessage()` is used in parent-to-child communication. `$("iframe")[0].contentWindow` is the iframe's window object.

### Steps

*Note: The example code below assumes you're using jQuery.*

1. In the parent window, add the function that sends a message to the child iframe:

    ```javascript
    <script type="text/javascript">
      function sendMessageToChild(message) {
        // Call postMessage on the child iframe's window object
        // (See https://developer.mozilla.org/en-US/docs/Web/API/HTMLIFrameElement/contentWindow
        // for more information on getting the window object of an iframe element)
        $("iframe")[0].contentWindow.postMessage(message, $("iframe").attr("src"));
         console.log("Sent message to the child iframe.");
      }
    </script>
    ```

2. In the child iframe, add the logic that handles messages sent from other windows:

    ```javascript
    <script type="text/javascript">
      // Determine how the browser should listen for messages from other
      // windows. If `addEventListener` exists, then use that. Otherwise, use
      // `attachEvent` because an older browser is probably being used. Also,
      // use a callback to handle messages so that both methods of "message
      // listening" can be routed to the same function. The callback in this
      // example is `handleMessage` and it will take one argument (the
      // `MessageEvent` object).
      if (window.addEventListener) {
        window.addEventListener("message", handleMessage);
      } else {
        window.attachEvent("onmessage", handleMessage);
      }

      /**
       * Handle a message that was sent from some window.
       *
       * @param {MessageEvent} event The MessageEvent object holding the message/data.
       */
      function handleMessage(event) {
          console.log("Received a message from " + event.origin + ".");
    
          // If the window that sent this message is not the parent window, then
          // don't process the message.
          if (event.origin != "http://dev.parent.com") {
              console.log("The message came from some site we don't know. We're not processing it.");
              return;
          }
    
          // When one window sends a message, or data, to another window via
          // `{targetWindow}.postMessage()`, the message (the first argument) in the
          // `{targetWindow}.postMessage()` call is accessible via `event.data` here.
          var dataFromParentWindow = event.data;
    
          console.log(dataFromParentWindow);
      }
    </script>
    ```

3. Open up your project in the browser and type the following in the console:

    ```javascript
    sendMessageToChild("Hello, child iframe!");
    ```

    *Note: You might see `undefined` pop up in the console. That doesn't mean the code is broken. That's going to happen because `sendMessageToChild()` doesn't return anything.*
