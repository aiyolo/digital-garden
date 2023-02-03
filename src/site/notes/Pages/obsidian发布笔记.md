---
{"aliases":null,"tags":[null],"source":null,"created":"2023-01-14 14:43:21","updated":"2023-01-22 19:14:40","title":"Obsidian 发布笔记","dg-publish":true,"permalink":"/Pages/obsidian发布笔记/","dgPassFrontmatter":true}
---


# Obsidian 发布笔记

1. First off, you will need a GitHub account. If you don't have this, create one [here](https://github.com/signup).
2. You'll also need a Netlify account. You can sign up using your GitHub account [here](https://app.netlify.com/)
3. Open [this repo](https://github.com/oleeskild/digitalgarden), and click the green "Deploy to netlify" button. This will open netlify which in turn will create a copy of this repository in your GitHub accont. Give it a fitting name like 'my-digital-garden'. Follow the steps to publish your site to the internet.
4. Now you need to create an access token so that the plugin can add new notes to the repo on your behalf. Go to [this page](https://github.com/settings/tokens/new?scopes=repo) while logged in to GitHub. The correct settings should already be applied. If you don't want to generate this every few months, choose the "No expiration" option. Click the "Generate token" button, and copy the token you are presented with on the next page.
5. In Obsidian open the setting menu and find the settings for "Digital Garden". The top three settings here is required for the plugin to work. Fill in your GitHub username, the name of the repo with your notes which you created in step 3. Lastly paste the token you created in step 4. The other options are optional. You can leave them as is.
6. Now, let's publish your first note! Create a new note in Obsidian. And add the following to the top of your file.

```
---
dg-home: true
dg-publish: true
---
```

**This does two things:**

- The dg-home setting tells the plugin that this should be your home page or entry into your digital garden. (It only needs to be added to _one_ note, not every note you'll publish).
	
- The dg-publish setting tells the plugin that this note should be published to your digital garden. Notes without this setting will not be published. (In other terms: Every note you publish will need this setting.)

7\. Open your command pallete by pressing CTRL+P on Windows/Linux (CMD+P on Mac) and find the "Digital Garden: Publish Single Note" command. Press enter.  
8\. Go to your site's URL which you should find on [Netlify](https://app.netlify.com/). If nothing shows up yet, wait a minute and refresh. Your note should now appear.
