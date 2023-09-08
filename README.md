# BridgeU Etherpad comments plugin.

This repository represents BridgeU's modifications of the ep_comments_page
Etherpad plugin.

We use the comment plugin's functionality in our platform so that teachers can
add comments to student's writing builder work (and the students can reply). In
general, things that happen in Etherpad stay in Etherpad - most of the work
happens in an iframe and communicates directly with the Etherpad app
server/database. We don't even especially worry about tracking the content of
the documents in the parent database, so long as students are able to get back
to their original document later.

However, we do want to know when a teacher leaves a comment on a student's
document, so that we can send an in-app/email notification to the student
immediately.

Unfortunately, the architecture of the comments plugin makes it really hard for
us to access that information from outside of the comment plugin itself, if we
don't hook directly into the plugin's functionality. This plugin does not
generate any externally trackable events, instead it communicates directly with
the server using its own channel within a websocket, that other client-side code
doesn't have any visibility of.

This gave us two approaches to implementing comment notifications in BridgeU's
writing builder:

1. Write our own Etherpad plugin and find a way for that plugin to a) be aware
    of the messages emitted from the comments plugin, and b) communicate that
    from the server back to the client from our *own* websocket connection.

    In theory this is very plausible, but in practice turned out to be tricky
    enough that it didn't seem like a good investment of time

2. Fork the comment plugin and inject our own event handling code directly into
    the comment creation process.

    This isn't a perfect process as any diversion from the base functionality
    makes it trickier to upgrade the comment plugin at a later date.

3. For completeness, we could also hook into the comment creation process from a
    server-side point of view and trigger notifications in our parent project
    directly from the Etherpad backend.

    Actually this would be the ideal case, but would require a significant
    architectural change which does not have enough benefit in a cost/benefit
    comparison.

This repository represents the outcome of using option 2 as our approach.

What follows is the original README for this project:

---

![Publish Status](https://github.com/ether/ep_comments_page/workflows/Node.js%20Package/badge.svg) ![Backend Tests Status](https://github.com/ether/ep_comments_page/workflows/Backend%20tests/badge.svg)

# Comments and annotations for Etherpad

![Screen shot](https://user-images.githubusercontent.com/220864/98013526-617ff900-1df2-11eb-88b6-cf259372f6ca.PNG)

## Installing this plugin with npm.
```
npm install ep_comments_page
```

## Extra settings
This plugin has some extra features that can be enabled by changing values on `settings.json` of your Etherpad instance.

### Alternative comment display
There is an alternative way to display comments. Instead of having all comments visible on the right of the page, you can have just an icon on the right margin of the page. Comment details are displayed when user clicks on the comment icon:

![Screen shot](http://i.imgur.com/cEo7PdL.png)

To use this way of displaying comments, add the following to your `settings.json`:
```
// Display comments as icons, not boxes
"ep_comments_page": {
  "displayCommentAsIcon": true
},
```

### Highlight selected text when creating a comment
It is also possible to mark the text originally selected when user adds a comment:
![Screen shot](http://i.imgur.com/AhaVgRZ.png)

To enable this feature, add the following code to your `settings.json`:
```
// Highlight selected text when adding comment
"ep_comments_page": {
  "highlightSelectedText": true
},
```

**Warning**: there is a side effect when you enable this feature: a revision is created everytime the text is highlighted, resulting on apparently "empty" changes when you check your pad on the timeslider. If that is an issue for you, we don't recommend you to use this feature.

### Disable HTML export
By default comments are exported to HTML, but if you don't wish to do that then you can disable it by adding the following to your `settings.json`:
```
"ep_comments_page": {
  "exportHtml": false
},
```

## Creating comment via API
If you need to add comments to a pad:

* Call this route to create the comments on Etherpad and get the comment ids:
  ```
  curl -X POST http://localhost:9001/p/THE_PAD_ID/comments -d "apikey=YOUR_API_KEY" -d 'data=[{"name":"AUTHOR","text":"COMMENT"}, {"name":"ANOTHER_AUTHOR","text":"ANOTHER_COMMENT"}]'
  ```

  The response will be:
  ```
  {"code":0,"commentIds":["c-VEtzKolgD5krJOVU","c-B8MEmAT0NJ9usUwc"]}
  ```

* Use the returned comment ids to set the pad HTML [via API](http://etherpad.org/doc/v1.5.6/#index_sethtml_padid_html):
  ```
  My comment goes <span class="comment c-VEtzKolgD5krJOVU">here<span>.
  ```

  Result:
  ![Screen shot](http://i.imgur.com/KM4lPJx.png)

NOTE: Adding a comment to a pad via API will make the other editors with that pad to be alerted, but this feature is only active if your Etherpad is run in `loadTest` mode. Read [the Etherpad Guide](https://github.com/ether/etherpad-lite/wiki/Load-Testing-Etherpad) for how to enable load testing.

## License
Apache 2
