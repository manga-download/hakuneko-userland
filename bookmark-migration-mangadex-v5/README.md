The following sample script goes through all your bookmarks and migrate the old MangaDex bookmarks to the new v5 format  
This script can be injected directly into HakuNeko (Developer Tools => Console Tab)

:warning: Before running this script, make sure to backup your bookmark file :warning:  
See: https://hakuneko.download/docs/install/#user-data

```javascript
(async function() {
    for(let bookmark of Engine.BookmarkManager.bookmarks) {
        if(bookmark.key.connector === 'mangadex' && !bookmark.key.manga.includes('-')) {
            try {
                await new Promise(resolve => setTimeout(resolve, 250)); // throttle, max. 5 req/sec
                const oldID = parseInt(bookmark.key.manga.match(/\d+/));
                const response = await fetch('https://api.mangadex.org/legacy/mapping', {
                    method: 'POST',
                    body: JSON.stringify({ type: 'manga', ids: [ oldID ] }),
                    headers: { 'content-type': 'application/json' }
                });
                const data = await response.json();
                if(data[0].result === 'ok' && data[0].data.attributes.legacyId === oldID) {
                    bookmark.key.manga = data[0].data.attributes.newId;
                    console.log('Updated:', bookmark.title.manga, '|', oldID, '=>', bookmark.key.manga);
                } else {
                    throw new Error('Failed to get new ID from MangaDex API');
                }
            } catch (error) {
                console.warn('Failed to get new manga ID for: ', bookmark.key.manga, error);
            }
        }
    }
})();
```

After running, check the output for warnings/errors  
If the update failed, simply restart HakuNeko to avoid breaking your bookmark file  
If the update was successful, inject the following script to save your migrated bookmark file

```javascript
Engine.BookmarkManager.saveProfile('default', undefined);
```
