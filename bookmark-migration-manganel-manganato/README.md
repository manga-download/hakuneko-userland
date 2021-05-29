The following sample script goes through all your bookmarks and migrate the old MangaNel bookmarks to the new Manganato format  
This script can be injected directly into HakuNeko (Developer Tools => Console Tab)

:information_source: Before running this script, make sure the manga list from Manganato is updated :information_source:  
:warning: Before running this script, make sure to backup your bookmark file :warning:  
See: https://hakuneko.download/docs/install/#user-data

```javascript
(async function() {
    const manganel = Engine.Connectors.find(connector => connector.id === 'manganel');
    await manganel._getUpdatedMangasFromFile();
    if(!manganel.mangaCache || !manganel.mangaCache.length || manganel.mangaCache.some(manga => manga.id.includes('/manga/'))) {
        return console.error('Failed to migrate bookmarks from manganel to manganato! Please synchronize (update) the manga list from manganato with HakuNeko firdt before running this script.');
    }
    for(let bookmark of Engine.BookmarkManager.bookmarks) {
        if(bookmark.key.connector === 'manganel' && (bookmark.key.manga.includes('manganelo.com/manga/') || bookmark.key.manga.startsWith('/manga/'))) {
            try {
                const matches = manganel.mangaCache.filter(manga => manga.title.toLowerCase() === bookmark.title.manga.toLowerCase());
                if(matches.length === 0) {
                    console.warn('Bookmark not updated (need to be done manually in HakuNeko)! No matching manga found for:', bookmark.title.manga);
                }
                if(matches.length === 1) {
                    console.log('Updated:', bookmark.key.manga, '=>', matches[0].id);
                    bookmark.key.manga = matches[0].id;
                }
                if(matches.length > 1) {
                    console.warn('Bookmark not updated (need to be done manually in HakuNeko)! Too many matching mangas found for:', bookmark.title.manga, '=>', matches.map(manga => manga.title).join(' || '));
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
