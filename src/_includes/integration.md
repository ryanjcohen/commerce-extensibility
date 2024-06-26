The `create`, `update`, and `delete` runtime actions in the Adobe Commerce integration starter kit perform one of the following functions:

- [Expose Commerce data](../pages/starter-kit/send-data.md) - Notifies an external back-office application when an `<object>` is created, updated, or deleted in Adobe Commerce. Actions that react to Adobe Commerce events and notify the external back-office application are located in the `actions/<object>/commerce` folder.
- [Enrich Shopping experience](../pages/starter-kit/receive-data.md) - Notifies Adobe Commerce when an `<object>` is created, updated, or deleted in an external back-office application. Actions that react to back-office application events and notify Adobe Commerce are located in the `actions/<object>/external` folder.

![starter kit diagram](../pages/_images/starter-kit.png)

- **Preprocessing data** - Any preprocessing needed before calling the Adobe Commerce API can be implemented in the `preProcess` function in the `pre.js` file.
- **Postprocessing data** - Any postprocessing needed after calling the Adobe Commerce API can be implemented in the `postProcess` function in the `post.js` file.
