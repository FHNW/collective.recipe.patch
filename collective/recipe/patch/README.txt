
Supported options
=================

The recipe supports the following options:

path
    Define a directory in which the patch should be applied.
    example: src/some/directory/

egg
    Define which egg should be patched. You can also pin to version.
    example: some.egg<=1.1.1

patches
    Paths to patch files. These patches are applied in order.
    example: patches/my_very_sprecial.patch
             patches/another_loverly.patch

Example usage
=============

Our demo package which we will patch:

    >>> mkdir(sample_buildout, 'demo')
    >>> write(sample_buildout, 'demo', 'README.txt', " ")
    >>> write(sample_buildout, 'demo', 'demo.py',
    ... """# demo egg 
    ... """)
    >>> write(sample_buildout, 'demo', 'setup.py',
    ... """
    ... from setuptools import setup
    ...
    ... setup(
    ...     name = "demo",
    ...     version='1.0',
    ...     py_modules=['demo']
    ...     )
    ... """)
    >>> print system(buildout+' setup demo bdist_egg'), # doctest: +ELLIPSIS
    Running setup script 'demo/setup.py'.
    ...

Create our patch:

    >>> write(sample_buildout, 'demo.patch',
    ... """diff --git demo.py demo.py
    ... --- demo.py
    ... +++ demo.py
    ... @@ -1 +1,2 @@
    ...  # demo egg
    ... +# patching
    ... """)

Let's write out buildout.cfg to patch our demo package:

    >>> write(sample_buildout, 'buildout.cfg',
    ... """
    ... [buildout]
    ... parts = demo-patch
    ... index = demo/dist/
    ...
    ... [demo-patch]
    ... recipe = collective.recipe.patch
    ... egg = demo==1.0
    ... patches = demo.patch
    ... """)

Running the buildout gives us:

    >>> print system(buildout)
    Not found: demo/dist/zc.buildout/
    ...
    Installing demo-patch.
    ...
    Got demo 1.0.
    root: reading patch .../demo.patch
    ...
    root: successfully patched ...develop-eggs/demo-1.0-py2.6.egg/demo.py

    >>> ls(sample_buildout, 'develop-eggs', 'demo-1.0-py2.6.egg')
    d  EGG-INFO
    -  demo.py
    -  demo.pyc
    -  demo.pyo
    >>> cat(sample_buildout, 'demo', 'demo.py')
    # demo egg
    >>> cat(sample_buildout, 'develop-eggs', 'demo-1.0-py2.6.egg', 'demo.py')
    # demo egg
    # patching

Multiple patches
----------------

If you have more than one patch to apply:

    >>> write(sample_buildout, 'another.patch',
    ... """diff --git demo.py demo.py
    ... --- demo.py
    ... +++ demo.py
    ... @@ -1,2 +1 @@
    ... -# demo egg
    ...  # patching
    ... """)

Update your buildout.cfg to list the new patch. In this case,
another.patch should be applied after demo.patch:
    
    >>> write(sample_buildout, 'buildout.cfg',
    ... """
    ... [buildout]
    ... parts = demo-patch
    ... index = demo/dist/
    ...
    ... [demo-patch]
    ... recipe = collective.recipe.patch
    ... egg = demo==1.0
    ... patches = demo.patch
    ...           another.patch
    ... """)

Running the buildout gives us:

    >>> print system(buildout)
    Not found: demo/dist/zc.buildout/
    ...
    Installing demo-patch.
    ...
    Got demo 1.0.
    root: reading patch .../demo.patch
    ...
    root: successfully patched ...develop-eggs/demo-1.0-py2.6.egg/demo.py
    root: reading patch .../another.patch
    ...
    root: successfully patched ...develop-eggs/demo-1.0-py2.6.egg/demo.py

    >>> cat(sample_buildout, 'develop-eggs', 'demo-1.0-py2.6.egg', 'demo.py')
    # patching

External binaries
-----------------

We can also set an external binary to use for patching:

    >>> write(sample_buildout, 'buildout.cfg',
    ... """
    ... [buildout]
    ... parts = demo-patch
    ... index = demo/dist/
    ...
    ... [demo-patch]
    ... recipe = collective.recipe.patch
    ... patch-binary = patch
    ... egg = demo==1.0
    ... patches = demo.patch
    ... """)

Running the buildout gives us:

    >>> print system(buildout)
    Not found: demo/dist/zc.buildout/
    ...
    Installing demo-patch.
    ...
    Got demo 1.0.
    root: reading patch .../demo.patch
    patching file demo.py
    ...

    >>> ls(sample_buildout, 'develop-eggs', 'demo-1.0-py2.6.egg')
    d  EGG-INFO
    -  demo.py
    -  demo.py.orig
    -  demo.pyc
    -  demo.pyo
    >>> cat(sample_buildout, 'demo', 'demo.py')
    # demo egg
    >>> cat(sample_buildout, 'develop-eggs', 'demo-1.0-py2.6.egg', 'demo.py')
    # demo egg
    # patching

Patching an egg installed in another part
-----------------------------------------

Another possibility is to install an egg with zc.recipe.egg (or
probably any other recipe) and patch it afterwards.  However, it is
necessary to install the egg unzipped, and the egg may end up in the
eggs-folder instead the develop-eggs folder.

    >>> write(sample_buildout, 'buildout.cfg',
    ... """
    ... [buildout]
    ... parts = demo-egg demo-patch
    ... index = demo/dist/
    ...
    ... [demo-egg]
    ... recipe = zc.recipe.egg
    ... eggs = demo==1.0
    ... unzip = true
    ...
    ... [demo-patch]
    ... recipe = collective.recipe.patch
    ... egg = ${demo-egg:eggs}
    ... patches = demo.patch
    ... """)

Running the buildout gives us:

    >>> print system(buildout)
    Not found: demo/dist/zc.buildout/
    ...
    Installing demo-egg.
    ...
    Got demo 1.0.
    Installing demo-patch.
    ...
    root: successfully patched ...eggs/demo-1.0-py2.6.egg/demo.py

    >>> ls(sample_buildout, 'eggs', 'demo-1.0-py2.6.egg')
    d  EGG-INFO
    -  demo.py
    -  demo.pyc
    -  demo.pyo
    >>> cat(sample_buildout, 'demo', 'demo.py')
    # demo egg
    >>> cat(sample_buildout, 'eggs', 'demo-1.0-py2.6.egg', 'demo.py')
    # demo egg
    # patching
