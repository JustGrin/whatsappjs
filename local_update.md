## 更新日志

###  2025-7-23

client.getChats 添加参数,让getChats接口可以分批请求。 修改whatsappjs库

1. `whatsappjs\src\Client.js`
```ts
  /**
     * Get all current chat instances
     * @returns {Promise<Array<Chat>>}
     */
    async getChats() {
        const chats = await this.pupPage.evaluate(async () => {
            return await window.WWebJS.getChats();
        });

        return chats.map(chat => ChatFactory.create(this, chat));
    }
```
改为

```ts
    /**
     * Get all current chat instances
     * @param {Object} searchOptions Options for searching chats. Right now only since is supported.
     * @param {Number} [searchOptions.skip] Only chats whose internal timestamp `chat.t` is **greater than or equal** to this value will be returned.
     * @param {Number} [searchOptions.limit] Only chats whose internal timestamp `chat.t` is **greater than or equal** to this value will be returned.
     * @returns {Promise<Array<Chat>>}
     */
    async getChats(searchOptions) {
        const chats = await this.pupPage.evaluate(async (options) => {
            return await window.WWebJS.getChats({...options});
        }, searchOptions);

        return chats.map(chat => ChatFactory.create(this, chat));
    }


```

2. `whatsappjs\src\util\Injected\Utils.js`

```ts
    window.WWebJS.getChats = async () => {
        const chats = window.Store.Chat.getModelsArray();
        const chatPromises = chats.map(chat => window.WWebJS.getChatModel(chat));
        return await Promise.all(chatPromises);
    };
```
改为
```ts
      window.WWebJS.getChats = async (options = {}) => {
        const { skip = 0, limit = 0 } = options;

        const allChats = window.Store.Chat.getModelsArray();
        const filteredChats = limit
            ? allChats.slice(skip, skip + limit)
            : allChats;
        
        const chatPromises = filteredChats.map(chat => window.WWebJS.getChatModel(chat));
        return await Promise.all(chatPromises);
    };
```