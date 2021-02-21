---
layout: page
title: Getting started dictating mathematics
---

*21/02/2021 Page updated to recommend [talon](https://talonvoice.com/) for mathematics dictation rather than mathfly, which is now outdated. The old pages can be found [here]({{ site.baseurl }}/index_old.html), and the old instructions will still work, but if you are looking to get started dictating maths then proceed on this page.*

---

# What is talon?
[Talon](https://talonvoice.com/) is a free, cross-platform application that provides customisable voice control and eye tracking. It can be used to create voice commands for almost anything, including navigating a browser, writing code and in our case, **dictating maths into a document processor**. Talon contains a built-in speech recognition engine, and on Windows it can load commands into Dragon NaturallySpeaking, so it can work alone or as a drop-in replacement for the old mathfly system and others like MathTalk.

This video gives a brief introduction to talon, and a demonstration of what dictating maths with it looks like (thanks to [ma-anwar](https://github.com/ma-anwar) for the great video):

{% include youtubePlayer.html id="3wiC3xKnVgc" %}

---

# Dictating mathematics using talon

Talon scripts for dictating maths into into the [LyX](https://www.lyx.org/) or [Scientific Notebook 5.5](https://www.mackichan.com/index.html?products/dnloadreq55.html~mainFrame) document processors can be found by following this link: [https://github.com/mrob95/mathfly-talon](https://github.com/mrob95/mathfly-talon).

If you are new to all of this, follow these instructions to get started:
1. Start by following the [talon getting started guide](https://talonvoice.com/docs/) to download talon and a general purpose set of scripts (currently `knausj_talon`).
2. Once you have talon installed follow the [knausj_talon getting started guide](https://github.com/knausj85/knausj_talon#getting-started-with-talon) to familiarise yourself with the basic commands, including the phonetic alphabet. The learning curve here is quite steep, but it is worth persevering, as this is by far the most efficient and powerful way to control a computer by voice.
3. Once you are set up and comfortable with talon, download or clone the [mathfly-talon repository](https://github.com/mrob95/mathfly-talon) into your user folder to add maths commands. To get started, say the `help maths` command while in LyX or Scientific Notebook for instructions and a list of available commands.

If you have any problems with getting set up or using these scripts (or just want to chat to other users), feel free to ask for help in the [#maths channel on the talon slack](https://app.slack.com/client/T7FPSMV8F/C01ETRZNT46).

---

Links:
* [Short video demonstration](https://www.youtube.com/watch?v=7eZ6fMztvwA)
* [#maths slack channel](https://app.slack.com/client/T7FPSMV8F/C01ETRZNT46)
* [ma-anwar's version of these configs, with various additions](https://github.com/ma-anwar/mathfly)
