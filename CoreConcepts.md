## [Isolated World](https://developer.chrome.com/docs/extensions/develop/concepts/content-scripts#isolated_world)

> [!NOTE]
> Extensions can **run scripts** that read and modify the content of a page. These are called **content scripts**. 

> [!TIP]
> They live in an **isolated world**, meaning they can make changes to their JavaScript environment **without conflicting** with their host page or other extensions' content scripts.

## [ExecutionWorld](https://developer.chrome.com/docs/extensions/reference/api/scripting#type-ExecutionWorld)

### "ISOLATED"  
Specifies the isolated world, which is the execution environment **unique to this extension**.

### "MAIN"  
Specifies the main world of the DOM, which is the execution environment **shared with the host page's JavaScript**.




