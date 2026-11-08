---
name: ketchup-video
description: "Catch the user up like ketchup, with a short subtitled video of slides for what happened and the waiting calls as text under it. Use for ketchup-video."
---

# Ketchup video

Follow the ketchup skill (`../ketchup/SKILL.md`), but show what happened in a short subtitled video instead of its summary and visuals. Its window starts at the last message the user wrote before `ketchup-video`, not before any earlier `ketchup`. Writing and rendering the video, outside the repo, is the one exception to its read-only rule. The reply is ketchup's opening sentence, the embedded video, one line naming any work left that you can do yourself, and the action items. When the video shows a call's options, call each option by the name the video gives it. If the render fails, say so in one line and give only the calls.

## Slides

Use the screenshots, recordings, diffs, and data the work produced, not the chat's prose about them. Before you draw, write the one sentence the user should leave with and one line per slide. Drop each slide that does not change what the user knows. Use at most 10 slides, and fewer is better. Neither note goes in the reply.

Each slide makes one point. Build the slides on one picture and change it from slide to slide, rather than cut to an unrelated one. A color or shape means the same thing on every slide. Show the work in pictures, such as how a system works, a PR stack with each PR's state, a before and after of a fix, or options side by side. Point at what changed with arrows, highlights, and motion. A slide is never a bullet list, a status list, a table, or boxes with nothing connecting them.

Keep text to the minimum. Put each label on the thing it names, in plain words a person would say, not tags like "Tests: pass". Never caption what a picture shows, and never give a fact twice, across slides or in the text under them.

Show what came of each thing the user asked for, and whether it is done, still running, failed, or not checked. Never show something done that still waits on the user. Show any bug found and fixed on the way. If a visible change has no screenshot, say so on its slide and name what changed on screen. Draw only from facts you have. Never invent a number, and never draw a screen you did not capture.

## Subtitles

Give each slide one short subtitle line, about ten words, that says its point the way you would tell the user, from only what that slide shows. Say what the picture means rather than read out its labels. Put the subtitle in a strip below the slide, the same height on every slide and empty except for the subtitle, so it never covers the slide. Set it small, but readable when the video plays at chat size.

## Render

Render one silent landscape video, such as an mp4, with motion between slides. Every frame is the slide itself, edge to edge, above its subtitle strip. Make the frames from the slides, never by recording a screen, and burn each subtitle into its slide's frames so it stays on screen for the slide's whole hold. The hold is the time a slide is fully on screen, not counting the motion into or out of it. Hold each slide long enough to read the words it adds, at least one second for each three words. A slide adds its subtitle and each word that was not on the slide before it, including a label it changes. The first slide adds all its words. Words inside a screenshot do not count.

Build each slide as an image first, then assemble the video with Muse's media tools, giving the tool the slide images, the subtitle text for each slide, and the hold length each slide needs. The media pipeline generates video from a prompt and source images; it does not do frame-accurate compositing, so describe the layout exactly (slide edge to edge above a subtitle strip of constant height, subtitle burned in for the whole hold) and keep the slide count low so the result stays faithful.

Before you embed the video, check the rendered file. Watch it at chat size and check that each subtitle is readable and covers nothing, and that each slide holds long enough to read the words it adds. Fix any slide you cannot read in that time, render again, and check again.

Skipped in Muse: the scripted frame-by-frame hold measurement. Upstream matches the rendered file's frames to each slide to measure every hold exactly; Muse's media tools expose no frame-level timing to measure against, so the check above is a watch-through, not a measurement. If the media tool cannot hold a slide long enough or burn subtitles in reliably, say so in one line and give only the calls, as for any render failure.
