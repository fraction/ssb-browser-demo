# SSB browser demo

![Screenshot of ssb browser demo][screenshot]

Playground [ssb-server] in a browser. This was originally
made as a demo for my bornhack [talk][bornhack-talk].

The client was made for two purposes: test ssb in a browser and for
evaluating different partial replication strategies.

Partial replication is done by only getting the last few `post`
messages for users as a way of onboarding users quickly. The client
can download and index around 5.000 messages in 30 seconds on my
really slow laptop (same speed on a phone). For this, a blob
(generate-onboarding-json.js) must be provided that serves as a
trusted onboard mechanism and as such should only be used between
friends. Currently it can be hard to find new users of this client
because one doesn't follow a pub. I plan on fixing this using
[ssb-peer-invites]. Another way this project tries to make partial
replication better is by using the [ssb-contact-msg] library where
contact messages are linked together.

As a way to let people explore the messags of users outside this
initial list of people, the [ssb-get-thread] plugin is used to get
threads from the server. This has privacy implications so this need to
be configurable, see TODO.

The UI is written in vue.js and can display posts and self assigned
profile about messages. Leaving out likes was done on purpose as an
experiment. I don't plan on adding them.

Things that work:
 - viewing posts and threads
 - posting and replying to messages
 - posting blobs
 - viewing profiles and setting up your own profile
 - private messages including decrypting private blobs
 - off-chain chat using ssb-tunnel for e2e encrypted messages
 - ooo messages

Tested with Chrome and Firefox. Chrome is faster because it uses fs
instead of indexeddb.

An online version is available for testing [here][test-server]

# Running locally

For testing this in Chrome locally, one needs to be run it with:
--allow-file-access-from-files

Also be sure to enable all 5 WASM options in
chrome://flags/. Otherwise crypto will be super slow (like validate).

The following patches (patch -p0 < x.patch) from the patches folder
are needed:
 - epidemic-broadcast-fix-replicate-multiple.patch
 - flumeview-level-mkdirp.patch
 - ssb-ebt.patch
 - ssb-friends.patch
 - ssb-tunnel.patch

The following branches are references directly until patches are merged and pushed:
 - https://github.com/dominictarr/flumelog-aligned-offset/pull/1
 - https://github.com/flumedb/flumeview-level/pull/22
 - https://github.com/ssbc/ssb-validate/pull/16

For a smaller bundle file, you can also apply
patches/sodium-browserify.patch

# Server

Server needs to have ws enabled.

```
"ws": [
 { "port": 8989, "host": "::", "scope": "public", "transform": "shs" }
]
```

The
[ssb-partial-replication](https://github.com/arj03/ssb-partial-replication)
plugin is needed for faster sync. Also the
[ssb-get-thread](https://github.com/arj03/ssb-get-thread) plugin is
used for browsing threads you don't currently have, such as from
people outside the people you have synced or older messages because of
the partial replication nature.

# Onboarding file

Generate a file of all feeds following with seq nos. The perspective
(user) can be changed in top.

```
node generate-onboarding-json.js > onboard.json
```

Add it as a blob to the network:

```
cat onboard.json | ../ssb-minimal-pub-server/bin.js blobs.add
```

=> something like

&YTbAj6FWYBZOyKLeGXb+Kb4wLIlEoryq9PL1nhlCek4=.sha256

# browserify 2mb

Removing blobs means that we go down to 1.6mb. ssb-backlinks brings
this back to 2mb because of level.

browserify --full-paths core.js > bundle-core.js
browserify --full-paths ui/browser.js > bundle-ui.js

ssb-markdown increases the size quite substantially

# TODO

- exif stripping for images
- peer-invites
- port over ssb-friend-pub
- disable or trust pubs as a way to control when to fetch threads

## uglifyify

browserify --full-paths -g uglifyify -p common-shakeify core.js > bundle-core.js
browserify --full-paths -g uglifyify -p common-shakeify browser-test.js > bundle-test.js

=> 1.2mb

# Other

## Force WASM locally (outside browser)

rm -rf node_modules/sodium-chloride/

## check contents of db

```
var pull = require("pull-stream")

pull(
  store.stream(),
  pull.drain((msg) => {
    console.log(msg)
  })
)
```

List all files in browser

``` javascript
function listDir(fs, path)
{
	fs.root.getDirectory(path, {}, function(dirEntry){
		var dirReader = dirEntry.createReader();
		dirReader.readEntries(function(entries) {
		for(var i = 0; i < entries.length; i++) {
			var entry = entries[i];
			if (entry.isDirectory) {
				console.log('Directory: ' + entry.fullPath);
				listDir(fs, entry.fullPath)
			}
			else if (entry.isFile)
                console.log('File: ' + entry.fullPath);
			}
		})
	})
}

window.webkitRequestFileSystem(window.PERSISTENT, 0, function (fs) {
	listDir(fs, '/.ssb-lite/')
})
```

## indexes

Backlinks & query uses flumeview-level that stores it's db in indexdb
in the browser. These indexes are much slower in the browser.

## oasis

%owvZa0OwPBH2olaKUiex1wqyBO/+AeBlxEHcH7jORtA=.sha256

## mcss generate css

mcss plugs/app/page/books.mcss -o books.css

[screenshot]: assets/screenshot.jpg
[ssb-server]: https://github.com/ssbc/ssb-server
[bornhack-talk]: https://people.iola.dk/arj/2019/08/11/bornhack-talk/
[ssb-get-thread]: https://github.com/arj03/ssb-get-thread
[ssb-peer-invites]: https://github.com/ssbc/ssb-peer-invites
[test-server]: https://between-two-worlds.dk/browser.html
[ssb-contact-msg]: https://github.com/ssbc/ssb-contact-msg
