2025-April-14 update
====================

This is a clone of the downloadable file at <https://pypi.org/project/mediawiki-parser/>

Specifically, it's from this `tar.gz` file:
`https://files.pythonhosted.org/packages/46/96/84d6cfc99aa21663af8ffccd6618481a0da21a4f18daba51c727f716ad42/mediawiki-parser-0.4.1.tar.gz`

Other than changes to this readme (which I changed from an `.rst` file to an `.md` file), I've only made two commits:

- the first is my ruff auto-linter changes.

- the second, [here](https://github.com/Brown-University-Library/mediawiki-parser-0.4.1_with_pijnu_fix/commit/814df70f76dad81397248da97cb5ae7cbd06828a), is the fix to be able to install the `pijnu` dependency when installing via [uv](https://github.com/astral-sh/uv). 
  
The purpose, to be able to install via uv, like (all on one line):

```
uv pip install git+ssh://git@github.com/Brown-University-Library/mediawiki-parser-0.4.1_with_pijnu_fix.git@516ca5e58b2efbbe3a91f8c526ddd8533c6c690c
```

Everything below, in this file, is unchanged.

---
---

.. image:: https://travis-ci.org/peter17/mediawiki-parser.svg?branch=master
   :alt: Build Status
   :target: https://travis-ci.org/peter17/mediawiki-parser

Presentation
============

This is a parser for MediaWiki's (MW) syntax. It's goal is to transform wikitext into an abstract syntax tree (AST) and then render this AST into various formats such as plain text and HTML.

It is an original work by Peter Potrowl and his mentor Erik Rose achieved during the Google Summer of Code 2011.


Requirements
============

This parser relies on Pijnu. You must install the latest version of Pijnu, available at: https://github.com/peter17/pijnu

Do not use the version from http://spir.wikidot.com, which is outdated.

For basic and simple installation, just try:

::

 pip install mediawiki-parser

How it works
============

Two files, preprocessor.pijnu and mediawiki.pijnu describe the MW syntax using patterns that form a grammar. Another Python tool called Pijnu will interpret those grammars and use them to match the wikitext content and build the AST.

Then, specific Python functions will render the leaves of the AST into the wanted format.

The reason why we use two grammars is that we will first substitute the templates in the wikitext with a preprocessor before actually parsing the content of the page.

Building the parsers
====================

The preprocessor and mediwiki parsers must be built from the Pijnu
grammars before you can use mediawiki-parser. You can build them through
setup.py, possibly setting the PYTHONPATH to point at pijnu:

::

 cd /PATH/TO/mediawiki-parser/
 env PYTHONPATH=/PATH/TO/pijnu python setup.py build_parsers

How to test
===========

The current simplest way to test the tool is to put wikitext inside the wikitext.txt file. Then, run:

::

 python parser.py

and the wikitext will be rendered as HTML in the article.htm file.

Other ways might be implemented in the future.

Unit tests
----------

Install nose and run:

::

 cd /PATH/TO/mediawiki-parser/
 env PYTHONPATH=/PATH/TO/pijnu/ nosetests tests

How to use in a program
=======================

Example for HTML
----------------
In order to use this tool to render wikitext into HTML in a Python program, you can use the following lines:

::

 templates = {}
 allowed_tags = []
 allowed_self_closing_tags = []
 allowed_attributes = []
 interwiki = {}
 namespaces = {}

 from mediawiki_parser.preprocessor import make_parser
 preprocessor = make_parser(templates)

 from mediawiki_parser.html import make_parser
 parser = make_parser(allowed_tags, allowed_self_closing_tags, allowed_attributes, interwiki, namespaces)

 preprocessed_text = preprocessor.parse(source)
 output = parser.parse(preprocessed_text.leaves())

The `output` string will contain the rendered HTML. You should describe the behavior you expect by filling the variables of the first lines:
 * if the wikitext calls foreign templates, put their names and content in the `templates` dict (e.g.: `{'my template': 'my template content'}`)
 * if some HTML tags are allowed on your wiki, list them in the `allowed_tags` list (e.g.: `['center', 'big', 'small', 'span']`; avoid `'script'` and some others, for security reasons)
 * if some self-closing HTML tags are allowed on your wiki, list them in the `allowed_self_closing_tags` list (e.g.: `['br', 'hr']`; avoid `'script'` and some others, for security reasons)
 * if some HTML tags are allowed on your wiki, list the attributes they can use the `allowed_attributes` list (e.g.: `['style', 'class']`; avoid `'onclick'` and some others, for security reasons)
 * if you want to be able to use interwiki links, list the foreign wikis in the `interwiki` dict (e.g.: `{'fr': 'http://fr.wikipedia.org/wiki/'}`)
 * if you want to be able to distinguish between standard links, file inclusions or categories, list the namespaces of your wiki in the `namespaces` dict (e.g.: `{'Template': 10, 'Category': 14, 'File': 6}` where the numbers are the namespace codes used in MW)

Example for text
----------------
In order to use this tool to render wikitext into text in a Python program, you can use the following lines:

::

 templates = {}

 from mediawiki_parser.preprocessor import make_parser
 preprocessor = make_parser(templates)

 from mediawiki_parser.text import make_parser
 parser = make_parser()

 preprocessed_text = preprocessor.parse(source)
 output = parser.parse(preprocessed_text.leaves())

The `output` string will contain the rendered text.
If the wikitext calls foreign templates, put their names and content in the `templates` dict (e.g.: `{'my template': 'my template content'}`)

Example for templates substitution
----------------------------------
If you just want to replace the templates in a given wikitext, you can just call the preprocessor and no rendering postprocessor:

::

 templates = {}

 from mediawiki_parser.preprocessor import make_parser
 preprocessor = make_parser(templates)

 output = preprocessor.parse(source)

The `output` string will contain the rendered wikitext.
Put the templates names and content in the `templates` dict (e.g.: `{'my template': 'my template content'}`)

Postprocessors
--------------

The parser produces an AST. In order to provide human readable output, three postprocessors are provided:
 * html.py, for HTML output
 * text.py, for text output
 * raw.py, for raw output

For now, we mainly focused on HTML postprocessor. The text output might not be as cleaned as expected.

You can adapt them according to your needs.

Known bugs
==========

This tool should be able to render any wikitext page into text or HTML.

However, it does not intent to be bug-for-bug compatible with MW. For instance, using HTML entities in template calls (e.g.: `'{{temp&copy;late}}`') is currently not supported.

Please don't hesitate to report bugs that you may find when using this tool.

Special thanks
==============
 * To Nicholas Burlett for his directory restructure, performance improvements and other fixes
