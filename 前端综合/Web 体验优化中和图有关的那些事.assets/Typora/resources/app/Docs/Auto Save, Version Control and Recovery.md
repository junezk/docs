# Auto Save, Version Control and Recovery

## macOS

On macOS, the operation system will schedule auto save operation for document-based apps, like Typora or TextEdit. So "auto-save" is always enabled as a system feature.

![general](img/general.png)

If you want Typora to auto save content when quit or close, without popping up a confirm dialog, please **uncheck** the checkbox one.

If you want Typora to restore all windows/documents when restart, please **uncheck** the checkbox two.

## Windows/Linux 

 ![Snip20161027_2](img/Snip20161027_2.png)

You could enable this feature on preferences panel.

By default, the documents will be saved every 5 minutes. 

If you want to change the time interval, please click "Open Advanced Settings" button on preferences panel, which would pop up a folder named `conf`, then edit or create a file named `conf.user.json`, and modify/add following setting:

```json
{
  "autoSaveTimer": 5 // Double, default is 5. The unit is "minute"
}
```

### Recover Unsaved Drafts (Windows/Linux)

No matter whether the "auto-save" option is enabled or not, if Typora exit or crashed without saving file, or you accidentally quite Typora without saving your writings, you could click the "Recover Unsaved Drafts" button to found some writing drafts auto saved by Typora.

The filename of those backed-up drafts is like `{date}-{filename}.md`, if your content is newly created without a file path (which is, "Untitled"), the `{filename}` part is auto generated, which is usually the first heading or first sentence. You could find and copy out the corresponding backup file to retrieve some part of your writings.