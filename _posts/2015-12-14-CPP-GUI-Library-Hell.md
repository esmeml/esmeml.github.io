---
layout: post
title: C++ GUI Libraries
date: 2015-12-14 15:30:00 -05:00
license: cc0
categories:
- Work
tags:
- Near The Resolution
---
So, one of the many projects that I have been working on as of late is a game
engine for a small game studio called Near The Resolution. Really awesome
people to work for, lemme tell ya. They are allowing me to release the engine
under one of my crazy "No Rights Reserved" licenses. I have learned that one
of the unfortunate parts of game design is that if you're not using C++ then
you are essentially doomed to having to rewrite the wheel. After a lot of
research, I settled on utilizing SFML as the basis of the game engine, because
it is released under zlib.

One of the many, I suppose, _features_ of C++ is that a number of libraries are
available for solving things without having to rewrite the wheel. This makes me
personally uncomfortable on a number of levels, but that comes with
["C Hacker Syndrome"][1] which I definitely suffer from. Anyways, I decided
that I would attempt to put this.. _feature_ to my advantage, and find a GUI
library that works properly with SFML, is clean and simple, and is small.

Turns out, this isn't actually possible.

### [SFGUI][2] ###
SFGUI is the first thing I spent time working on. The problem with SFGUI is it
suffers from "There's a fleck on the speck on the tail on the frog on the bump
on the branch on the log in the hole in the bottom of the sea" Syndrome. If you
want to make a button, for example.

    // Create the label.
	m_label = sfg::Label::Create( "Hello world!" );

	// Create a simple button and connect the click signal.
	auto button = sfg::Button::Create( "Greet SFGUI!" );
	button->GetSignal( sfg::Widget::OnLeftClick ).Connect( std::bind( &HelloWorld::OnButtonClick, this ) );

	// Create a vertical box layouter with 5 pixels spacing and add the label
	// and button to it.
	auto box = sfg::Box::Create( sfg::Box::Orientation::VERTICAL, 5.0f );
	box->Pack( m_label );
	box->Pack( button, false );

	// Create a window and add the box layouter to it. Also set the window's title.
	auto window = sfg::Window::Create();
	window->SetTitle( "Hello world!" );
	window->Add( box );

	// Create a desktop and add the window to it.
	sfg::Desktop desktop;
	desktop.Add( window );

	// We're not using SFML to render anything in this program, so reset OpenGL
	// states. Otherwise we wouldn't see anything.
	render_window.resetGLStates();

So you want a button. There's a Label on the button in the box on the window
on the desktop in the SFML window on the openGL at the bottom of the code tree.
Needless to say, this lasted about five minutes in our source tree before I
broke down sobbing. Not to mention, the entire system renders in its own area
of the main SFML window. So when I tried to make an SFGUI button in the SFML
window on top of our background, yeah it didn't go to well, and I don't need
_another_ engine on top of the engine on top of this game engine. That's just..
absurd. Not to mention the documentation sucks. I hate crappy documentation.

### [TGUI][3] ###
TGUI bills itself as the "Simple GUI" for SFML. So naturally, I was really
quite excited to try it out. Until I figured out it required configuration
files, and every button had to be an image or it wouldn't render. So in theory,
you couldn't have a completely transparent button, which is one of the things
that this game will require. Secondly, if you require a configuration syntax,
you are _not_ a simple library. Ever. All I want is to be able to call methods
that create a button.

### [CEGUI][4] ###
Desperate times call for desperate measures. Or so I believed. Problem is,
CEGUI actually contains its own OpenGL renderer, which is redundant on top of
SFML. There is never a reason to have two OpenGL renderers, they will conflict.
Furthermore, CEGUI requires XML. Which is -not- a simple library. They claim
that this is to reduce the frustration of having to change your code to adjust
the GUI interface, but I would rather adjust my code than end up dependent on
XML files.

### Conclusion ### {: #cpp-library-hell-conclusion }
At the end of the day, All I really need to do for this particular engine is
lay out a few menus and then a general presentation on top of those for the
in game content. So I used [Thor][5] to render shapes, and then I'm writing my
own logic on top of that. This gives me total control over the implementation,
and the logic of the GUI system. Maybe I'm strange in that I don't mind having
to change values in my code and recompile, but I honestly prefer to do that.
Control over individual placement makes sense, and most of the time I define
my placement on top of positioning logic, so in those cases it makes more sense
to have it in the code than in an external file.

[1]: https://warp.povusers.org/OpenLetters/ResponseToTorvalds.html
[2]: https://sfgui.sfml-dev.de
[3]: https://tgui.eu
[4]: https://cegui.org.uk
[5]: https://www.bromeon.ch/libraries/thor/
