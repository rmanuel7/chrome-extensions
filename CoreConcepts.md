### [Isolated World](https://developer.chrome.com/docs/extensions/develop/concepts/content-scripts#isolated_world)

> [!NOTE]
> Extensions can **run scripts** that read and modify the content of a page. These are called **content scripts**. 

> [!TIP]
> They live in an **isolated world**, meaning they can make changes to their JavaScript environment **without conflicting** with their host page or other extensions' content scripts.

---

### [ExecutionWorld](https://developer.chrome.com/docs/extensions/reference/api/scripting#type-ExecutionWorld)

#### "ISOLATED"  
Specifies the isolated world, which is the execution environment **unique to this extension**.

#### "MAIN"  
Specifies the main world of the DOM, which is the execution environment **shared with the host page's JavaScript**.

---

### [One-time requests](https://developer.chrome.com/docs/extensions/develop/concepts/messaging#simple)
> [!NOTE]
> To send **a single message to another part of your extension**, and optionally get a response, call [`runtime.sendMessage()`](https://developer.chrome.com/docs/extensions/reference/api/runtime#method-sendMessage) or [`tabs.sendMessage()`](https://developer.chrome.com/docs/extensions/reference/api/tabs#method-sendMessage).

> [!IMPORTANT]
> These methods let you send a one-time **JSON-serializable message** from a **content script to the extension**, or from **the extension to a content script**.

> [!TIP]
> You **can't** use a promise and a callback in the same call.
