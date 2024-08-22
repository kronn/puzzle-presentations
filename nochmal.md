---
revealOptions:
  transition: slide
  markdown:
    breaks: true

---
<!-- .slide: class="l-cover" -->

# *nochmal*
## Migrating User-Uploads

Matthias Viehweger
viehweger@puzzle.ch

----
<!-- .slide: class="l-agenda" -->

## 2024-08-21
# Agenda

- Problem <!-- .element: class="fragment" -->
- Solution <!-- .element: class="fragment" -->
- Approaches <!-- .element: class="fragment" -->
- Take-Aways <!-- .element: class="fragment" -->

----
<!-- .slide: class="l-agenda" -->
# `whoami`

<ul>
<li class="fragment">Matthias Viehweger
<li class="fragment">Ruby since 1.8.7
<li class="fragment">Rails since 2.3
<li class="fragment">Puzzle ITC since 2016
<li class="fragment">github.com/kronn
<li class="fragment">@kronn@ruby.social
</ul>

---
<!-- .slide: class="l-title02" -->
## The Problem
# many uploads and the need to migrate

- over 100GiB <!-- .element: class="fragment" -->
- many files <!-- .element: class="fragment" -->
- multiple apps <!-- .element: class="fragment" -->
- different backend <!-- .element: class="fragment" -->
- different storage <!-- .element: class="fragment" -->

Note: At first, we could not count because GlusterFS. At the end, we did not care. Or at least no long enough to remember it now.

The Backend changed from Carrierwave to ActiveStorage, the storage went from "local" PersistentVolumes with GlusterFS to S3-compatible Buckets.

---
<!-- .slide: class="l-title02" -->
## The Solution
# automation with a funny name

- multiple apps <!-- .element: class="fragment" -->
- needed to reupload uploads <!-- .element: class="fragment" -->
- if extracted, it needed a funny name <!-- .element: class="fragment" -->
- hence: nochmal <!-- .element: class="fragment" -->
- think of baby sinclair/dinosaurs <!-- .element: class="fragment" -->
- or any kid <!-- .element: class="fragment" -->

Note: We needed the solution in multiple apps ourselves, so it was clear that it had to become a gem at some point.

And we program in ruby, so we couldn't name it RailsUploadReuploaderFactoryInstanceProvider

----
<!-- .slide: class="l-thanks" -->

## The Solution
# rake-task for reuploading

`rake nochmal:reupload[from_service,to_service]` <!-- .element: class="fragment" -->

Note: The first app had already active-storage, so we created some code to reupload all the uploads from one service to the other.

----
<!-- .slide: class="l-thanks" -->

## The Solution
# rake-task and guidance for migrating from carrierwave

`gem install nochmal` <!-- .element: class="fragment" -->
`rake nochmal:carrierwave:analyze` <!-- .element: class="fragment" -->
`rake nochmal:carrierwave:migrate` <!-- .element: class="fragment" -->
`rake nochmal:carrierwave:resume` <!-- .element: class="fragment" -->

Note: Then we extracted this into a gem and added the migration from carrierwave.

Analyze suggests a helper that wraps both carrierwave and activestorage, prefering activestorage. Also, it provides some guidance on the expected changes to the application.

Migrate migrates the uploads. It transfers things from the carrierwave-storage to the first configured activestorage service

Resume, well, resumes an interrupted migration. Think OOMKill or so. If you want, you can resume without a prior migrate, but then the task will let you know. If that's what you're after: go ahead.

----
<!-- .slide: class="l-thanks" -->

## The Solution
# control details with env-vars

`NOCHMAL_MIGRATION` <!-- .element: class="fragment" -->
`NOCHMAL_KEEP_METADATA` <!-- .element: class="fragment" -->

Note: Since we run the apps in a kubernetes distribution, we added some ENV-Vars to control how the application reacts to certain stages of the migration.

`NOCHMAL_MIGRATION` as guard to deactivate the migrations for the uploader itself

`NOCHMAL_KEEP_METADATA` if you want to look into the metadata needed for resuming after the reupload

----
<!-- .slide: class="l-thanks" -->

## The Solution
# And then run it

around 42 deployments <!-- .element: class="fragment" -->
6 apps IIRC <!-- .element: class="fragment" -->
approx 120GiB <!-- .element: class="fragment" -->
no data-loss <!-- .element: class="fragment" -->
no downtime <!-- .element: class="fragment" -->
migration itself was done in a week <!-- .element: class="fragment" -->

---
<!-- .slide: class="l-title02" -->
## Alternative Approaches
# how hard can it be?

- copy files manually <!-- .element: class="fragment" -->
- accept downtime <!-- .element: class="fragment" -->
- accept data-loss and start over <!-- .element: class="fragment" -->

Note: Migrating one file might take 5 to 15 minutes, so...

No transitionary Code, no cleanup, no problem? But also: no uptime

Why not just nuke it and blame russian hackers from the CIA or so?

---
<!-- .slide: class="l-title02" -->

## Take-Aways

- Uploads are easy <!-- .element: class="fragment" -->
- Switching Upload-Backends is hard <!-- .element: class="fragment" -->
- Or: was. 😜 <!-- .element: class="fragment" -->

Note:
- Copying one file is easy
- Copying two files is boring
- Copying > 100 GiB in many small files manually while migrating the application-data from carrierwave to activestorage is...
  lol, nope


---
<!-- .slide: class="l-thanks" -->

# Merci

- github.com/puzzle/nochmal
---
<!-- .slide: class="l-cover" -->

# *nochmal*
## Migrating User-Uploads

```bash
github.com/puzzle/nochmal
```

Matthias Viehweger

viehweger@puzzle.ch
@kronn@ruby.social
