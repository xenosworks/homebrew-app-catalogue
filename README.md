# Homebrew App Catalogue

The catalogue the [Homebrew App Store](https://github.com/xenosworks/homebrew-app-store) shows on the Xbox 360:
`catalogue.json`, its signature `catalogue.json.sig`, and each app's screenshot in `shots/`, served from this repository
by GitHub Pages at `https://xenosworks.github.io/homebrew-app-catalogue/catalogue.json`.

The store takes the catalogue only when xenosworks's key signed it, and each download only when its SHA-256 is the one
the catalogue gives, so the signature covers every package and screenshot the store shows or installs. The signature
says the list is the one xenosworks published, with each package's exact hash; it doesn't say xenosworks wrote an app.
Before an app not by xenosworks installs for the first time, the store says so and waits for A.

## Listing your app

Anyone can list homebrew here by pull request. Your app needs:

- a `.xex` that runs on a console that runs unsigned code, with its title ID and a version in its execution info, as
  [xsdk](https://github.com/xenosworks/xsdk)'s `xsdk_add_xex(target TITLE_ID ... NAME "..." VERSION 1.2.300.4)` writes
  them; the store compares versions from there, so each release raises it;
- its source, public, under a licence that lets it be shared;
- a package: a zip of the files that go in the app's folder, `hdd:\apps\<id>\`, published as a release asset of your
  app's own repository. Make it with xsdk's `tools/store_catalogue.py pack`, which zips the folder and writes the zip's
  size and SHA-256, each file's, and the `.xex`'s title ID and version into your entry;
- a screenshot of the app itself on the console, a PNG or JPEG of 2 MB at most, 1280x720, in `shots/<id>.png`.

Your pull request adds your entry to `catalogue.json` and your screenshot to `shots/`:

```json
{"id": "my-app", "name": "My App", "author": "Your name", "category": "tools",
 "sentence": "One line saying what it is.", "needs": "What the player supplies, or nothing",
 "changes": "What's new in this version.", "source": "https://github.com/you/my-app", "license": "MIT",
 "screenshot": "shots/my-app.png", "launch": "my_app.xex"}
```

then:

```bash
python3 <xsdk>/tools/store_catalogue.py pack catalogue.json my-app <your app's folder> my-app-1.0.0.0.zip \
    --url https://github.com/you/my-app/releases/download/v1.0.0.0/my-app-1.0.0.0.zip
python3 <xsdk>/tools/store_catalogue.py check catalogue.json
```

`needs` shows on one line of the app's page, which holds about 40 characters; the store cuts a longer one short.
Upload the zip to that release as it is: the catalogue holds its SHA-256, and any other zip is refused. Ids are up to
32 lower-case letters, digits and `-`.

FATX, the console's file system, takes up to 42 characters in a file or folder name and 250 in a path, not counting
the drive ([FATXplorer](https://fatxplorer.eaton-works.com/)). The store writes each file as `<name>.part` before
renaming it, so each file or folder name in a package has 37 characters at most, and `check` refuses a longer one.
Paths have room to spare: with `\apps\<id>\` in front, each path in a package can have over 200 characters.

## Signing

xenosworks reviews each pull request, then signs the catalogue on a machine of its own with a key kept off line:

```bash
python3 <xsdk>/tools/store_catalogue.py sign catalogue.json <the key>
python3 <xsdk>/tools/store_catalogue.py check catalogue.json --public <the key>.pub.pem --packages <the packages>
```

Signing writes each screenshot's SHA-256 into the catalogue and raises its serial; the store refuses a catalogue older
than the one it has, so an old catalogue can't take apps back to older versions.
