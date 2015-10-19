+++
date = 2015-10-06T13:30:00Z
title = "How we stopped using the mouse and started drawing molecules with our fingertips"
lang = "en"

[[conferences]]
name = "Qt World Summit"
date = 2015-10-06T13:30:00Z

[[material]]
name = "Slides"
url  = "https://speakerdeck.com/masci/how-we-stopped-using-the-mouse-and-started-drawing-molecules-with-our-fingertips-not-the-usual-porting-story"

+++

Porting to mobile a Qt desktop application that lets you draw molecules, crunches numbers, stores data, displays plot and graphs it’s something that goes far beyond converting a QWidget to a Qml component. You have to change user's perspective, merging what they expect from a mobile application with what they expect from a scientific software. You have to 
outsource heavy computational parts and data storage. You have to code from scratch components that don’t exist yet in the Qml ecosystem.

With the arrival of Qt 5 and the support for Android and iOS, we realized that simply moving our legacy code to the new version of the framework still keeping the QWidget approach wouldn’t be worth it: we needed to port our applications to Qml and open to a whole new set of platforms.

In this talk you’ll get the lessons learned during a porting journey lasted more than 6 months, with particular reference to the following topics:

- Rethink a complex user model for drawing molecules so that one can use their fingers on mobile and touch screens, still keeping it usable on a desktop through the mouse 
- Moving heavy computations on the cloud using REST and OAuth2 to exchange data. 
- Provide a theming system for the UI, following Material Design principles but keeping a good user experience on the desktop 
- Refactor graphic and drawing components with a pixel independent approach 
- Develop a brand new interactive 2D plot with Qml in mind
