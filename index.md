---
layout: page
title: What is it?
---

A free, open source utility for dictating mathematical notation and LaTeX quickly and easily using [Dragon Professional Individual](https://www.nuance.com/en-gb/dragon/business-solutions/dragon-professional-individual.html). Currently supports the free, open source LyX document processor and Scientific Notebook 5.5 (allowing for drop-in replacement of MathTalk). Based on Natlink, dragonfly and Caster. With thanks to Alex Boche and David Conway.

* [GitHub](https://github.com/mrob95/mathfly)
* [Gitter channel](https://gitter.im/mathfly-dictation/community) - [![Join the chat at https://gitter.im/mathfly-dictation/community](https://badges.gitter.im/mathfly-dictation/community.svg)](https://gitter.im/mathfly-dictation/community?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
* Contact: [mr1128@york.ac.uk](mailto:mr1128@york.ac.uk)
* [Installation instructions]({{ site.baseurl }}/installation.html)
* For complete lists of the commands which Mathfly provides, read the [Core]({{ site.baseurl }}/docs/Core.pdf), [LyX]({{ site.baseurl }}/docs/LyX.pdf), [Scientific Notebook 5.5]({{ site.baseurl }}/docs/Scientific_Notebook.pdf) and [LaTeX]({{ site.baseurl }}/docs/LaTeX.pdf) documentation.

***

## Quick
Mathfly interprets dictated commands on-the-fly quickly and continuously, making it far faster than other systems. As fast as you can speak, Mathfly can interpret.

While I have no data comparing dictation speed with normal writing speed, I will say that I have been using versions of this software to do exams throughout my economics undergraduate degree, and have never needed extra time or struggled to finish. Video demonstration:

{% include youtubePlayer.html id="7eZ6fMztvwA" %}

***

## Infinitely modifiable
The vast majority of Mathfly\'s commands are contained in plaintext config files in `mathfly/config/`, designed to be easy to edit and extend. If you need a new command, it\'s as simple as adding one line of plaintext. If you keep getting a command wrong, you can change it. If there are commands you don\'t need, you can delete them.

{% include youtubePlayer.html id="vLwu9SWK030" %}

For an even easier way of adding new commands, `alias` turns highlighted text into a new command.

{% include youtubePlayer.html id="mZ-Y8O5RrUY" %}

***

## LaTeX
Mathfly also contains a module for dictating into LaTeX documents, with commands for inserting objects, beginning and ending environments, reproducing premade templates, and crawling the web for references, all with simple voice commands. For a full list of commands, read the [LaTeX]({{ site.baseurl }}/docs/LaTeX.pdf) documentation.

{% include youtubePlayer.html id="N9OzgJFP8tM" %}