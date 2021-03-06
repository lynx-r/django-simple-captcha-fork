Welcome to Django Simple Captcha's documentation!
=================================================

******************
About this project
******************

Django Simple Captcha is an extremely simple, yet highly customizable Django application to add captcha images to any Django form.

.. image:: https://github.com/mothsART/django-simple-captcha/raw/master/docs/Captcha3.png

Features
++++++++

* Very simple to setup and deploy, yet very configurable
* Can use custom challenges (e.g. random chars, simple maths, dictionary word, ...)
* Custom generators, noise and filter functions alter the look of the generated image
* Supports text-to-speech audio output of the challenge text, for improved accessibility

Requirements
++++++++++++

* Django 1.0+
* A fairly recent version of the Python Imaging Library (PIL) compiled with FreeType support
* Flite is required for text-to-speech (audio) output, but not mandatory

******************
Contents:
******************

.. toctree::
   :maxdepth: 2

   usage.rst
   advanced.rst

